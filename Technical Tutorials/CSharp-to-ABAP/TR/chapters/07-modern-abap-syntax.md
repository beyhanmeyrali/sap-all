# Kısım 7: Modern ABAP Söz Dizimi (7.4+) — Güzel Kısım

*ABAP ciddi bir güncelleme aldı. Artık inanacağından daha çok C# ve Python'a benziyor — ve bu onu eğlenceli kılan şey.*

---

## ☕ Bilmediğin güncelleme

Çoğu ABAP iş ilanında yer almayan bir sır var: modern SAP sistemlerinde (ABAP 7.4+ üzerinde çalışan her şey — aktif SAP ERP kurulumlarının büyük bölümünü ve tüm S/4HANA kurulumlarını kapsar) karşılaştığın ABAP놀랍도록 C# veya Python'a benziyor.

Satır içi bildirimler. Koleksiyon ifadeleri. String şablonları. Fonksiyonel tablo işlemleri. Mükemmel değil — ABAP'ın hâlâ kendine özgü tuhaflıkları var — ama modern ABAP ile modern C# arasındaki uçurum, SAP'ın ünü kadar büyük değil. C# geliştiricilerine modern ABAP'ı ilk kez gösterdiğimde olağan tepki şu olur: *"Oh. Bu... aslında fena değil."*

Bu kısım ödül niteliğinde. Kısım 6'daki ayrıntılı klasik kalıpları alıp modern şekilde yeniden yazıyoruz. Ardından klasik ve moderniyi yan yana koyuyoruz, böylece her ikisini de okuyabilirsin (çünkü production kod tabanları bir karışımdır).

> 🧭 **İş hayatında:** Her sistem 7.4+ çalıştırmıyor ve her kod tabanı yapabilecekken bile modern söz dizimini kullanmıyor. `SE38` → `DEMO_ABAP_LANGUAGE_VERSION` çalıştırarak sisteminin ABAP sürümünü kontrol et. S/4HANA'daysan neredeyse kesinlikle 7.55+ üzerindesindir. Eski bir ECC sistemindeysen 7.02'de olabilir ve bu kısmın bir kısmı derlenmeyebilir. Sürümünü bil.

---

## 7.1 Satır İçi Bildirimler — DATA(x) ve FINAL(x)

### 1️⃣ Benzetme

Klasik ABAP, yürütülebilir koddan önce metodun en üstünde her değişkenin bildirilmesini gerektiriyordu — eski C kodu veya Pascal gibi. ABAP 7.4, satır içi bildirimler sundu: bir değişkeni ilk atama noktasında bildir ve derleyici tipi çıkarır.

Bu tam olarak C#'taki `var` veya Python'daki örtük tip çıkarımıdır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — var bildirim noktasında tipi çıkarır
var customer = db.Customers.Find("CUST01");   // tip = Customer?
var count    = customers.Count();              // tip = int
var name     = customer?.Name ?? "Unknown";    // tip = string
```

```python
# Python — her zaman örtük
customer = get_customer("CUST01")   # tip çıkarılır
count    = len(customers)
name     = customer.name if customer else "Unknown"
```

### 3️⃣ ABAP'taki karşılığı

```abap
" Klasik ABAP — her şeyi en üste bildir
DATA lv_customer_id TYPE kunnr.
DATA ls_customer    TYPE kna1.
DATA lt_orders      TYPE STANDARD TABLE OF zorders WITH EMPTY KEY.

lv_customer_id = 'CUST0001'.
SELECT SINGLE * FROM kna1 INTO ls_customer WHERE kunnr = lv_customer_id.
SELECT * FROM zorders INTO TABLE lt_orders WHERE kunnr = lv_customer_id.

" ---------------------------------------------------------------

" Modern ABAP 7.4+ — DATA() satır içi bildirimi
" Tip, sağ taraf bağlamından çıkarılır

DATA(lv_customer_id) = 'CUST0001'.   " tip = string değişmez bağlamı

SELECT SINGLE * FROM kna1
  INTO @DATA(ls_customer)             " ls_customer'ı TYPE kna1 olarak bildirir
  WHERE kunnr = @lv_customer_id.

SELECT * FROM zorders
  INTO TABLE @DATA(lt_orders)         " lt_orders'ı internal table olarak bildirir
  WHERE kunnr = @lv_customer_id.

