# Kısım 17: AMDP (ABAP Managed Database Procedures)

*CDS view yetmediğinde — SQLScript döngülerini ve geçici tabloları ABAP dünyasına getirmek.*

---

## ☕ Önce zihinsel model

CDS view'lar JOIN'ler, filtreler ve projeksiyonlar için harika — SQL'in doğal olarak yaptığı şeyler. Ama SQL bildirimseldir: *ne* istediğinizi tanımlarsınız, nasıl hesaplayacağınızı değil. Bazen gerçekten prosedürel bir şeye ihtiyacınız olur: sonuç kümesi üzerinde yineleyin, kümülatif toplamları koruyun, geçici tablo oluşturun, koşullara göre dallanın ve şekillendirilmiş bir sonuç döndürün. Bu artık bir view değil — bu bir stored procedure.

Klasik ABAP dünyasında bunu veriyi ABAP'a çekerek, döngüde dolaşarak ve bellekte hesaplayarak yapardınız. HANA'da bu verimsizdir. Veritabanı, bu tür işlemler için ABAP uygulama sunucunuzdan çok daha fazla işlem gücüne sahiptir.

**AMDP — ABAP Managed Database Procedures**, SAP'ın yanıtıdır: HANA SQLScript prosedürü yazın, ama onu bir ABAP sınıfından çağırın ve yönetin. İki dünyanın en iyisi — ABAP'ın tip güvenliği ve taşıma sistemi, HANA'nın işlem gücü.

> 💡 **Kısa gerçek kontrolü:** AMDP bir uzman aracıdır. Tipik bir projede veri ihtiyaçlarının %90'ı Open SQL + CDS ile karşılanır. Bir CDS view çok karmaşık hale geldiğinde, performans profili ABAP taraflı işlemenin darboğaz olduğunu gösterdiğinde ya da gerçekten prosedürel hesaplama (karmaşık kümülatif toplamlar, karar-ağacı tarzı mantık, yinelemeli veri dönüşümleri) gerektiğinde AMDP'ye uzanırsınız.

---

## 17.1 CDS ne zaman yetmez

### 1️⃣ Benzetme

.NET veya Python dünyasında yaşadığınız ORM-vs-stored-procedure tartışmasını düşünün:

- **ORM / LINQ:** CRUD, ilişkiler, basit filtreler için mükemmel. Temiz, test edilebilir, DB'den bağımsız.
- **Ham SQL view:** Her yerde yeniden kullandığınız sabit bir projeksiyon için iyi.
- **Stored procedure:** Karmaşık prosedürel mantığınız olduğunda — döngüler, cursor'lar, geçici tablolar, çok adımlı hesaplamalar — ve bunların uygulama sunucusuna gidiş dönüş yapmadan *veritabanı motorunun içinde* gerçekleşmesini istediğinizde.

ABAP'ın karşılığı:

| Senaryo | Araç |
|---------|------|
| Basit SELECT, JOIN, filtre | Open SQL |
| Yeniden kullanılabilir projeksiyon, Fiori/OData için anotasyonlu | CDS View Entity |
| Prosedürel mantık, döngüler, geçici tablolar — DB'de kalmalı | AMDP |
| Mantık iş kuralı ağırlıklı ve DB performansı sorun değil | ABAP sınıf metodu |

### Yaygın AMDP kullanım senaryoları

- **Kümülatif bakiye hesaplama** — sıralı FI belgelerini yineleyip kümülatif bakiyeyi korumanız gerekir; bir pencere fonksiyonu *belki* işe yarar, ama iş kuralı bir prosedür döngüsünün daha temiz olduğu kadar karmaşıktır.
- **Çok adımlı stok tahsisi** — tek bir SQL ifadesiyle düzgün eşleşmeyen öncelik kurallarıyla siparişlere stok tahsis etmek.
- **Karmaşık hiyerarşik BOM patlatma** — özyineleme derinliği ve ayıklama kurallarının önemsiz olmadığı malzeme listesi.
- **Özel analitik toplama** — HANA analitik motorunun doğrudan sunmadığı bir şey.

