# Kısım 13: Uygulamalı — BAPI_ACC_DOCUMENT_POST ile Deftere İşleme

*FI deftere işleme BAPI'si, SAP finans entegrasyonunun "Hello World"ü. Bunu düzgün yapalım — başlık, satır kalemleri, commit, hata yönetimi ve hepsi.*

---

## ☕ Bu kısım neden var?

Her SAP projesi eninde sonunda FI'ya (Finansal Muhasebe) dokunur. İster eski faturaları taşıyor, ister bir harcama deftere işleme entegrasyonu kuruyor, ister şirketler arası transferleri otomatikleştiriyor olun, bir finansal belge deftere işlemeniz gerekecektir. `BAPI_ACC_DOCUMENT_POST` bunu yapmanın standart ve desteklenen yoludur.

Bu kısım tamamen uygulamalıdır. "Finansal belge nedir?"den SE38'e yapıştırabileceğiniz, şirket kodunu ve G/L hesaplarını sisteminize göre ayarlayıp çalıştırabileceğiniz eksiksiz, çalışır bir ABAP raporuna kadar gidiyoruz.

---

## 13.1 "Finansal Belge Deftere İşlemek" Ne Demek?

### 1️⃣ Benzetme

Muhasebede her işlem bir **çift kayıttır**: her borç için eşit bir alacak olmalıdır. SAP bunu katı bir kural olarak uygular — bir belgedeki tüm satır kalem tutarlarının toplamı **sıfır** olmalıdır (borç ve alacaklar yerel para biriminde sıfıra eşitlenir).

Deftere işlenmiş bir FI belgesi şunlara sahiptir:
- Bir **başlık** — şirket kodu, deftere işleme tarihi, belge türü, para birimi.
- **Satır kalemleri** — her satır, borç (`S` = Almanca Soll) veya alacak (`H` = Almanca Haben) tutarıyla bir G/L hesabına (genel muhasebe hesabı) işlenir.
- Tüm `S` tutarları − tüm `H` tutarlarının toplamı sıfıra eşit olmalıdır.

Kısıt denetimli bir veritabanı işlemi gibi düşünün: dengelenmezse her şey geri alınır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — dengeli bir muhasebe işleminin zihinsel modeli
public class JournalEntry
{
    public string CompanyCode  { get; set; }
    public DateTime PostingDate { get; set; }
    public string Currency     { get; set; }
    public List<JournalLine> Lines { get; set; }

    public bool IsBalanced() =>
        Lines.Sum(l => l.IsDebit ? l.Amount : -l.Amount) == 0;
}

public record JournalLine(string GlAccount, decimal Amount, bool IsDebit);
```

```python
# Python
from dataclasses import dataclass, field
from decimal import Decimal

@dataclass
class JournalEntry:
    company_code: str
    posting_date: str
    currency: str
    lines: list = field(default_factory=list)

    def is_balanced(self) -> bool:
        return sum(
            l['amount'] if l['is_debit'] else -l['amount']
            for l in self.lines
        ) == Decimal('0')