" Metot çağrılarında:
DATA(lo_handler) = NEW zcl_order_handler( ).   " satır içi bildirildi + örneklendi
DATA(lv_result)  = lo_handler->calculate( lv_amount ).

" Satır içi bildirimiyle LOOP AT:
LOOP AT lt_orders INTO DATA(ls_order).
  DATA(lv_tax) = ls_order-amount * '0.19'.   " ilk kullanımda bildirildi
  WRITE: / ls_order-order_id, lv_tax.
ENDLOOP.
```

**`FINAL` — const/readonly karşılığı (ABAP 7.56+):**

```abap
" FINAL değişmez bir değişken bildirir — 'const' veya 'readonly' gibi
FINAL(lv_tax_rate) = '0.19'.
" lv_tax_rate = '0.21'.   ← derleme hatası: FINAL'e atama yapılamaz

" Döngü değişkenleri için çok kullanışlı, değiştirmeyi hiç düşünmediğinde:
LOOP AT lt_orders INTO FINAL(ls_order).
  " ls_order burada salt okunur — kendi kendini belgeler + zorlanır
  WRITE: / ls_order-amount.
ENDLOOP.
```

> 💡 Yalnızca okurken döngülerde `DATA` yerine `FINAL` tercih et. Niyeti iletir ve kazara döngü içi atamaları önler.

---

## 7.2 VALUE #( ) ve CORRESPONDING #( ) — Koleksiyon İfadeleri

### 1️⃣ Benzetme

`VALUE #( )`, ABAP'ın koleksiyon/struct başlatıcısıdır — C#'taki nesne başlatıcı `new T { ... }` veya Python'daki sözlük değişmezi `{"key": "val"}` gibi. Geçici değişkenler olmadan satır içi bir değer (bir structure veya tablo) oluşturur.

`CORRESPONDING #( )`, aynı alan adlarını paylaşan iki structure veya tablo arasında alanları kopyalar — C#'taki AutoMapper veya Python'daki `{**dict1, **dict2}` tarzı birleştirme gibi.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# nesne başlatıcı
var order = new Order
{
    OrderId    = "ORD1",
    CustomerId = "CUST01",
    Amount     = 100m,
    Currency   = "EUR"
};

// C# koleksiyon başlatıcı
var orders = new List<Order>
{
    new Order { OrderId = "ORD1", CustomerId = "C1", Amount = 100m },
    new Order { OrderId = "ORD2", CustomerId = "C1", Amount = 200m },
};

// AutoMapper / BeanUtils — eşleşen alanları kopyala
var dto = mapper.Map<OrderDto>(order);
```

```python
# Python struct/dict başlatma
order = {"order_id": "ORD1", "customer_id": "CUST01", "amount": 100.0}

orders = [
    {"order_id": "ORD1", "customer_id": "C1", "amount": 100.0},
    {"order_id": "ORD2", "customer_id": "C1", "amount": 200.0},
]

# Sözlükler arasında eşleşen alanları kopyala (basit sürüm)
dto = {k: order[k] for k in ("order_id", "amount")}
```

### 3️⃣ ABAP'taki karşılığı — VALUE #( )

```abap
" VALUE #( ) bir structure için — kurucu ifade
DATA(ls_order) = VALUE zorders(
  order_id    = 'ORD1'
  kunnr       = 'CUST01'
  amount      = 100
  waers       = 'EUR'
).

" VALUE #( ) bir internal table için — satır listesi
DATA(lt_orders) = VALUE zorders_tab(
  ( order_id = 'ORD1'  kunnr = 'CUST01'  amount = 100  waers = 'EUR' )
  ( order_id = 'ORD2'  kunnr = 'CUST01'  amount = 200  waers = 'EUR' )
  ( order_id = 'ORD3'  kunnr = 'CUST02'  amount =  50  waers = 'USD' )
).

" VALUE #( ) — # "bağlamdan tipi çıkar" anlamına gelir
" Tip biliniyorsa (ör. DATA bildiriminden), # kullan:
DATA lt_known TYPE STANDARD TABLE OF zorders WITH EMPTY KEY.
lt_known = VALUE #(
  ( order_id = 'ORD4'  kunnr = 'CUST03'  amount = 300 )
).

