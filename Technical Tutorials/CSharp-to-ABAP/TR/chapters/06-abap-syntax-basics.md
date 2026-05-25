# Kısım 6: C#/Python Geliştiricileri için ABAP Söz Dizimi Temelleri

*Zaten yaptığın şeyler için yeni yazımlar — ve en çok önem taşıyan tek bakış açısı değişikliği.*

---

## ☕ Dürüst bir oryantasyon

ABAP uzundur. 1980'lerde programcı olmayan iş geliştiricileri için tasarlandığında bu kasıtlıydı ve neredeyse İngilizce gibi okunacak kadar sözcüklüdür. C# veya Python'dan gelince ABAP'a ilk tepkin muhtemelen şu olacak: *"Bu kadar çok kelimeye neden ihtiyaç var ki?"*

Benimsemeye çalış. Fazla sözlülüğü bir kez kabul ettikten sonra ABAP aslında oldukça okunabilir. `LOOP AT customers INTO work_area.` ifadesini `foreach (var c in customers)` kadar yanlış okumak daha zordur. Ve söz dizimini öğrendikten sonra kalıplar sonsuzca tekrar eder — SAP'ın kod tabanı, herkesin aynı dili aynı şekilde yazması nedeniyle kayda değer ölçüde tutarlıdır.

Bu kısım, gerçek biletlerin %90'ında kullanacağın günlük söz dizimini kapsar. Zaten programlamayı bildiğin için hızlı ilerliyoruz — yalnızca yeni yazımları öğrenmek gerekiyor.

> ⚠️ **Baştan büyük bir bakış açısı değişikliği:** C#/Python'da hatalar exception fırlatır. ABAP'ta çoğu işlem, `sy-subrc` sistem değişkeninde bir dönüş kodu ayarlar. **`sy-subrc`'yi kontrol etmezsen, hata durumunda sessizce devam edersin.** Bu, exception tabanlı dillerden gelen geliştiriciler için en büyük tuzaktır. Bu noktayı kısım boyunca vurgulamaya devam edeceğiz.

---

## 6.1 Değişkenler ve Tipler — DATA, TYPES ve Yerleşik Tipler

### 1️⃣ Benzetme

