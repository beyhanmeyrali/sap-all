# Ek A: C# / Python ↔ ABAP Hızlı Başvuru

*Sürekli güncellenen çeviri sözlüğü — çalışırken bu eki açık tutun.*

---

## 📌 Bu ek nasıl kullanılır?

Buradaki her tablo doğrudan bir eşleşme sunar: sol sütun C# ya da Python'da yazdıklarınız; sağ sütun ise SAP/ABAP karşılığı. Temiz bir eşdeğer olmadığı durumlarda, sahte bir benzetme uydurmak yerine bunu açıkça belirtiyoruz.

---

## 1 — Kavramlar: Ekosistem haritası

| Bildiğiniz kavram (C# / Python) | SAP'ın karşılığı | SAP aracı / transaction |
|---|---|---|
| Class library / NuGet paketi | **Package + Transport Request** | SE80, ADT (Eclipse) |
| Solution / proje klasörü | **Package** (namespace kapsayıcısı) | SE80, DEVC nesnesi |
| NuGet paket yöneticisi | Doğrudan karşılığı yok — standart SAP nesneleri **Support Packages / Notes** aracılığıyla gelir | SNOTE, transaction SPAM |
| Entity Framework modeli / SQLAlchemy | **DDIC** (Data Dictionary) — tablolar, yapılar, veri elemanları, domain'ler | SE11 |
| SQL Server / PostgreSQL | **SAP HANA** (ve taşınabilirlik için HANA SQL değil **Open SQL** yazarsınız) | SE11, HANA Studio / HDBSQL |
| ASP.NET Web API controller | **OData servisi** (klasik: SEGW; modern: RAP CDS tabanlı servis) | SEGW, ADT |
| DTO / record / dataclass | **Structure + Work Area** | SE11 (structure tipi) |
| `List<T>` / pandas DataFrame | **Internal Table** | ABAP dil yapısı |
| `IEnumerable<T>` / generator | **LOOP AT** (lazy streaming karşılığı yok; ABAP tam sonuç kümesini okur) | — |
| `interface` + dependency injection | **Interface** (`INTERFACE … ENDINTERFACE`) + framework seviyesi DI için **BAdI** | SE18, SE19, SE24 |
| Abstract class / virtual method | ABAP abstract class / method (aynı kavram, farklı anahtar kelime sırası) | SE24, ADT |
| `try / catch / finally` | `TRY … CATCH cx_root … CLEANUP … ENDTRY` | — |
| Özel exception sınıfı | `cx_static_check`, `cx_dynamic_check` veya `cx_no_check` sınıfından miras alan bir sınıf | SE24 |
| `async / await` | ⚠️ Klasik ABAP'ta **doğrudan karşılığı yoktur**. Arka plan işlemi için **background job** kullanılır (SM36/SM37). Modern ABAP Cloud'da sınırlı paralellik için `CL_ABAP_PARALLEL` mevcuttur — ancak "async/await" değil, "zamanlanmış iş" olarak düşünün. | SM36, SM37 |
| `Task<T>` / `Promise<T>` | ⚠️ **Karşılığı yoktur.** SAP dialog katmanında senkron-öncelikli çalışır. | — |
| `HttpClient` / `requests` | `CL_HTTP_CLIENT` (klasik) veya `CL_WEB_HTTP_CLIENT_MANAGER` (modern) | — |
| REST API endpoint | **OData servisi** (SEGW/RAP) veya **RFC etkin Function Module** | SEGW, SE37 |
| XML/JSON serileştirme | `CL_SXML_*`, `CL_ABAP_CONV_OUT_CE`, `/UI2/CL_JSON` — manuel ya da sınıf tabanlı | — |
| Unit test (`xUnit`, `pytest`) | ADT'ye entegre **ABAP Unit** (`CLASS … FOR TESTING`) | ADT test runner |
| Loglama / Serilog | **Application Log** (`BAL_*` function module'leri) veya geliştirme sırasında `CL_DEMO_OUTPUT` | SLG1 |
| Regex | `CL_ABAP_REGEX` + `CL_ABAP_MATCHER` | — |
| Git / sürüm kontrolü | **abapGit** (açık kaynak, Git ↔ ABAP nesneleri eşleştirir) + **Transport Requests** (sistemler arası taşıma) | SE01, SE09, STMS |

---

## 2 — Sözdizimi karşılaştırması

Üç sütun: **C#** | **Python** | **ABAP**. Çeviriyi bulmak için satır boyunca okuyun.

### 2.1 Değişken tanımlama

```csharp
// C# — güçlü tipli / var
int count = 0;
string name = "Acme";
var customer = new Customer();
```

```python
# Python — dinamik tipli
count = 0
name = "Acme"
customer = Customer()
```

```abap
" ABAP — DATA ifadesi (7.40'tan itibaren satır içi DATA() de kullanılabilir)
DATA lv_count TYPE i.
DATA lv_name  TYPE string.
DATA lo_cust  TYPE REF TO zcl_customer.

" Satır içi tanımlama (modern, tercih edilen)
DATA(lv_total) = 0.
```

> ⚠️ **Dikkat:** ABAP ön ekleri bir kural, derleyici tarafından zorunlu kılınmaz: `lv_` = yerel değişken, `lt_` = yerel tablo, `ls_` = yerel yapı, `lo_` = yerel nesne referansı, `gv_` / `gt_` / `gs_` = global. Bunları her yerde göreceksiniz.

---

### 2.2 String interpolasyon / biçimlendirme

```csharp
// C#
string msg = $"Customer {id} has {count} orders.";
```

```python
# Python
msg = f"Customer {id} has {count} orders."
```

```abap
" ABAP — string şablonları (|…| sözdizimi, ABAP 7.40+)
DATA(lv_msg) = |Customer { lv_id } has { lv_count } orders.|.

" Klasik birleştirme (eski kod — eski sistemlerde sık görülür)
CONCATENATE 'Customer' lv_id 'has' lv_count 'orders.'
  INTO lv_msg SEPARATED BY space.
```

---

### 2.3 Koleksiyonlar / internal table başlatma

```csharp
// C#
var orders = new List<Order>
{
    new Order { Id = 1, Amount = 100m },
    new Order { Id = 2, Amount = 200m }
};
```

```python
# Python
orders = [
    {"id": 1, "amount": 100},
    {"id": 2, "amount": 200},
]
```

```abap
" ABAP — VALUE constructor (7.40+)
TYPES: BEGIN OF ty_order,
         id     TYPE i,
         amount TYPE p DECIMALS 2,
       END OF ty_order.
DATA lt_orders TYPE TABLE OF ty_order WITH EMPTY KEY.

lt_orders = VALUE #(
  ( id = 1  amount = '100.00' )
  ( id = 2  amount = '200.00' )
).
```

---

### 2.4 Döngü + filtre

```csharp
// C#
foreach (var o in orders.Where(o => o.Amount > 100))
    Console.WriteLine(o.Id);
```

```python
# Python
for o in [o for o in orders if o["amount"] > 100]:
    print(o["id"])
```

```abap
" ABAP — LOOP AT … WHERE (7.40+)
LOOP AT lt_orders INTO DATA(ls_order) WHERE amount > 100.
  WRITE: / ls_order-id.
ENDLOOP.

" Field-symbols varyantı (büyük tablolarda daha hızlı — yerinde değiştirme yapar)
LOOP AT lt_orders ASSIGNING FIELD-SYMBOL(<s_order>) WHERE amount > 100.
  WRITE: / <s_order>-id.
ENDLOOP.
```

---

### 2.5 Null / başlangıç değeri kontrolü

```csharp
// C#
if (name == null || name == string.Empty) { ... }
if (obj == null) { ... }
```

```python
# Python
if not name: ...
if obj is None: ...
```

```abap
" ABAP — IS INITIAL her türün "sıfır değerini" kontrol eder
IF lv_name IS INITIAL.         " boş string, ya da sıfır, ya da başlangıç tarihi
  ...
ENDIF.
IF lo_object IS INITIAL.       " null referans
  ...
ENDIF.
" Modern: nesne referansları için IS NOT BOUND
IF lo_object IS NOT BOUND.
  ...
ENDIF.
```

> 💡 ABAP değişkenleri **her zaman** türlerinin sıfır değeriyle başlatılır (`0`, `''`, tarih için `'00000000'` vb.) — başlatılmamış / tanımsız durum yoktur.

---

### 2.6 Üçlü / koşullu ifade

```csharp
// C#
string label = active ? "Active" : "Inactive";
```

```python
# Python
label = "Active" if active else "Inactive"
```

```abap
" ABAP — COND (7.40+)
DATA(lv_label) = COND #( WHEN lv_active = abap_true
                          THEN 'Active'
                          ELSE 'Inactive' ).

" Eski kodlarda hâlâ yaygın olan klasik IF/ELSE
IF lv_active = abap_true.
  lv_label = 'Active'.
ELSE.
  lv_label = 'Inactive'.
ENDIF.
```

---

### 2.7 Switch / desen eşleştirme

```csharp
// C#
string desc = status switch
{
    "A" => "Active",
    "I" => "Inactive",
    _   => "Unknown"
};
```

```python
# Python (3.10+)
match status:
    case "A": desc = "Active"
    case "I": desc = "Inactive"
    case _:   desc = "Unknown"
```

```abap
" ABAP — CASE
CASE lv_status.
  WHEN 'A'. lv_desc = 'Active'.
  WHEN 'I'. lv_desc = 'Inactive'.
  WHEN OTHERS. lv_desc = 'Unknown'.
ENDCASE.

" SWITCH ifadesi (7.40+, satır içi)
DATA(lv_desc2) = SWITCH #( lv_status
  WHEN 'A' THEN 'Active'
  WHEN 'I' THEN 'Inactive'
  ELSE          'Unknown' ).
```

---

### 2.8 Exception yönetimi

```csharp
// C#
try
{
    var result = Divide(a, b);
}
catch (DivideByZeroException ex)
{
    Console.WriteLine(ex.Message);
}
finally
{
    Cleanup();
}
```

```python
# Python
try:
    result = divide(a, b)
except ZeroDivisionError as ex:
    print(ex)
finally:
    cleanup()
```

```abap
" ABAP
TRY.
    DATA(lv_result) = divide( a = lv_a  b = lv_b ).
  CATCH cx_sy_zerodivide INTO DATA(lx_ex).
    WRITE: / lx_ex->get_text( ).
  CLEANUP.
    " bu TRY bloğundan yakalanmamış bir exception fırlatıldığında çalışır
    cleanup( ).
ENDTRY.
```

> 📌 `cx_root`, ABAP'taki her exception'ın üst sınıfıdır — yakalamak C#'ta `Exception` yakalamakla eşdeğerdir. Mümkün olduğunca özel sınıfı yakalamayı tercih edin.

---

### 2.9 Veritabanından / tablodan tek satır okuma

```csharp
// C# (EF Core)
var customer = dbContext.Customers.FirstOrDefault(c => c.Id == id);
if (customer == null) throw new NotFoundException();
```

```python
# Python (SQLAlchemy)
customer = session.query(Customer).filter_by(id=id).first()
if customer is None:
    raise NotFoundException()
```

```abap
" ABAP — SELECT SINGLE
SELECT SINGLE *
  FROM zcustomer
  WHERE id = @lv_id
  INTO @DATA(ls_customer).

IF sy-subrc <> 0.
  RAISE EXCEPTION TYPE zcx_not_found.
ENDIF.
```

> ⚠️ **`sy-subrc` ABAP dönüş kodudur.** `0` = başarılı. Her veritabanı ifadesi, READ TABLE, CALL FUNCTION vb. bunu ayarlar. Her zaman kontrol edin. Her çağrı sonrası `null` kontrolü yapmanın ABAP karşılığıdır.

---

## 3 — Veri tipi eşleştirmesi

| C# tipi | Python tipi | ABAP tipi | Notlar |
|---|---|---|---|
| `int` | `int` | `i` | 32-bit integer |
| `long` | `int` | `int8` | 64-bit integer (7.40+) |
| `decimal` | `Decimal` | `p LENGTH n DECIMALS d` | Paketlenmiş ondalık — **para için kullanın** |
| `double` / `float` | `float` | `f` | Kayan nokta — finansal işlemlerde kullanmayın |
| `string` | `str` | `string` | Dinamik uzunluk (heap) |
| `char` | `str` (uzunluk 1) | `c LENGTH n` | Sabit uzunluklu char, sağ taraftan boşlukla doldurulur |
| `bool` | `bool` | `abap_bool` (`c LENGTH 1`) | `abap_true` = `'X'`, `abap_false` = `' '` — **yerel bool türü yoktur** |
| `DateTime` | `datetime` | `d` (tarih, YYYYAAGG) + `t` (saat, SSDDSS) | Tarih ve saat **ayrı türlerdir** |
| `DateTimeOffset` | — | `utclong` (ABAP 7.53+) | Mikrosaniye hassasiyetinde UTC zaman damgası |
| `TimeSpan` | `timedelta` | Doğrudan tür yok — saniye cinsinden hesaplayın (`i`) | — |
| `byte[]` | `bytes` | `xstring` | Ham ikili / hex string |
| `Guid` | `uuid.UUID` | `sysuuid_x16` / `sysuuid_c32` | `cl_system_uuid=>create_uuid_c32_static( )` |
| `enum` | `Enum` | Sabit değerli **Domain** (DDIC) veya `CONSTANTS` bloğu | Dil düzeyinde enum anahtar kelimesi yoktur |
| `null` | `None` | IS INITIAL / IS NOT BOUND | ABAP değişkenleri gerçekten null olmaz; başlangıç değerleri vardır |
| `object` | `object` | `TYPE REF TO object` | Kök referans tipi |

---

## 4 — Sezgisel model tuzakları

Bunlar her C#/Python geliştiricisinin ilk haftalarında tökezlediği noktalardır. Şimdi okuyun; garip bir şey olduğunda tekrar okuyun.

| # | Tuzak | Beklentiniz | ABAP ne yapar |
|---|---|---|---|
| 1 | **1'den başlayan tablo indeksleri** | `list[0]` ilk elemandır | `READ TABLE itab INDEX 1` ilk satırdır. 0 indeksi yoktur. |
| 2 | **Exception yerine `sy-subrc`** | Hatalı DB çağrısı exception fırlatır | Çoğu ABAP ifadesi `sy-subrc`'yi sessizce ayarlar. Manuel kontrol zorunludur. |
| 3 | **Büyük/küçük harf duyarsız** | `customerName ≠ CustomerName` | ABAP anahtar kelimeleri ve tanımlayıcıları tamamen büyük/küçük harf duyarsızdır. `DATA` = `data` = `Data`. |
| 4 | **`=` hem atama hem de karşılaştırmadır** | Eşitlik için `==`, atama için `=` | ABAP'ta `IF a = b` bir karşılaştırmadır. `a = b.` (IF dışında) ise atamadır. Bağlam belirler. |
| 5 | **Yapılar için değer semantiği** | Nesneler referanstır | ABAP **yapıları** değer türleridir — birini diğerine atamak tüm alanları kopyalar. C#'ın `struct`'ı gibi. |
| 6 | **Client / MANDT** | Bir veritabanı, bir şema | SAP aynı fiziksel veritabanında **birden fazla client** (mantıksal kiracı) çalıştırır. Client'a bağımlı her tabloda `MANDT` anahtar alanı vardır. Open SQL mevcut client'ı otomatik filtreler — çoğunlukla açıkça yazmazsınız, ancak var olduğunu bilmelisiniz. |
| 7 | **`COMMIT WORK`** | ORM / transaction otomatik commit veya rollback yapar | ABAP'ta veritabanı değişiklikleri siz açıkça `COMMIT WORK` çağırana kadar **commit edilmez**. Background job çökmesi commit edilmemiş değişiklikleri geri alır. `COMMIT WORK`'ü unutmak yaygın bir hatadır. |
| 8 | **Çöp toplayıcı kaygısı yok** | GC belleği yönetir | ABAP referans sayımlı bellek yönetimi kullanır. Dairesel referanslar sızıntıya yol açabilir, ancak pratikte nadiren sorun oluşturur. Internal table'lar kapsam dışına çıkınca temizlenir. |
| 9 | **İfadeleri nokta sonlandırır** | `;` ifadeleri sonlandırır | ABAP'ta her ifade `.` (nokta) ile biter. Eksik nokta sözdizimi hatasıdır. |
| 10 | **`,` ile zincirleme** | Bir satırda birden fazla ifade | ABAP'ta **zincir sözdizimi** vardır: `WRITE: / lv_a, lv_b, lv_c.` — `:` zinciri başlatır, `,` devam ettirir. |
| 11 | **WRITE ve çıktı yöntemleri** | `Console.WriteLine` / `print()` | `WRITE: / text.` klasik listeye çıktı verir. OOP kodunda `cl_demo_output` veya message class kullanın. |
| 12 | **Form rutinlerinde `TABLES` parametresi** | Koleksiyonları normal parametre olarak geçin | Eski stil alt programlar (`FORM/ENDFORM`) internal table'ları `TABLES` parametresiyle geçirir — eski kodlarda göreceğiniz bir tuhaflık. ABAP OOP'ta diğer parametreler gibi geçirilir. |
| 13 | **Transport-öncelikli geliştirme** | Git'e commit et, bitti | Her ABAP değişikliği bir **Transport Request** içinde yapılır. Nesnelerin canlıya çıkması için DEV → QAS → PRD üzerinden transport edilmesi gerekir. Üretim ortamında hiçbir şey "kendiliğinden" belirir. |
| 14 | **`TYPE` ve `LIKE`** | Tipler için tek anahtar kelime | `TYPE` bir DDIC tipi veya ABAP yerleşik tipine referans verir. `LIKE` mevcut bir değişkenin tipini kopyalar. Yeni kodda `TYPE` kullanmayı tercih edin. |
| 15 | **Sabit uzunluklu char dolgusu** | String'ler doğal olarak kırpılır/karşılaştırılır | `c` alanları sabit uzunluklarına kadar boşlukla doldurulur. `'AB'` ile `'AB  '` karşılaştırması çalışır; ancak birleştirme boşlukları getirir. Dinamik string'ler için `string` tipini kullanın. |

---

## 5 — Hızlı başvuru: Her gün yazacağınız yaygın ABAP kalıpları

```abap
" ── Internal table'a SELECT ────────────────────────────────────────────
SELECT id, name, city
  FROM zcustomer
  WHERE region = @lv_region
  INTO TABLE @DATA(lt_customers).

" ── READ TABLE — tek satır bulma ────────────────────────────────────────
READ TABLE lt_customers
  WITH KEY id = lv_id
  INTO DATA(ls_cust).
IF sy-subrc = 0.
  " bulundu
ENDIF.

" ── MODIFY TABLE — bellekte satır güncelleme ────────────────────────────
ls_cust-city = 'Munich'.
MODIFY TABLE lt_customers FROM ls_cust.

" ── Tablo satırı silme ──────────────────────────────────────────────────
DELETE TABLE lt_customers WITH TABLE KEY id = lv_id.

" ── Hızlı arama için sorted / hashed tablo ──────────────────────────────
DATA lt_hash TYPE HASHED TABLE OF ty_customer
     WITH UNIQUE KEY id.

" ── String işlemleri ────────────────────────────────────────────────────
DATA(lv_upper)  = to_upper( lv_name ).
DATA(lv_length) = strlen( lv_name ).
FIND 'error' IN lv_message IGNORING CASE.  " sy-subrc = 0 ise bulundu

" ── Tarih aritmetiği ─────────────────────────────────────────────────────
DATA(lv_today)    = sy-datum.             " sistem tarihi, d tipi
DATA(lv_tomorrow) = lv_today + 1.        " tarih aritmetiği olduğu gibi çalışır

" ── Nesne oluşturma ─────────────────────────────────────────────────────
DATA(lo_service) = NEW zcl_order_service( ).
DATA(lv_result)  = lo_service->process( iv_id = lv_id ).
```

---

*[← İçindekiler](../content.md) | [← Önceki: RAP: RESTful Application Programming (Final)](35-rap-restful-application-programming.md) | [Sonraki: Ek B: Yeni Başlayanlar İçin Sözlük →](appendix-b-glossary.md)*