```

### BAPI yapıları

`BAPI_ACC_DOCUMENT_POST`, tek bir satır yazmadan önce anlamanız gereken dört temel parametre kullanır:

| Parametre | Tür | Amaç |
|-----------|------|---------|
| `DOCUMENTHEADER` | Yapı `BAPIACHE09` | Belge başlığı (şirket kodu, tarihler, tür) |
| `ACCOUNTGL` | Tablo `BAPIACGL09` | G/L hesap satır kalemleri (satır başına bir satır) |
| `CURRENCYAMOUNT` | Tablo `BAPIACCR09` | Her satır için tutar + para birimi (`ITEMNO_ACC` ile bağlı) |
| `RETURN` | Tablo `BAPIRET2` | Hatalar, uyarılar ve elde edilen belge numarası |

> ⚠️ **C#/Python tuzağı:** Tutarlar ve G/L satır kalemleri *ayrı* tablolarda yaşar ve `ITEMNO_ACC` ile bağlıdır. Bu, çoğu yeni geliştiriciye sürpriz yapar. `ACCOUNTGL`'deki 1. satırın `ITEMNO_ACC = 1` değeri vardır; `CURRENCYAMOUNT`'taki karşılık gelen satır da `ITEMNO_ACC = 1` değerine sahiptir. Bunlar o anahtar üzerinden birleştirilir. Aynı BAPI çağrısındaki iki tablo arasında bire-bir yabancı anahtar olarak düşünebilirsiniz.

---

## 13.2 SE37'de BAPI Interface'ini Okumak

Kod yazmadan önce **SE37**'ye gidip interface'i kendiniz görün.

1. SAP komut alanına `SE37` yazın, Enter'a basın.
2. "Function Module" alanına `BAPI_ACC_DOCUMENT_POST` yazın.
3. Enter'a basın (veya Display düğmesine).
4. Beş sekme görürsünüz: **Attributes**, **Import**, **Export**, **Changing**, **Tables**, **Exceptions**.

Önemli olanlar:

**Import sekmesi — `DOCUMENTHEADER` yapısı (BAPIACHE09):**

| Alan | Tür | Açıklama |
|-------|------|------------|
| `BUS_ACT` | `CHAR4` | İş işlemi (`RFBU` = GL deftere işleme) |
| `USERNAME` | `USNAM` | Kim deftere işliyor (genellikle `sy-uname`) |
| `COMP_CODE` | `BUKRS` | Şirket kodu (örn. `'1000'`) |
| `DOC_DATE` | `DATS` | Belge tarihi |
| `PSTNG_DATE` | `DATS` | Deftere işleme tarihi |
| `DOC_TYPE` | `BLART` | Belge türü (`SA` = G/L hesap belgesi, `KR` = satıcı faturası) |
| `CURRENCY` | `WAERS` | Para birimi anahtarı (`USD`, `EUR`) |
| `REF_DOC_NO` | `XBLNR` | Referans belge numarası (harici referans — izleme için kullanışlı) |
| `HEADER_TXT` | `BKTXT` | Başlık metni |

**Tables sekmesi — `ACCOUNTGL` (BAPIACGL09), temel alanlar:**

| Alan | Açıklama |
|-------|------------|
| `ITEMNO_ACC` | Kalem numarası (1, 2, 3…) — `CURRENCYAMOUNT` ile bağlantı |
| `GL_ACCOUNT` | G/L hesap numarası (10 karakter, sıfır dolgulu) |
| `COMP_CODE` | Şirket kodu (başlık değerini tekrarlayın) |
| `PSTNG_DATE` | Deftere işleme tarihi (başlık değerini tekrarlayın) |
| `ITEM_TEXT` | Satır kalemi metni |
| `TAX_CODE` | Vergi kodu (basit G/L deftere işlemeler için boş bırakın) |

**Tables sekmesi — `CURRENCYAMOUNT` (BAPIACCR09), temel alanlar:**

| Alan | Açıklama |
|-------|------------|
| `ITEMNO_ACC` | `ACCOUNTGL` satırına geri bağlanır |
| `CURRENCY` | Para birimi anahtarı |
| `AMT_DOCCUR` | Belge para birimindeki tutar (pozitif = borç, negatif = alacak) |

> 🧭 **İş hayatında:** SE37'de `BAPI_ACC_DOCUMENT_POST`'u interaktif olarak test etmek için **F8**'e basın. Tablo düğmelerine tıklayarak yapıları doldurun, geçerli şirket kodu, G/L hesapları ve dengeli bir tutar girin ve çalıştırın. RETURN tablosunu izleyin. Bu "önce SE37'de test et" iş akışı, karşılaştığınız her yeni BAPI'de saatlerce hata ayıklama süresinden sizi kurtarır.

---

## 13.3 ABAP'ta Başlık ve Satır Kalemi Tablolarını Doldurmak

Her şeyi eksiksiz bir programa bağlamadan önce veri yapılarını oluşturalım.

```abap
" ── Yerel tür bildirimleri ──────────────────────────────────────────────────
DATA: ls_header     TYPE bapiache09,
      lt_accountgl  TYPE TABLE OF bapiacgl09,
      lt_currency   TYPE TABLE OF bapiaccr09,
      lt_return     TYPE TABLE OF bapiret2.

DATA: ls_gl_line    TYPE bapiacgl09,
      ls_curr_line  TYPE bapiaccr09.