> ⚠️ **C#/Python tuzağı:** AMDP metot gövdesi içine ABAP ifadeleri *koyamazsınız* — metot saf SQLScript'tir. `SELECT INTO @lt_result` yok, `LOOP AT` yok, alan sembolü yok. Metot gövdesi olduğu gibi HANA'ya iletilir. Bunu Razor ile C# dosyasına gömülmüş JavaScript şablonu yazmak gibi düşünün: tek dosyada iki ayrı dünya.

---

## 17.2 AMDP sınıfı anatomisi

AMDP, özel bir interface ve özel bir metot anahtar kelimesiyle sıradan bir ABAP sınıfıdır.

### Üç parça

1. **`IF_AMDP_MARKER_HDB` implemente edin** — bu bir işaret interface'idir (implemente edilecek metot yoktur). ABAP runtime'a "bu sınıf veritabanı prosedürleri içeriyor" der.
2. **Metodu** sınıf tanımında normal şekilde beyan edin.
3. **Metodu** `BY DATABASE PROCEDURE FOR HDB LANGUAGE SQLSCRIPT` ekiyle implemente edin.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — Dapper veya ADO.NET ile stored procedure çağrısı
// Prosedür DB'de; C# sadece çağırıyor
public async Task<IEnumerable<SalesAggregate>> GetSalesAggregatesAsync(
    string companyCode, int fiscalYear)
{
    using var conn = new SqlConnection(_connectionString);
    return await conn.QueryAsync<SalesAggregate>(
        "usp_GetSalesAggregates",
        new { CompanyCode = companyCode, FiscalYear = fiscalYear },
        commandType: CommandType.StoredProcedure);
}
```

```python
# Python — psycopg2 ile stored procedure çağrısı
def get_sales_aggregates(conn, company_code: str, fiscal_year: int):
    cursor = conn.cursor()
    cursor.callproc("get_sales_aggregates", [company_code, fiscal_year])
    return cursor.fetchall()
```

Her iki durumda da: ABAP/Python çağırıcıdır; iş DB'de gerçekleşir. AMDP aynı fikirdir, yalnızca prosedür tanımı *bir ABAP sınıfının içinde* yaşar — ayrı olarak veritabanında değil; versiyonlanmış, taşınmış ve tip güvenli.

### 3️⃣ ABAP'taki karşılığı — eksiksiz bir AMDP sınıfı

```abap
"! AMDP sınıfı: belirli bir dönem için müşteri başına satış toplamlarını hesaplar.
"! Tamamen HANA SQLScript'te çalışır — ABAP belleğinde değil.
CLASS zcl_sales_aggregator DEFINITION
  PUBLIC FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    " ① İşaret interface'i — metot yok, sadece runtime'a bayrak
    INTERFACES if_amdp_marker_hdb.

    " ② AMDP metodunu diğer ABAP metotları gibi beyan et
    "   Not: parametreler DDIC tipleriyle yazılmış
    TYPES:
      BEGIN OF ty_sales_total,
        kunnr   TYPE kunnr,      " müşteri numarası
        name1   TYPE name1,      " müşteri adı
        netwr   TYPE netwr_ak,   " net değer
        waerk   TYPE waerk,      " para birimi
      END OF ty_sales_total,
      tt_sales_total TYPE STANDARD TABLE OF ty_sales_total WITH EMPTY KEY.

    CLASS-METHODS get_sales_by_customer
      IMPORTING
        iv_bukrs      TYPE bukrs          " şirket kodu
        iv_date_from  TYPE dats           " başlangıç tarihi
        iv_date_to    TYPE dats           " bitiş tarihi
      EXPORTING
        et_result     TYPE tt_sales_total
      RAISING
        cx_amdp_error.

ENDCLASS.