" VALUE #( BASE ... ) ile mevcut tabloya satır ekle
lt_known = VALUE #( BASE lt_known   " mevcut satırları koru
  ( order_id = 'ORD5'  kunnr = 'CUST04'  amount = 400 )
).

" Bir structure'ı başlangıç değerlerine başlat / temizle
CLEAR ls_order.           " eski yol (hâlâ geçerli)
ls_order = VALUE #( ).    " modern yol — açık boş oluşturma
```

**CORRESPONDING #( ) — alan alan kopyalama:**

```abap
" Bir structure'dan diğerine eşleşen alanları kopyala
" (Yalnızca aynı adı VE uyumlu tipe sahip alanlar kopyalanır)

DATA ls_bapi_order TYPE bapiordhdr.   " BAPI header structure'ı
DATA ls_my_order   TYPE zorders.      " özel structure'm

ls_my_order-order_id = 'ORD1'.
ls_my_order-kunnr    = 'CUST01'.
ls_my_order-amount   = 500.

" CORRESPONDING adların eşleştiği alanları kopyalar
ls_bapi_order = CORRESPONDING #( ls_my_order ).
" → yalnızca HER İKİ structure'da da aynı adla mevcut alanları kopyalar

" Tablolar için CORRESPONDING:
DATA lt_bapi_orders TYPE STANDARD TABLE OF bapiordhdr WITH EMPTY KEY.
lt_bapi_orders = CORRESPONDING #( lt_orders ).

" MAPPING ile CORRESPONDING — kopyalama sırasında alanları yeniden adlandır
ls_bapi_order = CORRESPONDING #( ls_my_order
  MAPPING kunnr = kunnr_ext   " ls_my_order-kunnr_ext → ls_bapi_order-kunnr eşle
).

" EXCEPT ile CORRESPONDING — belirli alanları hariç tut
ls_bapi_order = CORRESPONDING #( ls_my_order
  EXCEPT amount waers   " bu alanları kopyalama
).
```

> ⚠️ **C#/Python tuzağı:** `CORRESPONDING`, büyük structure'larla kullanımda cazip ama tehlikelidir — eşleştiremediği alanları sessizce atlar. Her iki structure'da da bir alan adı değişirse, kopyalama o alan için sessizce durur. Dikkatli kullan; ne eşlediğini belgele.

---

## 7.3 String Şablonları — |Merhaba { lv_name }|

### 1️⃣ Benzetme

Klasik ABAP string birleştirme can sıkıcıydı — `CONCATENATE str1 str2 INTO result`. ABAP 7.4, string şablonları tanıttı: `{ değişken }` yer tutucularıyla `|...|` içine sarılmış satır içi ifadeler. Bu C#'taki `$"..."` enterpolasyonu veya Python'daki `f"..."` f-string'lerdir.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# string enterpolasyonu
string name = "Acme GmbH";
decimal amount = 12500.50m;
string msg = $"Fatura: {name}: EUR {amount:N2}";
// → "Fatura: Acme GmbH: EUR 12,500.50"
```

```python
# Python f-string
name   = "Acme GmbH"
amount = 12500.50
msg    = f"Fatura: {name}: EUR {amount:,.2f}"
# → "Fatura: Acme GmbH: EUR 12,500.50"
```

### 3️⃣ ABAP'taki karşılığı

```abap
DATA lv_name   TYPE string VALUE 'Acme GmbH'.
DATA lv_amount TYPE p DECIMALS 2 VALUE '12500.50'.

" Basit string şablonu
DATA(lv_msg) = |Fatura: { lv_name }: EUR { lv_amount }|.

" { } içindeki biçimlendirme seçenekleri
DATA(lv_date_str) = |Bugün: { sy-datum DATE = USER }|.
" DATE = USER → kullanıcının tarih formatı ayarlarına göre biçimlendirir
" DATE = ISO  → YYYY-MM-DD
" DATE = RAW  → YYYYMMDD (ABAP iç formatı)

DATA(lv_num_str) = |Tutar: { lv_amount NUMBER = USER }|.
" NUMBER = USER → kullanıcı yerel ayarına göre binlik ayırıcıyla biçimlendirir

" Zincirleme / çok satırlı string'ler
DATA(lv_body) = |Sayın { lv_name },| &&
                |Sipariş toplamınız: { lv_amount }.| &&
                |Teşekkürler.|.
" && string birleştirmedir (eski CONCATENATE'in yerini alır)

" Şablon içinde koşullu (ABAP 7.56+)
DATA(lv_status_text) = |Durum: { COND #( WHEN lv_active = 'X' THEN 'Aktif' ELSE 'Pasif' ) }|.

" Klasik ABAP birleştirme — eski kodda görürsün
DATA lv_old_msg TYPE string.
CONCATENATE 'Fatura: ' lv_name ': EUR ' lv_amount
  INTO lv_old_msg SEPARATED BY space.   " hantal, modern kodda kullanılmaz
```