" ── Belge başlığını doldur ──────────────────────────────────────────────────
ls_header-bus_act    = 'RFBU'.             " GL deftere işleme
ls_header-username   = sy-uname.           " geçerli kullanıcı
ls_header-comp_code  = '1000'.             " şirket kodunuz
ls_header-doc_date   = sy-datum.           " bugün
ls_header-pstng_date = sy-datum.           " bugün
ls_header-doc_type   = 'SA'.               " SA = GL hesap belgesi
ls_header-currency   = 'USD'.
ls_header-ref_doc_no = 'EXT-REF-001'.      " harici referans
ls_header-header_txt = 'ABAP''den test deftere işleme'.

" ── Satır kalemi 1: BORÇ hesap 400000 (gider), USD 1000 ───────────────────
CLEAR ls_gl_line.
ls_gl_line-itemno_acc  = '1'.
ls_gl_line-gl_account  = '0000400000'.     " gider GL hesabı (10 basamak, sıfır dolgulu)
ls_gl_line-comp_code   = '1000'.
ls_gl_line-pstng_date  = sy-datum.
ls_gl_line-item_text   = 'Gider satırı'.
APPEND ls_gl_line TO lt_accountgl.

CLEAR ls_curr_line.
ls_curr_line-itemno_acc  = '1'.
ls_curr_line-currency    = 'USD'.
ls_curr_line-amt_doccur  = '1000.00'.      " pozitif = borç
APPEND ls_curr_line TO lt_currency.

" ── Satır kalemi 2: ALACAK hesap 100000 (banka), USD -1000 ─────────────────
CLEAR ls_gl_line.
ls_gl_line-itemno_acc  = '2'.
ls_gl_line-gl_account  = '0000100000'.     " banka GL hesabı
ls_gl_line-comp_code   = '1000'.
ls_gl_line-pstng_date  = sy-datum.
ls_gl_line-item_text   = 'Banka takas kalemi'.
APPEND ls_gl_line TO lt_accountgl.

CLEAR ls_curr_line.
ls_curr_line-itemno_acc  = '2'.
ls_curr_line-currency    = 'USD'.
ls_curr_line-amt_doccur  = '-1000.00'.     " negatif = alacak
APPEND ls_curr_line TO lt_currency.
```

> 💡 **Pratikte sıfır kuralı:** 1. satır +1000,00 (borç), 2. satır −1000,00 (alacak). Toplam = 0. Tutarlarınız sıfıra eşitlenmezse, BAPI RETURN tablosuna `F5703` hata mesajını koyar ve hiçbir belge deftere işlenmez. Exception fırlatılmaz — her seferinde RETURN'ü kontrol edin.

---

## 13.4 BAPI_TRANSACTION_COMMIT ve RETURN Tablosunu Kontrol Etmek

### Neden commit ayrıdır?

BAPI'ler SAP'ın **Mantıksal İşlem Birimi (LUW)** — veritabanı işlemi — içinde çalışır. Bir yazma BAPI'si çağırdığınızda, kayıtları kilitler ve değişiklikleri SAP'ın güncelleme görevinde hazırlıkta tutar. `BAPI_TRANSACTION_COMMIT`'i açıkça çağırana kadar veritabanına hiçbir şey commit edilmez. Bu kasıtlıdır: birden fazla BAPI'yi sırayla çağırabilir (başlık deftere işle, kalemler deftere işle, takip belgesi tetikle) ve hepsini tek seferde commit edebilirsiniz. Bir şeyler ters giderse `BAPI_TRANSACTION_ROLLBACK` çağırırsınız ve hiçbir şey yazılmaz.

```abap
" ── BAPI_TRANSACTION_COMMIT çağır ─────────────────────────────────────────────
CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
  EXPORTING
    wait = 'X'.   " 'X' = senkron — güncelleme görevinin bitmesini bekler
                  " Boşluk = asenkron (güncelleme tamamlanmadan döner)