CLASS zcl_sales_aggregator IMPLEMENTATION.

  METHOD get_sales_by_customer
    BY DATABASE PROCEDURE              "③ Sihirli anahtar kelime
    FOR HDB                            "   HANA için (HDB = HANA DB)
    LANGUAGE SQLSCRIPT                 "   Dil = SQLScript
    OPTIONS READ-ONLY                  "   Bu prosedür yalnızca okur
    USING                              "④ Kullanılan tüm DB nesnelerini beyan et
      vbak                             "   Satış siparişi başlığı
      vbap                             "   Satış siparişi kalemleri
      kna1.                            "   Müşteri ana verisi

    " ─────────────────────────────────────────────────────────────────
    " Buradan itibaren her şey saf SQLScript — ABAP değil!
    " Kullanabilecekleriniz: geçici tablolar (:lt_xxx), değişkenler,
    " döngüler, IF/ELSEIF, standart SQL fonksiyonları, HANA'ya özgü fonksiyonlar.
    " ─────────────────────────────────────────────────────────────────

    DECLARE lv_date_from NVARCHAR(8);
    DECLARE lv_date_to   NVARCHAR(8);

    " SQLScript ABAP tiplerini doğrudan kullanmaz — dönüştürün
    lv_date_from := :iv_date_from;
    lv_date_to   := :iv_date_to;

    " Adım 1: müşteri başına sipariş değerlerini topla
    lt_order_agg = SELECT
        h.kunnr,
        SUM( i.netwr ) AS total_netwr,
        h.waerk
      FROM vbak AS h
      INNER JOIN vbap AS i ON i.vbeln = h.vbeln
      WHERE h.bukrs   = :iv_bukrs
        AND h.erdat  >= :lv_date_from
        AND h.erdat  <= :lv_date_to
        AND h.abgru  = ''              -- reddedilmemiş
      GROUP BY h.kunnr, h.waerk;

    " Adım 2: müşteri adıyla zenginleştir
    et_result = SELECT
        a.kunnr,
        c.name1,
        a.total_netwr AS netwr,
        a.waerk
      FROM :lt_order_agg AS a
      INNER JOIN kna1 AS c ON c.kunnr = a.kunnr
      ORDER BY a.total_netwr DESC;

  ENDMETHOD.

ENDCLASS.
```

Temel anatomi noktaları:

| Öğe | Anlamı |
|-----|--------|
| `IF_AMDP_MARKER_HDB` | ABAP'a söyler: "bu sınıfın DB prosedürleri var." Zorunlu. |
| `BY DATABASE PROCEDURE FOR HDB LANGUAGE SQLSCRIPT` | Aşağıdaki her şey SQLScript; HANA'ya gönderilir |
| `OPTIONS READ-ONLY` | Optimizasyon ipucu — yazma prosedürleri için atlayın |
| `USING vbak vbap kna1` | Açık bağımlılık bildirimi — ABAP, prosedürün hangi tabloları kullandığını izler |
| `:lt_order_agg` | SQLScript tablo değişkeni (geçici tablo) — `:` ön eki SQLScript'e özgü, ABAP değil |
| `et_result` | EXPORTING parametresi — SQLScript doğrudan buna atar |

---

## 17.3 AMDP'yi ABAP'tan çağırma, hata ayıklama ve performans

### Çağırma — hiçbir fark yok

```abap
" AMDP metodunu herhangi bir statik sınıf metodu gibi çağırın
DATA lt_result TYPE zcl_sales_aggregator=>tt_sales_total.

TRY.
    zcl_sales_aggregator=>get_sales_by_customer(
      IMPORTING
        ev_result    = lt_result        " bekleyin — çağırılandan EXPORTING = çağıranda IMPORTING
      EXPORTING
        iv_bukrs     = '1000'
        iv_date_from = '20240101'
        iv_date_to   = '20241231'
    ).
  CATCH cx_amdp_error INTO DATA(lx_amdp).
    " AMDP düzeyinde hata: prosedür HANA'da başarısız oldu
    MESSAGE lx_amdp->get_text( ) TYPE 'E'.