**String işlemleri (modern):**

```abap
" Uzunluk
DATA(lv_len) = strlen( lv_name ).           " .Length gibi

" İçerir / alt string bul
IF lv_name CS 'GmbH'.                       " CS = Contains String
  " ...
ENDIF.

" Değiştir
DATA(lv_clean) = replace( val = lv_name  sub = ' GmbH'  with = '' ).

" Büyük/küçük harfe çevir
DATA(lv_upper) = to_upper( lv_name ).
DATA(lv_lower) = to_lower( lv_name ).

" Kırp
DATA(lv_trimmed) = condense( lv_name ).     " baştaki/sondaki boşlukları kaldırır
```

---

## 7.4 Modern SELECT — Satır İçi INTO, Single, Toplamlar, @ Host Değişkenleri

### 1️⃣ Benzetme

ABAP 7.4+, Open SQL'i standart SQL söz dizimine çok daha yakın hale getirdi: satır içi `INTO DATA(...)`, `@` host değişkeni ön eki, alt sorgular, ortak tablo ifadeleri ve CAST. Modern SAP sistemlerinde yazmak istediğin SQL budur.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// LINQ / EF Core
var orders = await db.Orders
    .Where(o => o.CustomerId == customerId && o.Amount > minAmount)
    .Select(o => new { o.OrderId, o.Amount, o.Currency })
    .OrderByDescending(o => o.Amount)
    .ToListAsync();

var total = await db.Orders
    .Where(o => o.Currency == "EUR")
    .SumAsync(o => o.Amount);
```

```python
# SQLAlchemy 2.0 stili
stmt = (
    select(Order.order_id, Order.amount, Order.currency)
    .where(Order.customer_id == customer_id, Order.amount > min_amount)
    .order_by(Order.amount.desc())
)
orders = session.execute(stmt).all()

total = session.execute(
    select(func.sum(Order.amount)).where(Order.currency == "EUR")
).scalar()
```

### 3️⃣ ABAP'taki karşılığı

```abap
DATA lv_customer_id TYPE kunnr  VALUE 'CUST0001'.
DATA lv_min_amount  TYPE wrbtr  VALUE 500.

" Tam modern SELECT — @ host değişkenleri, satır içi INTO, ORDER BY
SELECT order_id, kunnr, amount, waers
  FROM zorders
  INTO TABLE @DATA(lt_orders)
  WHERE kunnr       = @lv_customer_id
    AND amount      > @lv_min_amount
  ORDER BY amount DESCENDING.

" SELECT SINGLE — birincil anahtar arama, satır içi
SELECT SINGLE name1, land1, ort01
  FROM kna1
  INTO @DATA(ls_addr)
  WHERE kunnr = @lv_customer_id.

IF sy-subrc <> 0.
  RAISE EXCEPTION TYPE zcx_not_found.
ENDIF.

" Toplama
SELECT waers,
       SUM( amount )   AS total_amount,
       COUNT( * )      AS order_count,
       MAX( amount )   AS max_order,
       AVG( amount )   AS avg_order
  FROM zorders
  INTO TABLE @DATA(lt_totals)
  WHERE kunnr = @lv_customer_id
  GROUP BY waers
  HAVING SUM( amount ) > 1000
  ORDER BY total_amount DESCENDING.

LOOP AT lt_totals INTO FINAL(ls_t).
  WRITE: / ls_t-waers, ls_t-total_amount, ls_t-order_count.
ENDLOOP.

" JOIN — modern söz diziminde daha temiz
SELECT o~order_id,
       o~amount,
       o~waers,
       c~name1   AS customer_name,
       c~land1   AS country
  FROM zorders   AS o
  INNER JOIN kna1 AS c  ON  o~kunnr  = c~kunnr
                        AND o~mandt  = c~mandt    " açık MANDT join
  INTO TABLE @DATA(lt_enriched)
  WHERE o~kunnr = @lv_customer_id.