" ── BAPI yazma çağrısından sonra doğrudan COMMIT WORK KULLANMAYIN ───────────
" COMMIT WORK.   " Yanlış — BAPI'nin güncelleme görevi mekanizmasını atlar
```

> ⚠️ **C#/Python tuzağı:** Bir BAPI'den sonra `COMMIT WORK` kullanmayın. SAP'ın güncelleme görevi (veritabanına gerçekte yazan arka plan işlemi) yalnızca `BAPI_TRANSACTION_COMMIT` tarafından doğru biçimde tetiklenir. `COMMIT WORK`'ü doğrudan kullanmak kısmi yazmalara ve veri tutarsızlığına yol açabilir. Bu, ABAP başlangıç seviyesindeki geliştiricilerin en sık yaptığı hatalardan biridir.

### RETURN tablosunu kontrol etmek

`RETURN` tablosu `BAPIRET2` tipindedir. Her satır, bir `TYPE` alanına sahip bir mesajdır:

| TYPE | Anlamı | Eylem |
|------|---------|--------|
| `S` | Başarı | Her şey yolunda |
| `I` | Bilgi | Bilgilendirici — okuyun |
| `W` | Uyarı | Kontrol edin — yine de başarılı olabilir |
| `E` | Hata | Başarısız — commit YAPMAYIN, ROLLBACK çağırın |
| `A` | Durdurma | Kesin başarısızlık — ROLLBACK çağırın |

```abap
" ── RETURN tablosunu kontrol et ve commit veya rollback kararı ver ──────────────
DATA: lv_has_error TYPE abap_bool VALUE abap_false,
      lv_doc_num   TYPE bkpf-belnr.

LOOP AT lt_return INTO DATA(ls_return).
  WRITE: / ls_return-type, ls_return-id, ls_return-number, ls_return-message.

  IF ls_return-type = 'E' OR ls_return-type = 'A'.
    lv_has_error = abap_true.
  ENDIF.

  " Belge numarası bu BAPI için return mesaj parametresinde bulunur
  IF ls_return-type = 'S' AND ls_return-id = 'RW' AND ls_return-number = '605'.
    " RW 605 mesajı: "& belgesi & şirket kodunda & mali yılı için deftere işlendi"
    lv_doc_num = ls_return-message_v1.
  ENDIF.
ENDLOOP.

IF lv_has_error = abap_true.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
  WRITE: / 'Deftere işleme başarısız — geri alındı.'.
ELSE.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING wait = 'X'.
  WRITE: / |Belge deftere işlendi: { lv_doc_num }|.
ENDIF.
```

---

## 13.5 Eksiksiz Çalışır ABAP Raporu

**SE38**'de (veya ADT'de) oluşturabileceğiniz, kendi kendine yeten tam bir rapor. `lc_bukrs` (şirket kodu), `lc_gl_debit` ve `lc_gl_credit`'i sisteminizde geçerli hesaplara göre ayarlayın. Geri kalanı, FI yapılandırılmış herhangi bir SAP sisteminde olduğu gibi çalışır.

```abap
*&---------------------------------------------------------------------*
*& Rapor  ZTEST_BAPI_FI_POST
*& Amaç: BAPI_ACC_DOCUMENT_POST'u göster — dengeli 2 satırlı
*&       G/L belgesi deftere işle ve elde edilen belge numarasını yazdır.
*&
*& KULLANIM:
*&   1. SE38'i açın, ZTEST_BAPI_FI_POST programı oluşturun (tür 1, havuzsuz)
*&   2. Bu kodu yapıştırın
*&   3. Sabitler bölümünü sisteminize göre ayarlayın
*&   4. Çalıştırmak için F8
*&---------------------------------------------------------------------*
REPORT ztest_bapi_fi_post.

*----------------------------------------------------------------------*
* SABİTLER — Bunları sisteminize göre ayarlayın
*----------------------------------------------------------------------*
CONSTANTS:
  lc_bukrs      TYPE bukrs  VALUE '1000',   " Şirket kodu
  lc_gl_debit   TYPE hkont  VALUE '0000400000',  " Gider GL hesabı (borç)
  lc_gl_credit  TYPE hkont  VALUE '0000100000',  " Banka/takas GL (alacak)
  lc_currency   TYPE waers  VALUE 'USD',
  lc_amount     TYPE wrbtr  VALUE '500.00',
  lc_doc_type   TYPE blart  VALUE 'SA'.     " SA = GL hesap belgesi

*----------------------------------------------------------------------*
* VERİ BİLDİRİMLERİ
*----------------------------------------------------------------------*
DATA: ls_header     TYPE bapiache09,
      lt_accountgl  TYPE TABLE OF bapiacgl09,
      lt_currency   TYPE TABLE OF bapiaccr09,
      lt_return     TYPE TABLE OF bapiret2.

DATA: ls_gl         TYPE bapiacgl09,
      ls_curr       TYPE bapiaccr09,
      ls_ret        TYPE bapiret2.