ENDTRY.

LOOP AT lt_result INTO DATA(ls_line).
  WRITE: / ls_line-kunnr, ls_line-name1, ls_line-netwr, ls_line-waerk.
ENDLOOP.
```

> ⚠️ **C#/Python tuzağı:** ABAP'ta EXPORTING/IMPORTING yönü *çağırılanın* bakış açısındandır. Metot `EXPORTING et_result` beyan eder; siz onu çağrı noktasında `IMPORTING et_result` olarak alırsınız. Bu C# geliştiricilerinin beklediğinin tersidir ("exporting" sanki *gönderiyormuş* gibi geliyor). Sadece şunu hatırlayın: EXPORTING = OUT parametresi = geri alıyorsunuz.

### AMDP hata ayıklama

Normal ABAP debugger SQLScript içine giremez. Hata ayıklama seçenekleriniz:

1. **ADT HANA Runtime Tools** (Eclipse'de) — SQLScript gövdesine `BREAK` ifadeleriyle kesme noktası koyun.
2. **HANA Studio / SQL Console** — üretilen prosedürü HANA kataloğundan (`SYS.PROCEDURES`) kopyalayıp doğrudan çalıştırın.
3. **Ara sonuç SELECT'leri ekleyin** — geçici olarak `:lt_order_agg`'dan SELECT ekleyip prosedür orta durumunu incelemek için ayrı bir çıktı tablosu olarak döndürün.
4. **SQL EXPLAIN PLAN** — üretilen prosedürü HANA'nın plan görselleştiricisinden geçirerek performans darboğazlarını bulun.

> 🧭 **İş hayatında:** Bir AMDP yavaşsa ilk kontrol edeceğiniz şey: `USING` cümlesinin bir tablo eksik olup olmadığı (tam referans taramasına yol açabilir), WHERE cümlesi alanlarının HANA sütun deposu indeksinde olup olmadığı ve SQLScript geçici tablolarında gereksiz cross-join yapılıp yapılmadığı. SQL Server'da yavaş bir stored procedure analiz etmekle aynı düşünce süreci.

### Performans değerlendirmeleri

- AMDP, HANA DB motorunda çalışır — ABAP uygulama sunucusuna ağ gidiş dönüşü ortadan kalkar.
- SQLScript prosedürü HANA tarafından ilk çağrıda derlenir ve önbelleğe alınır.
- Parametreler üretilen SQL'e literal olarak geçirilir; varsayılan olarak bağlama değişkeni değildir, bu nedenle yüksek kardinaliteli girdilerde plan önbelleği parçalanabilir — buna dikkat edin.
- `OPTIONS READ-ONLY`, HANA'nın ek okuma optimizasyonları uygulamasına olanak tanır.
- ABAP'a çok büyük sonuç kümesi döndürmekten kaçının; `et_result` tablosunun tamamı DB↔AppServer sınırını aşar.

---

## 17.4 Araç seçimi: Open SQL mu, CDS mi, AMDP mi?

Bu her gün karşılaşacağınız en pratik sorudur ve her geliştiricinin zaten bildiği tartışmayla birebir örtüşür.

### 1️⃣ Benzetme

| DB dünyası tartışması | ABAP/SAP karşılığı |
|-----------------------|-------------------|
| ORM (EF, SQLAlchemy) | Open SQL |
| SQL View / EF ham SQL | CDS View Entity |
| Stored Procedure | AMDP |

### Karar tablosu

| Kriter | Open SQL | CDS View | AMDP |
|--------|----------|----------|------|
| Basit SELECT / filtre / sıralama | ✅ En iyi | ✅ İyi | ❌ Fazla karmaşık |
| Çok tablolu join (yeniden kullanılabilir) | 🟡 Mümkün | ✅ En iyi | ❌ Fazla karmaşık |
| Fiori/OData için anotasyonlar | ❌ Yok | ✅ Tek seçenek | ❌ Yok |
| Pencere fonksiyonlu karmaşık toplama | 🟡 Mümkün | ✅ İyi | ✅ En iyi |
| Prosedürel mantık (döngüler, dallar, geçici tablolar) | ❌ Yok | ❌ Desteklenmiyor | ✅ Tek seçenek |
| HANA dışı DB'de çalışmalı | ✅ Evet | ✅ Evet | ❌ Yalnızca HANA |
| Saf ABAP birim testlerinde test edilebilir | ✅ Kolay (test double) | ✅ Kolay | 🟡 Zor (gerçek HANA gerekir) |
| ABAP'ta taşıma / versiyonlama | ✅ Evet | ✅ Evet | ✅ Evet |
| HANA'da toplu veri performansı | 🟡 İyi | ✅ Çok iyi | ✅ Mükemmel |

### 3️⃣ Kodda karar akışı

```abap
" Pratikte kural:
"
" 1. Bunu SELECT ... FROM <cds_view> WHERE ile yazabilir miyim?
"    → Open SQL. Bitti.
"
" 2. Bu mantık birden fazla yerde kullanılıyor mu, UI anotasyonları gerekiyor mu,
"    yoksa bir OData/Fiori uygulaması için veri modeli mi?
"    → CDS View Entity. Bir kez tanımla.
"
" 3. Mantık şunları gerektiriyor mu:
"    - Koşumlu durumla satırlarda yineleme?
"    - Karmaşık çok adımlı ara geçici tablolar?
"    - CDS'nin sunmadığı HANA'ya özgü analitik fonksiyonlar?
"    - ABAP taraflı işlemenin darboğaz olduğunu kanıtlanmış performans sorunu?
"    → AMDP. Ve yalnızca o zaman.
```

> 💡 **Clean core açısı:** AMDP *clean core değildir* (HANA'ya özgüdür, BTP ortamlarındaki ABAP Cloud Platform uyumlu değildir). BTP ABAP Environment (Steampunk) üzerine geliştiriyorsanız AMDP mevcut olmayabilir. On-premise S/4HANA için tam olarak desteklenir. Hedef ortamınızı bilin.

---

## 🧠 Özet

| Kavram | C#/Python karşılığı | ABAP/AMDP |
|--------|---------------------|-----------|
| DB'deki stored procedure | `SqlCommand` / `cursor.callproc()` | AMDP metot gövdesi (SQLScript) |
| Koddaki prosedür bildirimi | Dapper wrapper sınıfı | ABAP sınıfı + `IF_AMDP_MARKER_HDB` |
| Prosedürdeki geçici tablo | `#temp` / CTE | SQLScript tablo değişkeni `:lt_xxx` |
| Prosedür çağırma | `ExecuteStoredProcedure()` | Statik sınıf metodu çağrısı |
| Prosedür düzeyinde istisna | `SqlException` | `CX_AMDP_ERROR` |
| "Stored proc ne zaman kullanılır" tartışması | ORM vs. ham SQL vs. stored proc | Open SQL vs. CDS vs. AMDP |

**Hatırlanacak dört şey:**
1. AMDP = ABAP sınıfının içinde yönetilen SQLScript stored procedure. Yalnızca HANA.
2. Metot gövdesi saf SQLScript'tir — içinde ABAP ifadesi kullanılamaz.
3. `USING` cümlesi: SQLScript'in başvurduğu her DB tablosu/view'ını beyan edin.
4. Varsayılan araç Open SQL veya CDS olmalıdır; prosedürel DB mantığı gerçekten gerektiğinde ve ek karmaşıklığı haklı kılacak performans avantajı olduğunda AMDP'ye uzanın.

---

*[← İçindekiler](../content.md) | [← Önceki: CDS View'ları](16-cds-views.md) | [Sonraki: OData & REST: Büyük Resim →](18-odata-and-rest-intro.md)*