" CAST — SQL'de tip dönüştür (ABAP 7.5+)
SELECT order_id,
       CAST( amount AS DEC( 15,2 ) ) AS amount_dec
  FROM zorders
  INTO TABLE @DATA(lt_cast)
  WHERE kunnr = @lv_customer_id.

" UP TO N ROWS — sayfalama / ilk-N
SELECT *
  FROM zorders
  INTO TABLE @DATA(lt_latest)
  ORDER BY erdat DESCENDING
  UP TO 10 ROWS.
```

> 💡 **`SELECT SINGLE` vs `SELECT ... UP TO 1 ROWS`:** `SELECT SINGLE`'ı yalnızca WHERE cümlecikinin tam olarak bir satır döndürdüğünü bildiğinde (birincil anahtar) kullan. "Birçok satırdan en üstteki" için `ORDER BY` ile `UP TO 1 ROWS` kullan — semantik olarak açıktır ve optimizer bunu iyi işler.

**Alt Sorgular ve Ortak Tablo İfadeleri (ABAP 7.55+):**

```abap
" WHERE'de alt sorgu
SELECT order_id, amount
  FROM zorders
  INTO TABLE @DATA(lt_vip_orders)
  WHERE kunnr IN ( SELECT kunnr FROM zkna1_vip WHERE vip_flag = 'X' ).

" CTE (Common Table Expression) ile — ABAP 7.55+
WITH
  +vip_customers AS (
    SELECT kunnr FROM zkna1_vip WHERE vip_flag = 'X'
  ),
  +vip_orders AS (
    SELECT o~order_id, o~amount, o~kunnr
      FROM zorders AS o
      INNER JOIN +vip_customers AS v ON o~kunnr = v~kunnr
  )
SELECT *
  FROM +vip_orders
  INTO TABLE @DATA(lt_result).
```

---

## 7.5 Tablo İfadeleri, FILTER, REDUCE ve FOR — ABAP'ın LINQ Cevabı

### 1️⃣ Benzetme

ABAP 7.4, internal table'lar için C#'taki LINQ'ye ve Python'daki liste kavrama ifadelerine neredeyse 1:1 eşleşen fonksiyonel operatörler tanıttı. Modern programlamaya en çok benzeyen kısım budur.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// LINQ karşılıkları
var allOrders = new List<Order> { ... };

// Tek öğe arama
var order = allOrders.Single(o => o.OrderId == "ORD1");

// Yeni listeye filtrele
var eurOrders = allOrders.Where(o => o.Currency == "EUR").ToList();

// Dönüştür / yansıt
var amounts   = allOrders.Select(o => o.Amount).ToList();

// Topla
var total = allOrders.Sum(o => o.Amount);
var max   = allOrders.Max(o => o.Amount);

// Dönüşümle yeni listeye topla
var summaries = allOrders
    .Select(o => new OrderSummary(o.OrderId, o.Amount * 1.1m))
    .ToList();
```

```python
orders = [...]   # dict listesi veya dataclass örnekleri

# Filtrele
eur_orders = [o for o in orders if o["currency"] == "EUR"]

# Dönüştür
amounts = [o["amount"] for o in orders]

# Topla
total = sum(o["amount"] for o in orders)

# Filtrele + dönüştür
summaries = [
    {"id": o["order_id"], "adjusted": o["amount"] * 1.1}
    for o in orders
]
```

### 3️⃣ ABAP'taki karşılığı

**Tablo ifadeleri — itab[ key = value ]:**

```abap
" Anahtara göre doğrudan satır erişimi — tek aramalar için READ TABLE'ın yerini alır
" Sözlük[] erişimi gibi çalışır — bulunamazsa cx_sy_itab_line_not_found fırlatır

" Standard table'dan satıra erişim (anahtar arama)
DATA(ls_order) = lt_orders[ order_id = 'ORD1' ].   " bulunamazsa exception

" VALUE operatörüyle güvenli erişim (bulunamazsa başlangıç değeri döner, exception yok)
DATA(ls_safe) = VALUE #( lt_orders[ order_id = 'ORD1' ] OPTIONAL ).

" Tablo satırından tek alan oku
DATA(lv_amount) = lt_orders[ order_id = 'ORD1' ]-amount.

" Alanı satır içi değiştir (anahtar tanımı olmayan standard table'lar için)
lt_orders[ order_id = 'ORD1' ]-amount = 999.
```