DATA: lv_has_error  TYPE abap_bool,
      lv_doc_number TYPE belnr_d.

*----------------------------------------------------------------------*
* BELGE BAŞLIĞINI DOLDUR
*----------------------------------------------------------------------*
ls_header-bus_act    = 'RFBU'.
ls_header-username   = sy-uname.
ls_header-comp_code  = lc_bukrs.
ls_header-doc_date   = sy-datum.
ls_header-pstng_date = sy-datum.
ls_header-doc_type   = lc_doc_type.
ls_header-currency   = lc_currency.
ls_header-ref_doc_no = |ZTEST-{ sy-datum }|.
ls_header-header_txt = 'ZTEST_BAPI_FI_POST''dan BAPI test deftere işlemesi'.

*----------------------------------------------------------------------*
* SATIR KALEMİ 1 — BORÇ (pozitif tutar)
*----------------------------------------------------------------------*
CLEAR ls_gl.
ls_gl-itemno_acc = '1'.
ls_gl-gl_account = lc_gl_debit.
ls_gl-comp_code  = lc_bukrs.
ls_gl-pstng_date = sy-datum.
ls_gl-item_text  = 'Test gider borç satırı'.
APPEND ls_gl TO lt_accountgl.

CLEAR ls_curr.
ls_curr-itemno_acc = '1'.
ls_curr-currency   = lc_currency.
ls_curr-amt_doccur = lc_amount.           " pozitif = borç
APPEND ls_curr TO lt_currency.

*----------------------------------------------------------------------*
* SATIR KALEMİ 2 — ALACAK (negatif tutar, 1. satırı dengeler)
*----------------------------------------------------------------------*
CLEAR ls_gl.
ls_gl-itemno_acc = '2'.
ls_gl-gl_account = lc_gl_credit.
ls_gl-comp_code  = lc_bukrs.
ls_gl-pstng_date = sy-datum.
ls_gl-item_text  = 'Test banka takas alacak satırı'.
APPEND ls_gl TO lt_accountgl.

CLEAR ls_curr.
ls_curr-itemno_acc = '2'.
ls_curr-currency   = lc_currency.
ls_curr-amt_doccur = lc_amount * -1.      " negatif = alacak
APPEND ls_curr TO lt_currency.

*----------------------------------------------------------------------*
* BAPI'Yİ ÇAĞIR
*----------------------------------------------------------------------*
WRITE: / '--- BAPI_ACC_DOCUMENT_POST çağrılıyor ---'.
WRITE: / |  Şirket: { lc_bukrs }  Tarih: { sy-datum }  Tutar: { lc_amount } { lc_currency }|.

CALL FUNCTION 'BAPI_ACC_DOCUMENT_POST'
  EXPORTING
    documentheader = ls_header
  TABLES
    accountgl      = lt_accountgl
    currencyamount = lt_currency
    return         = lt_return.

*----------------------------------------------------------------------*
* RETURN TABLOSUNU DEĞERLENDİR
*----------------------------------------------------------------------*
WRITE: / '--- RETURN mesajları ---'.
lv_has_error = abap_false.

LOOP AT lt_return INTO ls_ret.
  " Her mesajı biçimlendirerek yazdır
  WRITE: / |  [{ ls_ret-type }] { ls_ret-id } { ls_ret-number }: { ls_ret-message }|.

  CASE ls_ret-type.
    WHEN 'E' OR 'A'.
      lv_has_error = abap_true.
    WHEN 'S'.
      " Belge deftere işlendi başarı mesajını kontrol et
      " BAPI_ACC_DOCUMENT_POST, belge numarasını RW 605 mesajı için
      " MESSAGE_V1 alanına koyar
      IF ls_ret-id = 'RW' AND ls_ret-number = '605'.
        lv_doc_number = ls_ret-message_v1.
      ENDIF.
  ENDCASE.
ENDLOOP.

*----------------------------------------------------------------------*
* COMMIT VEYA ROLLBACK
*----------------------------------------------------------------------*
IF lv_has_error = abap_true.
  WRITE: / '--- Hatalar bulundu — ROLLBACK çağrılıyor ---'.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
  WRITE: / 'Belge deftere işlenmedi. Yukarıdaki hata mesajlarını kontrol edin.'.