ABAP'ta değişken bildirimi C#'a benzer — kullanmadan önce bildir (klasik ABAP'ta; Kısım 7, satır içi `DATA(x)` söz dizimini ele alır). Tipler doğrudan bildiklerinizle eşleşir, sadece farklı adlara sahiptir.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# bildirimleri
int    count    = 0;
string name     = "Acme";
decimal amount  = 12.50m;
bool   isActive = true;
DateTime orderDate = DateTime.Today;
```

```python
# Python — dinamik, ancak tip ipuçları niyeti gösterir
count:     int   = 0
name:      str   = "Acme"
amount:    float = 12.50
is_active: bool  = True
from datetime import date
order_date: date = date.today()
```

### 3️⃣ ABAP'taki karşılığı

```abap
" Temel değişken bildirimleri (metodun/programın en üstünde)
DATA lv_count    TYPE i.           " tamsayı (4 bayt, ~2 milyar aralık)
DATA lv_name     TYPE string.      " dinamik uzunluklu string
DATA lv_amount   TYPE p LENGTH 10 DECIMALS 2.  " paketlenmiş ondalık
DATA lv_flag     TYPE xfeld.       " CHAR(1) 'X'=true, ' '=false
DATA lv_date     TYPE d.           " tarih: 8 karakterlik sayısal metin olarak YYYYMMDD
DATA lv_time     TYPE t.           " saat: 6 karakterlik sayısal metin olarak HHMMSS

" Başlangıç değerleri — ABAP varsayılan olarak sıfırla başlatır:
" i → 0,  string → '',  p → 0.00,  d → '00000000',  c → '   '
" Değer tipleri için null referans yok (C#'tan farklı olarak)

" Sabitler
CONSTANTS c_max_items TYPE i VALUE 100.
CONSTANTS c_true      TYPE xfeld VALUE 'X'.
```

**Temel ABAP yerleşik tipleri:**

| ABAP tipi | Anlamı | C# karşılığı | Python karşılığı |
|-----------|---------|---------------|-------------------|
| `i` | Tamsayı (4 bayt) | `int` | `int` |
| `int8` | 8 baytlık tamsayı (7.4+) | `long` | `int` |
| `p` | Paketlenmiş ondalık (BCD) | `decimal` | `Decimal` |
| `f` | Kayan nokta (8 bayt) | `double` | `float` |
| `c` | Sabit uzunluklu karakter | `char[]` / `string` | `str` |
| `string` | Değişken uzunluklu string | `string` | `str` |
| `n` | Sayısal metin (yalnızca rakamlar) | özel `struct` | `str` (rakamlar) |
| `d` | Tarih (YYYYMMDD) | `DateOnly` | `datetime.date` |
| `t` | Saat (HHMMSS) | `TimeOnly` | `datetime.time` |
| `x` | Hex/ham baytlar | `byte[]` | `bytes` |
| `xstring` | Değişken uzunluklu baytlar | `byte[]` | `bytes` |

> ⚠️ **C#/Python tuzağı:** ABAP `d` (tarih), `YYYYMMDD` olarak saklanır — 8 karakterli sayısal metin, bir `DateTime` nesnesi *değil*. `sy-datum` (bugünün tarihi) `d` tipindedir. Tarihler üzerinde aritmetik çalışır: `lv_date = sy-datum + 30.` 30 gün ekler. Ancak tarihleri dış sistemlere iletmek her zaman açık biçimlendirme gerektirir.

**TYPES ile tip bildirimleri:**

```abap
" TYPES yerel bir tip takma adı tanımlar — C#'ta 'type' veya Python'da TypeAlias gibi
TYPES ty_amount TYPE p LENGTH 10 DECIMALS 2.
TYPES ty_name   TYPE c LENGTH 40.

DATA lv_price   TYPE ty_amount.
DATA lv_company TYPE ty_name.
```

**DDIC tiplerini doğrudan başvurma (tercih edilen yaklaşım):**

```abap
" Tipi tahmin etmek yerine DDIC'e başvur:
DATA lv_customer TYPE kunnr.    " KUNNR data element'i → CHAR 10
DATA lv_country  TYPE land1.    " LAND1 → CHAR 3
DATA lv_netval   TYPE netwr.    " NETWR → CURR 15,2

" Veya doğrudan bir tablo alanına başvur:
DATA lv_cust_name TYPE kna1-name1.  " KNA1-NAME1 ile aynı tip
```

Bu son kalıp son derece yaygın ve güçlüdür. Değişkenin doğru uzunluk, tip ve DDIC meta verilerini otomatik alır — tahmin yok, SAP alanı değiştirirse bakım yok.

---

## 6.2 Kontrol Akışı — IF, CASE, DO, WHILE, LOOP

### 1️⃣ Benzetme

ABAP'ta kontrol akışı C/C#/Python'dan çok İngilizce cümlelere benzer. Küme parantezi yoktur. Bloklar `END<anahtar_kelime>` ile biter. Kalıbı görünce her yerde tutarlıdır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# kontrol akışı
if (x > 0)
    DoSomething();
else if (x == 0)
    DoOther();
else
    DoFallback();

switch (status) {
    case "A": ProcessActive(); break;
    case "I": ProcessInactive(); break;
    default:  ProcessUnknown(); break;
}

for (int i = 1; i <= 10; i++) { ... }
while (condition) { ... }
```

### 3️⃣ ABAP'taki karşılığı

```abap
" IF / ELSEIF / ELSE / ENDIF
IF lv_amount > 0.
  CALL METHOD process_positive.
ELSEIF lv_amount = 0.
  CALL METHOD process_zero.
ELSE.
  CALL METHOD process_negative.
ENDIF.

" Mantıksal operatörler: AND  OR  NOT  (&& || ! değil)
IF lv_country = 'DE' AND lv_currency = 'EUR'.
  ...
ENDIF.

" String karşılaştırma: = eşitliktir (string'ler için büyük/küçük harf duyarlı)
IF lv_name CS 'GmbH'.     " CS = contains string (Contains() gibi)
  ...
ENDIF.
```

**Karşılaştırma operatörleri:**

| ABAP | Anlamı | C# karşılığı |
|------|---------|---------------|
| `=` veya `EQ` | eşit | `==` |
| `<>` veya `NE` | eşit değil | `!=` |
| `<` veya `LT` | küçük | `<` |
| `>` veya `GT` | büyük | `>` |
| `<=` veya `LE` | küçük veya eşit | `<=` |
| `>=` veya `GE` | büyük veya eşit | `>=` |
| `CO` | yalnızca içerir | — |
| `CS` | string içerir | `.Contains()` |
| `CP` | kalıba uygundur (joker `*` ve `+`) | `.Matches()` (regex benzeri) |

```abap
" CASE / WHEN / ENDCASE — switch gibi
CASE lv_status.
  WHEN 'A'.
    process_active( ).
  WHEN 'I'.
    process_inactive( ).
  WHEN OTHERS.           " OTHERS = varsayılan
    process_unknown( ).
ENDCASE.

" DO döngüsü — sabit yinelemeler (for i=1 to n gibi)
DO 10 TIMES.
  lv_counter = lv_counter + 1.
  IF lv_counter = 5.
    CONTINUE.            " C# continue gibi
  ENDIF.
  IF lv_counter > 8.
    EXIT.                " C# break gibi
  ENDIF.
ENDDO.

" WHILE döngüsü
WHILE lv_index <= lv_max.
  process_item( lv_index ).
  lv_index = lv_index + 1.
ENDWHILE.
```

> 💡 Klasik ABAP'ta `for(int i; ...)` yoktur. Index döngüleri `DO n TIMES.` veya `LOOP AT itab ... FROM x TO y.` (aşağıda ele alındı) kullanır. Netlik için `sy-index`, `DO` döngüsü içinde otomatik olarak geçerli yineleme sayısını tutar.

---

## 6.3 Structure'lar ve Work Area'lar — DTO Karşılığı

### 1️⃣ Benzetme

**Structure**, adlandırılmış bir alan koleksiyonudur — C#'taki record/DTO'nun veya Python'daki dataclass'ın SAP karşılığı. **Work area** ise bir structure'ın tek bir örneğini tutan bir değişkendir — tablo verilerini okuduğun bellekteki "geçerli satır"dır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — record tipi (DTO)
public record Customer(string Id, string Name, string Country, decimal Balance);

// Kullanım: tek örnek
var customer = new Customer("CUST01", "Acme GmbH", "DE", 5000m);
Console.WriteLine($"{customer.Id}: {customer.Name}");
```

```python
from dataclasses import dataclass

@dataclass
class Customer:
    id:      str
    name:    str
    country: str
    balance: float

customer = Customer(id="CUST01", name="Acme GmbH", country="DE", balance=5000.0)
print(f"{customer.id}: {customer.name}")
```

### 3️⃣ ABAP'taki karşılığı

```abap
" Seçenek 1: Yerel bir structure tipi satır içi tanımla
TYPES: BEGIN OF ty_customer,
         id      TYPE kunnr,       " CHAR 10
         name    TYPE name1_gp,    " CHAR 40
         country TYPE land1,       " CHAR 3
         balance TYPE wrbtr,       " CURR 13,2
       END OF ty_customer.

" Bir work area bildir (o tipin tek satırlık değişkeni)
DATA ls_customer TYPE ty_customer.   " 'ls_' = yerel structure geleneği

" Alanları ata — '.' değil '-' kullanır
ls_customer-id      = 'CUST01'.
ls_customer-name    = 'Acme GmbH'.
ls_customer-country = 'DE'.
ls_customer-balance = 5000.

" Alanları oku
WRITE: / ls_customer-id, ls_customer-name.

" Seçenek 2: Work area'yı doğrudan DDIC tablosundan tiplendir (çok yaygın)
DATA ls_kna1 TYPE kna1.   " KNA1 tablosuyla tam aynı şekle sahip work area

" SELECT SINGLE sonrası:
SELECT SINGLE * FROM kna1 INTO @ls_kna1
  WHERE kunnr = 'CUST01'.

IF sy-subrc = 0.
  WRITE: / ls_kna1-name1, ls_kna1-land1.
ENDIF.
```

**ABAP adlandırma kuralları** (her kod tabanında görürsün):

| Ön ek | Anlamı | Örnek |
|--------|---------|---------|
| `lv_` | yerel değişken (skaler) | `lv_count`, `lv_name` |
| `ls_` | yerel structure (work area) | `ls_customer`, `ls_order` |
| `lt_` | yerel tablo (internal table) | `lt_customers`, `lt_orders` |
| `lc_` | yerel sabit | `lc_max`, `lc_true` |
| `lo_` | yerel nesne referansı | `lo_handler`, `lo_report` |
| `gv_`, `gs_`, `gt_` | global karşılıklar | (sınıf/program global'leri) |
| `iv_`, `is_`, `it_` | import parametresi | (metot parametreleri) |
| `ev_`, `es_`, `et_` | export parametresi | (metot parametreleri) |
| `rv_`, `rs_`, `rt_` | return parametresi | (metot dönüş değeri) |

Bunlar dil tarafından zorunlu kılınmaz — güçlü bir topluluk geleneğidir. Bunlara uymak, kodunu deneyimli herhangi bir ABAP yazarı için anında okunabilir kılar.

---

## 6.4 Internal Table'lar — ABAP'ın List\<T\> / DataFrame'i

### 1️⃣ Benzetme

**Internal table**, aynı structure'ı paylaşan satırların bellekteki bir listesidir. C#'ta `List<T>` veya Python'da pandas DataFrame olarak düşün. Tüm ABAP'ın en çok kullanılan veri yapısıdır.

Veritabanı satırlarını internal table'a okursun, bir `LOOP` içinde işlersin, satırları değiştirir ve isteğe bağlı olarak geri yazarsın.

### 2️⃣ Bunu zaten biliyorsun

```csharp
public record Order(string Id, string CustomerId, decimal Amount);

// Bildir
var orders = new List<Order>();

// Satır ekle
orders.Add(new Order("ORD1", "CUST01", 100m));
orders.Add(new Order("ORD2", "CUST01", 200m));

// Döngü
foreach (var o in orders)
    Console.WriteLine($"{o.Id}: {o.Amount}");

// Tek bul
var found = orders.FirstOrDefault(o => o.Id == "ORD1");
```

```python
orders = []
orders.append({"id": "ORD1", "customer_id": "CUST01", "amount": 100.0})
orders.append({"id": "ORD2", "customer_id": "CUST01", "amount": 200.0})

for o in orders:
    print(f"{o['id']}: {o['amount']}")

found = next((o for o in orders if o["id"] == "ORD1"), None)
```

### 3️⃣ ABAP'taki karşılığı — Üç Tablo Tipi

ABAP'ın üç çeşit internal table'ı vardır. Doğrusunu seçmek önemli performans farkı yaratır:

| Tip | ABAP anahtar kelimesi | .NET/Python benzetmesi | Ne zaman kullan |
|------|-------------|---------------------|----------|
| Standard | `STANDARD TABLE` | `List<T>` | Genel kullanım; yoğun ekleme; index ile okuma |
| Sorted | `SORTED TABLE` | `SortedList<K,V>` | Her zaman sıralı; hızlı ikili arama okumaları |
| Hashed | `HASHED TABLE` | `Dictionary<K,V>` | Yalnızca anahtar erişimi; O(1) arama |

```abap
" Internal table bildirimi (üç yol — erişim kalıbına göre seç)

" 1. Standard table (ekle ve döngü — en yaygın)
DATA lt_orders TYPE STANDARD TABLE OF ty_order WITH EMPTY KEY.

" 2. Sorted table (sıralı tut, hızlı WHERE-anahtar okumaları)
DATA lt_orders_s TYPE SORTED TABLE OF ty_order
                 WITH UNIQUE KEY order_id.

" 3. Hashed table (saf anahtar arama, sözlük gibi)
DATA lt_orders_h TYPE HASHED TABLE OF ty_order
                 WITH UNIQUE KEY order_id.
```

**Satır ekleme:**

```abap
" APPEND standard table'a satır ekler (List.Add gibi)
DATA ls_order TYPE ty_order.
ls_order-order_id    = 'ORD1'.
ls_order-customer_id = 'CUST01'.
ls_order-amount      = 100.
APPEND ls_order TO lt_orders.

" Kısayol (satır içi struct değişmez — Kısım 7'de düzgünce ele alınır)
APPEND VALUE #( order_id = 'ORD2'  customer_id = 'CUST01'  amount = 200 )
  TO lt_orders.

" INSERT belirli bir konuma ekler (standard table)
INSERT ls_order INTO lt_orders INDEX 1.  " başa ekle
```

**Döngü:**

```abap
" LOOP AT ... INTO work_area — ana yineleme yapısı
LOOP AT lt_orders INTO DATA(ls_order).
  WRITE: / ls_order-order_id, ls_order-amount.
ENDLOOP.

" LOOP AT ... INTO ... WHERE — filtrelenmiş döngü (ekstra IF gerekmez)
LOOP AT lt_orders INTO DATA(ls_order)
  WHERE customer_id = 'CUST01'.
  process_order( ls_order ).
ENDLOOP.

" LOOP AT ... ASSIGNING FIELD-SYMBOL — yerinde değiştir (C#'taki ref gibi)
LOOP AT lt_orders ASSIGNING FIELD-SYMBOL(<order>).
  <order>-amount = <order>-amount * 1.1.   " %10 artış, tabloyu değiştirir
ENDLOOP.
```

> ⚠️ **C#/Python tuzağı:** `LOOP AT ... INTO ls_row`, satırın bir *kopyasını* alır. Döngü içinde `ls_row`'u değiştirmek tabloyu DEĞİŞTİRMEZ. Tabloyu yerinde değiştirmek için `ASSIGNING FIELD-SYMBOL(<row>)` kullan — field symbol, C#'taki `ref` gibidir. Bu, yeni başlayanlar için en yaygın ABAP hatasıdır.

**Tek satır okuma:**

```abap
" READ TABLE — tek satır arama
READ TABLE lt_orders INTO DATA(ls_found)
  WITH KEY order_id = 'ORD1'.

IF sy-subrc = 0.              " 0 = bulundu, 4 = bulunamadı
  WRITE: / ls_found-amount.
ELSE.
  WRITE: / 'Sipariş bulunamadı'.
ENDIF.

" Tanımlı anahtarlı sorted/hashed table'lar için hız adına WITH TABLE KEY kullan:
READ TABLE lt_orders_h INTO DATA(ls_h)
  WITH TABLE KEY order_id = 'ORD1'.
```

**Diğer kullanışlı tablo işlemleri:**

```abap
" SORT — standard table'ı sıralar
SORT lt_orders BY customer_id ASCENDING amount DESCENDING.

" DELETE — satırları kaldır
DELETE lt_orders WHERE amount < 10.
DELETE TABLE lt_orders_h WITH TABLE KEY order_id = 'ORD1'.

" DESCRIBE TABLE — .Count gibi
DESCRIBE TABLE lt_orders LINES DATA(lv_count).
" Modern karşılığı: lv_count = lines( lt_orders ).

" CLEAR — tabloyu boşalt
CLEAR lt_orders.   " tüm satırları kaldırır (lt_orders = VALUE #( ). ile aynı)
```

---

## 6.5 Open SQL — Tabloya SELECT (sy-subrc ile)

### 1️⃣ Benzetme

ABAP'ın **Open SQL**'i, veritabanının üzerindeki soyutlama katmanıdır. SQL benzeri söz dizimi yazarsın; ABAP bunu altta yatan DB için (HANA, Oracle, DB2) çevirir. Ham SQL'den temel farklar: `FROM dual` yok, eski söz diziminde çok tablolu join yok (ABAP bunu kod içinde halleder) ve MANDT filtresi otomatik.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — EF Core / LINQ
using var db = new AppDbContext();

// Çoklu satır
var orders = db.Orders
    .Where(o => o.Country == "DE")
    .OrderBy(o => o.CreatedAt)
    .ToList();

// Tek satır
var order = db.Orders.SingleOrDefault(o => o.OrderId == "ORD1");
if (order == null) { /* bulunamadı durumunu işle */ }

// Toplama
var total = db.Orders.Sum(o => o.Amount);
```

```python
# Python — SQLAlchemy
from sqlalchemy import select, func

# Çoklu satır
stmt = select(Order).where(Order.country == "DE").order_by(Order.created_at)
orders = session.execute(stmt).scalars().all()

# Tek satır
order = session.get(Order, "ORD1")  # veya filtreli .first()
if order is None:
    pass  # bulunamadı durumunu işle
```

### 3️⃣ ABAP'taki karşılığı

```abap
" SELECT çoklu satır INTO TABLE
SELECT order_id, customer_id, amount, currency
  FROM zorders
  INTO TABLE @DATA(lt_orders)
  WHERE currency = 'EUR'
  ORDER BY order_id.

IF sy-subrc = 0.
  " Satırlar bulundu — lt_orders'ı işle
ELSE.
  " Hiçbir satır eşleşmedi — sy-subrc = 4
ENDIF.

" SELECT SINGLE — tek satır (birincil anahtar aramaları için kullan)
SELECT SINGLE *
  FROM kna1
  INTO @DATA(ls_customer)
  WHERE kunnr = @lv_customer_id.

IF sy-subrc <> 0.
  " Müşteri bulunamadı
  MESSAGE 'Müşteri bulunamadı' TYPE 'E'.
ENDIF.

" JOIN ile SELECT (ABAP 7.4+)
SELECT o~order_id, o~amount, c~name1 AS customer_name
  FROM zorders AS o
  INNER JOIN kna1 AS c ON o~kunnr = c~kunnr
  INTO TABLE @DATA(lt_result)
  WHERE o~currency = 'EUR'.

" Toplama
SELECT currency, SUM( amount ) AS total_amount
  FROM zorders
  INTO TABLE @DATA(lt_totals)
  GROUP BY currency.

" Host değişkenlerindeki @ ön eki ABAP 7.4+ söz dizimidir (strict modda zorunlu)
DATA lv_min_amount TYPE zorders-amount VALUE 1000.
SELECT * FROM zorders INTO TABLE @DATA(lt_big_orders)
  WHERE amount >= @lv_min_amount.
```

> ⚠️ **C#/Python tuzağı:** `ORDER BY` olmayan `SELECT SINGLE`, birden fazla satır eşleşirse belirsizdir. Yalnızca birincil anahtar veya benzersiz anahtar aramaları için kullan. "Birçok satırdan ilkini ver" için `ORDER BY` ile `SELECT ... UP TO 1 ROWS` kullan — semantik olarak açıktır ve optimizer bunu iyi işler.

> ⚠️ **C#/Python tuzağı:** **SELECT sonrası her zaman `sy-subrc` kontrol et.** Sıfır satır döndüren `SELECT INTO TABLE`, `sy-subrc = 4` ayarlar ama exception fırlatmaz. Kontrol etmeden devam edersen boş tabloyu sessizce işlersin. Hiçbir şey bulamayan `SELECT SINGLE` da `sy-subrc = 4` ayarlar. Bu, sy-subrc davulu — tekrar çalıyoruz.

**Yaygın `sy-subrc` değerleri:**

| sy-subrc | Anlamı |
|----------|---------|
| `0` | Başarılı / bulundu |
| `4` | Bulunamadı / eşleşme yok |
| `8` | Genel hata (ifadeye bağlı) |
| `12` | Daha spesifik hata |

**Open SQL aracılığıyla UPDATE, INSERT, DELETE:**

```abap
" Satır ekle
INSERT zorders FROM @ls_order.           " sy-subrc = 0 ok, 4 = yinelenen anahtar

" Satır güncelle
UPDATE zorders SET amount = @lv_new_amount
  WHERE order_id = @lv_id.             " sy-subrc = 0 ok, 4 = bulunamadı

" Satırları sil
DELETE FROM zorders WHERE currency = 'XXX'.

" MODIFY — INSERT veya UPDATE (upsert)
MODIFY zorders FROM @ls_order.
```

> 💡 **Performans kuralı #1:** LOOP içinde SELECT yapma ("N+1 sorgu" problemi). İhtiyacın olan tüm veriyi önceden internal table'lara çek, ardından bellekte işle. SAP sistemleri milyonlarca kaydı işler; satır başına veritabanı round-trip felakettir.

```abap
" YANLIŞ — N+1 problemi
LOOP AT lt_order_ids INTO DATA(lv_id).
  SELECT SINGLE * FROM kna1 INTO @DATA(ls_cust)
    WHERE kunnr = @lv_id.             " Satır başına DB çağrısı!
ENDLOOP.

" DOĞRU — FOR ALL ENTRIES ile tek SELECT
IF lt_order_ids IS NOT INITIAL.       " FOR ALL ENTRIES öncesi zorunlu kontrol
  SELECT *
    FROM kna1
    INTO TABLE @DATA(lt_customers)
    FOR ALL ENTRIES IN @lt_order_ids
    WHERE kunnr = @lt_order_ids-kunnr.
ENDIF.
```

> ⚠️ **FOR ALL ENTRIES tuzağı:** `lt_order_ids` boşsa, `FOR ALL ENTRIES` tablodan *tüm satırları* döndürür — WHERE cümlesi olmaya eşdeğerdir. Her zaman `IF table IS NOT INITIAL` ile koru.

---

## 6.6 Alt Programlar, Metotlar, MESSAGE ve sy-subrc Hata İşleme

### 1️⃣ Benzetme

ABAP kodu **FORM alt programları** (eski usul C fonksiyonları gibi) veya **metotlar** (modern, nesne yönelimli) olarak organize edilebilir. Yeni kodda her zaman metot kullan. Eski kodda FORM'larla karşılaşırsın ve onları okuman gerekir, ama yaz metot.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — exception'larla hata işlemeli metot
public decimal CalculateDiscount(string customerId, decimal amount)
{
    if (string.IsNullOrEmpty(customerId))
        throw new ArgumentException("Müşteri ID gerekli");

    var rate = GetDiscountRate(customerId);  // exception fırlatabilir
    return amount * rate;
}

// Çağıran
try
{
    var discounted = CalculateDiscount("CUST01", 1000m);
}
catch (ArgumentException ex)
{
    Console.Error.WriteLine(ex.Message);
}
```

### 3️⃣ ABAP'taki karşılığı

**FORM alt programları (eski usul — yalnızca oku, yenisini yazma):**

```abap
" Eski usul FORM — eski programlarda görürsün
FORM calculate_discount
  USING    iv_customer TYPE kunnr
           iv_amount   TYPE wrbtr
  CHANGING cv_discount TYPE wrbtr.

  DATA lv_rate TYPE p DECIMALS 2.
  " ... mantık ...
  cv_discount = iv_amount * lv_rate.
ENDFORM.

" Çağrılır:
PERFORM calculate_discount
  USING    lv_customer_id
           lv_amount
  CHANGING lv_discount.
```

**Metotlar (modern — her zaman bunları tercih et):**

```abap
" Bir sınıfta (tam olarak Kısım 11'de ele alınır):
METHOD calculate_discount.
  " Parametreler sınıf tanım bölümünde tanımlanır:
  " IMPORTING iv_customer TYPE kunnr
  "           iv_amount   TYPE wrbtr
  " RETURNING VALUE(rv_discount) TYPE wrbtr
  " RAISING   zcx_discount_error

  IF iv_customer IS INITIAL.
    RAISE EXCEPTION TYPE zcx_discount_error
      EXPORTING textid = zcx_discount_error=>invalid_customer.
  ENDIF.

  DATA(lv_rate) = get_discount_rate( iv_customer ).
  rv_discount = iv_amount * lv_rate.
ENDMETHOD.

" Çağrılır:
TRY.
  DATA(lv_discount) = lo_pricer->calculate_discount(
    iv_customer = lv_customer_id
    iv_amount   = lv_amount ).
CATCH zcx_discount_error INTO DATA(lx_error).
  MESSAGE lx_error->get_text( ) TYPE 'E'.
ENDTRY.
```

**MESSAGE ifadeleri — SAP'ın kullanıcı bildirim sistemi:**

```abap
" MESSAGE, SAP'ta geri bildirim göstermenin yoludur — raporlarda, dialog'larda vb.
" Söz dizimi: MESSAGE '<metin>' TYPE '<tip>'.
" Veya:     MESSAGE ID '<msgclass>' TYPE '<tip>' NUMBER '<num>' WITH '<değişken>'.

" Tipler:
" 'S' = Başarı (yeşil)     → durum çubuğunda gösterilir, devam eder
" 'I' = Bilgi              → popup, kullanıcının Tamam'a tıklaması gerekir
" 'W' = Uyarı              → dialog'larda popup, devam eder
" 'E' = Hata               → ekranı hatayla yeniden gösterir, akışı durdurur
" 'A' = İptal              → transaction'ı anında iptal eder
" 'X' = Sonlandırma        → program dump (yalnızca ölümcül geliştirici hatalarında kullan)

MESSAGE 'Sipariş başarıyla kaydedildi.' TYPE 'S'.
MESSAGE 'Müşteri bulunamadı.' TYPE 'E'.
MESSAGE 'İşlem tamamlandı: & sipariş aktarıldı.' TYPE 'I'
  WITH lv_count.  " & WITH parametresiyle değiştirilir
```

**sy-subrc — dönüş kodu kalıbı:**

```abap
" ABAP'ın temel hata işleme kalıbı:
" Çoğu ifade sy-subrc ayarlar. Kontrol ETMEN ZORUNLUDUR.

" READ TABLE
READ TABLE lt_orders INTO DATA(ls_order)
  WITH KEY order_id = lv_id.
IF sy-subrc <> 0.
  MESSAGE 'Sipariş bulunamadı.' TYPE 'E'.
  RETURN.   " C#'taki return; gibi — geçerli metot/form'dan çıkar
ENDIF.

" OPEN SQL
SELECT SINGLE * FROM kna1 INTO @DATA(ls_customer)
  WHERE kunnr = @lv_id.
IF sy-subrc <> 0.
  " bulunamadı durumunu işle
ENDIF.

" Function module çağrısı (Kısım 12'de ele alınır)
CALL FUNCTION 'BAPI_CUSTOMER_GETDETAIL'
  EXPORTING
    customerno = lv_id
  IMPORTING
    customeraddress = ls_address
  EXCEPTIONS
    customer_not_found = 1
    OTHERS             = 2.

CASE sy-subrc.
  WHEN 0.  " başarı
  WHEN 1.  MESSAGE 'Müşteri bulunamadı' TYPE 'E'.
  WHEN 2.  MESSAGE 'Beklenmeyen hata' TYPE 'E'.
ENDCASE.
```

> ⚠️ **C#/Python tuzağı:** ABAP'ın `TRY...CATCH` (sınıf tabanlı exception'lar için, ABAP 6.10+) kullanımı mevcuttur ve modern kod bunu nesne yönelimli hata işleme için kullanır. Ancak eski ABAP'ın büyük çoğunluğu — standart SAP function module'leri ve BAPI'lerin neredeyse tamamı dahil — `sy-subrc` veya `CALL FUNCTION`'daki `EXCEPTIONS` bloğunu kullanır. Her ikisini de bilmek zorundasın. Her kıdemli ABAP geliştirici, kodunu debug ederken hemen "sy-subrc'yi kontrol ettin mi?" diye soracaktır.

**Diğer önemli sistem değişkenleri (`sy-*`):**

| Değişken | Anlamı | C# karşılığı |
|----------|---------|-------------|
| `sy-subrc` | Son ifadenin dönüş kodu | (exception varlığı) |
| `sy-datum` | Bugünün tarihi (d tipi) | `DateOnly.Today` |
| `sy-uzeit` | Geçerli saat (t tipi) | `TimeOnly.Now` |
| `sy-uname` | Geçerli kullanıcının oturum adı | `Environment.UserName` |
| `sy-mandt` | Geçerli istemci | (kiracı ID'si) |
| `sy-langu` | Oturum açma dili | `CultureInfo.CurrentCulture` |
| `sy-tabix` | Geçerli döngü index'i (LOOP AT içinde) | döngü değişkeni `i` |
| `sy-dbcnt` | Son DB ifadesinden etkilenen satırlar | `ExecuteNonQuery()` dönüşü |
| `sy-msgv1..4` | Son MESSAGE'dan gelen mesaj değişkenleri | (mesaj argümanları) |

---

## 6.7 Tam Örnek — Hepsini Bir Araya Getirme

Bu kısmın her şeyini gösteren gerçekçi bir mini program: bir müşteri için sipariş okur, toplamları hesaplar ve sonuçları çıktılar.

```abap
REPORT z_order_summary.

" --- Tipler ve veri ---
TYPES: BEGIN OF ty_order_summary,
         currency TYPE waers,
         total    TYPE wrbtr,
         count    TYPE i,
       END OF ty_order_summary.

DATA: lv_customer  TYPE kunnr VALUE 'CUST0001',
      ls_customer  TYPE kna1,
      lt_orders    TYPE STANDARD TABLE OF zorders WITH EMPTY KEY,
      lt_summary   TYPE STANDARD TABLE OF ty_order_summary WITH EMPTY KEY,
      ls_summary   TYPE ty_order_summary.

" --- Müşteri master'ını oku ---
SELECT SINGLE *
  FROM kna1
  INTO @ls_customer
  WHERE kunnr = @lv_customer.

IF sy-subrc <> 0.
  MESSAGE 'Müşteri bulunamadı.' TYPE 'E'.
  RETURN.
ENDIF.

WRITE: / 'Müşteri:', ls_customer-name1.

" --- Siparişleri oku ---
SELECT *
  FROM zorders
  INTO TABLE @lt_orders
  WHERE kunnr = @lv_customer
  ORDER BY waers.

IF sy-subrc <> 0.
  WRITE: / 'Bu müşteri için sipariş bulunamadı.'.
  RETURN.
ENDIF.

" --- Para birimine göre topla ---
LOOP AT lt_orders INTO DATA(ls_order).
  READ TABLE lt_summary ASSIGNING FIELD-SYMBOL(<sum>)
    WITH KEY currency = ls_order-waers.

  IF sy-subrc = 0.
    " Para birimi zaten özette — ekle
    <sum>-total = <sum>-total + ls_order-amount.
    <sum>-count = <sum>-count + 1.
  ELSE.
    " Yeni para birimi — ekle
    ls_summary-currency = ls_order-waers.
    ls_summary-total    = ls_order-amount.
    ls_summary-count    = 1.
    APPEND ls_summary TO lt_summary.
  ENDIF.
ENDLOOP.

" --- Çıktı ---
WRITE: / '--- Sipariş Özeti ---'.
LOOP AT lt_summary INTO DATA(ls_sum_row).
  WRITE: / ls_sum_row-currency, ls_sum_row-total, ls_sum_row-count.
ENDLOOP.
```

Bu tek program şunları kullanır: `DATA`, `TYPES`, `SELECT SINGLE`, `sy-subrc` kontrolü, `SELECT INTO TABLE`, `LOOP AT ... INTO`, `READ TABLE ... ASSIGNING FIELD-SYMBOL`, `APPEND` ve `WRITE`. Bunlar günlük sürücü araçlarındır.

---

## 🧠 Özet

- **DATA / TYPES ile bildir**; mümkün olduğunda DDIC tiplerini başvur (`TYPE kunnr`, `TYPE kna1-name1`).
- ABAP 1 indekslidir. String karşılaştırması `CHAR` tipleri için sondaki boşlukları yoksayar. `null` yoktur — tipler varsayılan olarak sıfır/başlangıç değerlerine sahiptir.
- **`sy-subrc`** neredeyse her şey için dönüş kodudur. Her SELECT, READ TABLE, CALL FUNCTION veya MODIFY sonrasında kontrol et. Kontrol etmemek #1 ABAP hatasıdır.
- `LOOP AT ... INTO` bir kopya alır (yalnızca okuma). `LOOP AT ... ASSIGNING FIELD-SYMBOL(<satır>)` yerinde değiştirme için referans verir.
- LOOP içinde SELECT yerine **`FOR ALL ENTRIES IN`** kullan. Her zaman `IS NOT INITIAL` ile koru.
- `MESSAGE` ile `'S'`/`'I'`/`'W'`/`'E'`/`'A'` tipi, klasik ABAP genelinde kullanıcı geri bildirim kalıbını kontrol eder.
- FORM'lar eski usuldür; **metotlar** (CLASS/ENDCLASS) modern yaklaşımdır.
- `TRY/CATCH` sınıf tabanlı exception'lar için vardır; `sy-subrc` + `EXCEPTIONS` bloğu function module'ler için vardır. Her ikisini de bil.

---

*[← İçindekiler](../content.md) | [← Önceki: Data Dictionary (DDIC)](05-data-dictionary-ddic.md) | [Sonraki: Modern ABAP Söz Dizimi (7.4+) →](07-modern-abap-syntax.md)*