**FILTER — internal table'lar için WHERE:**

```abap
" FILTER yalnızca eşleşen satırları içeren yeni bir tablo oluşturur
" LINQ Where() veya koşullu Python liste kavrama gibi

" Yalnızca EUR siparişlerine filtrele
DATA(lt_eur) = FILTER #( lt_orders WHERE waers = 'EUR' ).

" Karmaşık koşullarla filtrele (anahtarda sorted/hashed table gerektirir)
DATA(lt_big) = FILTER #( lt_orders USING KEY primary_key WHERE amount > 500 ).

" Eski yol ile karşılaştır:
DATA lt_eur_old TYPE STANDARD TABLE OF zorders WITH EMPTY KEY.
LOOP AT lt_orders INTO DATA(ls_o) WHERE waers = 'EUR'.
  APPEND ls_o TO lt_eur_old.
ENDLOOP.
" → FILTER bu döngünün tamamının yerini alır
```

**REDUCE — skaler'e topla:**

```abap
" REDUCE bir tablodan tek bir değer hesaplar
" LINQ Aggregate() / Sum() veya Python functools.reduce() / sum() gibi

" Tüm tutarları topla
DATA(lv_total) = REDUCE wrbtr(
  INIT sum = VALUE wrbtr( )
  FOR  row IN lt_orders
  NEXT sum = sum + row-amount
).

" Koşul eşleşen satırları say
DATA(lv_eur_count) = REDUCE i(
  INIT cnt = 0
  FOR  row IN lt_orders WHERE ( waers = 'EUR' )
  NEXT cnt = cnt + 1
).

" Birleştirilmiş string oluştur (String.Join gibi)
DATA(lv_ids) = REDUCE string(
  INIT result = ``
  FOR  row IN lt_orders
  NEXT result = COND #(
    WHEN result IS INITIAL
    THEN row-order_id
    ELSE result && ', ' && row-order_id
  )
).
" → "ORD1, ORD2, ORD3"
```

**FOR — tablo kavrama (Python liste kavrama veya LINQ Select gibi):**

```abap
" FOR başka bir tabloyu iterleyerek yeni bir tablo oluşturur
" LINQ Select() veya Python [ifade for öğe in liste] gibi

" Yansıt: yalnızca ID'ler ve tutarlardan oluşan tablo oluştur (para birimi yok)
TYPES: BEGIN OF ty_summary,
         order_id TYPE zorders-order_id,
         amount   TYPE zorders-amount,
       END OF ty_summary.

DATA(lt_summary) = VALUE ty_summary_tab(
  FOR row IN lt_orders
  ( order_id = row-order_id  amount = row-amount )
).

" WHERE koşullu FOR (birleşik filtrele + yansıt)
DATA(lt_eur_summary) = VALUE ty_summary_tab(
  FOR row IN lt_orders WHERE ( waers = 'EUR' )
  ( order_id = row-order_id  amount = row-amount )
).

" Dönüşümlü FOR
DATA(lt_with_tax) = VALUE ty_summary_tab(
  FOR row IN lt_orders
  ( order_id = row-order_id  amount = row-amount * '1.19' )
).
```

**COND #( ) — satır içi koşullu (üçlü operatör):**

```abap
" COND #( WHEN ... THEN ... WHEN ... THEN ... ELSE ... )
" C#'taki ?:  veya  Python'daki (a if cond else b) gibi

DATA(lv_label) = COND string(
  WHEN lv_amount > 10000 THEN 'Büyük'
  WHEN lv_amount > 1000  THEN 'Orta'
  ELSE                        'Küçük'
).

" Klasik ile karşılaştır:
DATA lv_old_label TYPE string.
IF lv_amount > 10000.
  lv_old_label = 'Büyük'.
ELSEIF lv_amount > 1000.
  lv_old_label = 'Orta'.
ELSE.
  lv_old_label = 'Küçük'.
ENDIF.
```

**SWITCH #( ) — ifadeler için daha temiz CASE/WHEN:**