ELSE.
  WRITE: / '--- Hata yok — COMMIT çağrılıyor (WAIT = X) ---'.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING
      wait = 'X'.

  IF lv_doc_number IS NOT INITIAL.
    WRITE: / |BAŞARILI: { lc_bukrs } şirketinde FI belgesi { lv_doc_number } deftere işlendi.|.
    WRITE: / |  -> FB03'ü açın ve { lv_doc_number } / { lc_bukrs } belgesini görüntülemek için girin.|.
  ELSE.
    " Commit başarılı ama mesajlardan belge numarasını ayrıştıramadık.
    " Yukarıdaki RETURN tablosunda RW605 mesajını manuel olarak kontrol edin.
    WRITE: / 'COMMIT çağrıldı — yeni belge için FB03''ü kontrol edin.'.
  ENDIF.
ENDIF.

WRITE: / '--- Bitti ---'.
```

> 🧭 **İş hayatında:** Başarılı bir deftere işlemeden sonra **FB03**'ü (FI Belgesini Görüntüle) açın ve sonucu doğrulamak için belge numarasını, şirket kodunu ve mali yılı girin. İki satır kalemi, borç ve alacakla birlikte göreceksiniz. Bu, bir entegrasyonu test ederken FI danışmanınızın sizden yapmanızı isteyeceği manuel doğrulama adımıdır.

> ⚠️ **C#/Python tuzağı:** `BAPI_ACC_DOCUMENT_POST` tarafından döndürülen belge numarası, doğrudan bir `EXPORTING` parametresinde değildir — başarı mesajı `RW 605`'in `MESSAGE_V1` alanı olarak `RETURN` tablosundan gelir. "Sonuç bir return mesajına gömülü" bu kalıp, çoğu geliştiriciye ilk seferinde sürpriz yapar. Çıktının nerede olduğunu bilmek için her zaman BAPI'yi SE37'de açın ve manuel olarak test edin.

---

## Yaygın Hatalar ve Anlamları

| RETURN mesajı | Ne yanlış gitti | Düzeltme |
|---------------|-----------------|-----|
| `F5703 Borç/alacak tutarları dengelenmedi` | AMT_DOCCUR toplamı ≠ 0 | Pozitif + negatif tutarların sıfıra eşitlendiğini doğrulayın |
| `GLT2201 XXXXXXXX hesabı gerektiriyor...` | G/L hesap yapılandırma sorunu | FI danışmanınızla FS00'deki hesabı kontrol edin |
| `F5808 XXXX şirket kodu mevcut değil` | Yanlış şirket kodu | Geçerli bukrs için T001 tablosunu kontrol edin |
| `F5 002 XXXXXXXX tarihi açık dönemde değil` | Deftere işleme tarihi kapalı bir mali dönemde | Geçerli açık dönemde bir tarih kullanın |
| `RW 605 & belgesi şirket kodu...` | **Başarı** — istediğiniz bu | Belge numarası için MESSAGE_V1'i ayrıştırın |

---

## 🧠 Özet

- Bir finansal belge **sıfıra dengelenmelidir** — tüm satır tutarlarının toplamı = 0.
- `BAPI_ACC_DOCUMENT_POST` bir başlık yapısı (`BAPIACHE09`), G/L satır tablosu (`ACCOUNTGL`) ve `ITEMNO_ACC` ile bağlı bir para birimi-tutar tablosu (`CURRENCYAMOUNT`) alır.
- Kod yazmadan önce **SE37**'de interaktif olarak test edin — bu çok büyük hata ayıklama süresinden tasarruf sağlar.
- BAPI'yi çağırdıktan sonra: `TYPE = 'E'` / `'A'` hataları için `RETURN` tablosunu kontrol edin. Temizse `WAIT = 'X'` ile `BAPI_TRANSACTION_COMMIT` çağırın. Hata varsa `BAPI_TRANSACTION_ROLLBACK` çağırın.
- Deftere işlenen belge numarası, `RW 605` başarı mesajının `RETURN-MESSAGE_V1` alanında bulunur.
- Sonucu **FB03**'te (FI Belgesini Görüntüle) doğrulayın.

---

*[← İçindekiler](../content.md) | [← Önceki: Function Module, RFC & BAPI](12-function-modules-rfc-bapi.md) | [Sonraki: Standart SAP'ı Geliştirmek (Enhancement) →](14-abap-enhancements.md)*