```abap
" SWITCH #( ) ifade olarak CASE'dir
DATA(lv_currency_symbol) = SWITCH string( lv_currency
  WHEN 'EUR' THEN '€'
  WHEN 'USD' THEN '$'
  WHEN 'GBP' THEN '£'
  ELSE            lv_currency
).
```

---

## 7.6 Büyük Önce/Sonra — Klasikten Moderne Yeniden Yazım

Gerçekçi, çirkin bir klasik ABAP parçasını alalım ve yeniden yazalım. İş hayatında bir kod tabanını modernize ederken gerçekten yapacağın dönüşüm budur.

### Klasik ABAP (2005 civarı tarz)

```abap
" Klasik: ayrıntılı, her yerde geçici değişken, satır içi hiçbir şey yok
METHOD get_customer_totals.

  DATA: lt_orders   TYPE STANDARD TABLE OF zorders,
        ls_order    TYPE zorders,
        lt_result   TYPE STANDARD TABLE OF ty_cust_total,
        ls_result   TYPE ty_cust_total,
        lv_kunnr    TYPE kunnr,
        lv_found    TYPE i,
        lv_total    TYPE wrbtr.

  lv_kunnr = iv_customer.

  SELECT * FROM zorders INTO TABLE lt_orders
    WHERE kunnr = lv_kunnr AND waers = 'EUR'.

  IF sy-subrc <> 0.
    RETURN.
  ENDIF.

  CLEAR lv_total.
  LOOP AT lt_orders INTO ls_order.
    lv_total = lv_total + ls_order-amount.
  ENDLOOP.

  READ TABLE lt_result INTO ls_result
    WITH KEY kunnr = lv_kunnr.
  lv_found = sy-subrc.

  IF lv_found <> 0.
    CLEAR ls_result.
    ls_result-kunnr  = lv_kunnr.
    ls_result-total  = lv_total.
    ls_result-waers  = 'EUR'.
    APPEND ls_result TO lt_result.
  ELSE.
    ls_result-total = lv_total.
    MODIFY lt_result FROM ls_result
      TRANSPORTING total
      WHERE kunnr = lv_kunnr.
  ENDIF.

  et_result = lt_result.
ENDMETHOD.
```

### Modern ABAP (7.4+ ile yeniden yazım)

```abap
" Modern: satır içi bildirimler, VALUE #(), REDUCE, tablo ifadeleri
METHOD get_customer_totals.

  " Tek SELECT, satır içi tip
  SELECT order_id, kunnr, amount, waers
    FROM zorders
    INTO TABLE @DATA(lt_orders)
    WHERE kunnr = @iv_customer
      AND waers = 'EUR'.

  CHECK sy-subrc = 0.   " CHECK, koşul false ise metottan çıkar

  " REDUCE, biriktirici döngünün yerini alır
  DATA(lv_total) = REDUCE wrbtr(
    INIT sum = VALUE wrbtr( )
    FOR  row IN lt_orders
    NEXT sum = sum + row-amount
  ).

  " Tablo ifadesi + VALUE #( ), READ/MODIFY/APPEND mantığının yerini alır
  IF lt_result[ kunnr = iv_customer ] IS INITIAL.           " bulunamadı
    INSERT VALUE #( kunnr = iv_customer  total = lv_total  waers = 'EUR' )
      INTO TABLE et_result.
  ELSE.
    et_result[ kunnr = iv_customer ]-total = lv_total.      " doğrudan alan set
  ENDIF.

ENDMETHOD.
```

Modern sürüm daha kısa, daha ifadeli ve niyeti doğrudan iletiyor. `lv_found` geçici değişkeni yok. `CLEAR` yok. `APPEND/MODIFY` dansı yok. Sadece: seç → topla → upsert.

**Karşılaştırma için C# karşılığı:**

```csharp
public async Task<List<CustomerTotal>> GetCustomerTotals(string customerId)
{
    var orders = await db.Orders
        .Where(o => o.CustomerId == customerId && o.Currency == "EUR")
        .ToListAsync();

    if (!orders.Any())
        return new List<CustomerTotal>();

    var total = orders.Sum(o => o.Amount);

    var result = new List<CustomerTotal>();
    var existing = result.FirstOrDefault(r => r.CustomerId == customerId);
    if (existing == null)
        result.Add(new CustomerTotal(customerId, total, "EUR"));
    else
        existing.Total = total;

    return result;
}
```

```python
def get_customer_totals(customer_id: str) -> list[dict]:
    orders = [o for o in fetch_orders() if o["kunnr"] == customer_id and o["waers"] == "EUR"]
    if not orders:
        return []

    total = sum(o["amount"] for o in orders)

    existing = next((r for r in result if r["kunnr"] == customer_id), None)
    if existing is None:
        result.append({"kunnr": customer_id, "total": total, "waers": "EUR"})
    else:
        existing["total"] = total

    return result
```

Modern ABAP sürümü yapısal olarak hem C# hem de Python sürümlerine çok yakın. Kavramlar doğrudan eşleşiyor — yalnızca ABAP yazımlarını hatırlaman gerekiyor.

---

## 7.7 Hızlı Başvuru — Klasik vs Modern Yan Yana

| Ne istiyorsun | Klasik ABAP | Modern ABAP (7.4+) | C# karşılığı |
|---|---|---|---|
| Değişken bildir | `DATA lv_x TYPE i.` | `DATA(lv_x) = 0.` | `var x = 0;` |
| Değişmez bildir | (karşılığı yok) | `FINAL(lv_x) = 0.` | `const int x = 0;` |
| Struct değişmezi | `ls-field = val. ...` | `VALUE ty( field = val )` | `new T { Field = val }` |
| Tablo değişmezi | `APPEND ls TO lt.` | `VALUE #( (f=v) (f=v) )` | `new List<T> { ... }` |
| String enterpolasyonu | `CONCATENATE a b INTO s.` | `\|metin { değişken } daha\|` | `$"metin {değişken} daha"` |
| Üçlü operatör | `IF/ELSE/ENDIF` | `COND #( WHEN ... )` | `koşul ? a : b` |
| CASE ifadesi | `CASE/WHEN/ENDCASE` | `SWITCH #( ... )` | `switch(x) { case ... }` |
| Döngü yinele+oku | `LOOP + READ TABLE` | `itab[ key = val ]` | `list.Single(x=>...)` |
| Tabloyu filtrele | `LOOP + APPEND` | `FILTER #(...)` | `.Where(...)` |
| Topla | `LOOP + biriktir` | `REDUCE #(...)` | `.Sum()/.Aggregate()` |
| Tablo yansıt | `LOOP + APPEND` | `VALUE #( FOR row IN ... )` | `.Select(...)` |
| Eşleşen alanları kopyala | elle alan alan | `CORRESPONDING #(...)` | AutoMapper |

---

## 🧠 Özet

- **`DATA(x)`** değişkenleri satır içi bildirir — her şey için metot başı bildirimleri artık zorunlu değil. `FINAL(x)` onları değişmez yapar.
- **`VALUE #( )`** structure'ları ve tabloları satır içi oluşturur — ABAP nesne/koleksiyon başlatıcısı.
- **`CORRESPONDING #( )`** uyumlu structure'lar arasında adına göre alanları kopyalar — dikkatli kullan.
- **String şablonları** `|Merhaba { isim }|`, `CONCATENATE`'in yerini alır — `DATE = USER` ve `NUMBER = USER` gibi biçim seçenekleri destekler.
- **Modern SELECT**, `@` host değişkenleri, satır içi `INTO @DATA(...)`, toplamalar, JOIN'ler ve CTE'ler kullanır — standart SQL'e yakın.
- **Tablo ifadeleri** `itab[ key = val ]`, çoğu `READ TABLE` çağrısının yerini alır.
- **FILTER / REDUCE / FOR**, doğrudan LINQ / Python kavrama ifadelerine eşleşir: tabloyu filtrele, skaler'e topla, yeni tabloya yansıt.
- **`COND #( )` ve `SWITCH #( )`** satır içi koşullu ifadelerdir — ABAP'ın üçlü operatörleri.
- Modern ABAP, eski söz dizimine kıyasla C# ve Python'a çok daha yakın görünüyor. S/4HANA'da (7.4+), modern ABAP yazmalısın — kodun daha temiz olur ve meslektaşların teşekkür eder.

---

*[← İçindekiler](../content.md) | [← Önceki: ABAP Söz Dizimi Temelleri](06-abap-syntax-basics.md) | [Sonraki: Raporlar: Klasik ve ALV →](08-reports-classical-and-alv.md)*
