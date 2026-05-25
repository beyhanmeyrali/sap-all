# Kısım 24: Arama & Query Seçenekleri

*Koleksiyon endpoint'inizi gerçekten kullanışlı hale getirin: $filter, $search, $top/$skip, $orderby ve $inlinecount — hepsini GET_ENTITYSET içine bağlayın.*

---

## ☕ Bu kısım neden önemli?

Herhangi bir filtre uygulamadan 50.000 malzemenin tamamını döndüren bir GET_ENTITYSET, prototipleme için gayet iyi ama üretimde işe yaramaz. Gerçek Fiori uygulamaları birlikte `$filter`, `$top`, `$skip`, `$orderby` ve `$inlinecount=allpages` gönderir. Sizin işiniz bu OData query seçeneklerini Open SQL koşullarına çevirmek ve bunları doğru şekilde sayfalamaktır.

Bu kısım, kısım 23'teki iskeleti üretime hazır hale getirir. Kısımın sonunda tek bir metot gövdesi beş query mekanizmasının tümünü birden yönetecek.

---

## 24.1 $filter → WHERE koşulu

### 1️⃣ Benzetme

`$filter`, SQL WHERE koşulunun OData karşılığıdır. İstemci bunu URL parametresi olarak gönderir; siz de ABAP'ta önceden ayrıştırılmış bir *select-options* tablosu (SIGN, OPTION, LOW, HIGH) olarak okursunuz — raporlarda ve seçim ekranlarında zaten karşılaştığınız ABAP ranges kavramının ta kendisi.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# Web API — filter sorgu dizesi olarak gelir, IQueryable'a uygularsınız
[HttpGet]
public IEnumerable<Product> GetAll(
    [FromQuery] string? name = null,
    [FromQuery] decimal? minPrice = null,
    [FromQuery] decimal? maxPrice = null)
{
    var query = _db.Products.AsQueryable();
    if (name is not null)
        query = query.Where(p => p.Name.Contains(name));
    if (minPrice is not null)
        query = query.Where(p => p.Price >= minPrice.Value);
    if (maxPrice is not null)
        query = query.Where(p => p.Price <= maxPrice.Value);
    return query.ToList();
}
```

OData'da istemci aynı şeyi URL'de ifade eder:

```http
GET /ProductSet?$filter=Price ge 1000 and Price le 5000 and substringof('Lathe',Name)
```

### 3️⃣ ABAP'taki karşılığı — IT_FILTER_SELECT_OPTIONS'ı okuma

```abap
" IT_FILTER_SELECT_OPTIONS, /IWBEP/T_MGW_SELECT_OPTION tipindedir
" Her satır:
"   PROPERTY     = 'Price'     (OData özellik adı)
"   SELECT_OPTIONS = RANGE OF <abap-tipi>
"       SIGN   = 'I' (dahil et) veya 'E' (hariç tut)
"       OPTION = 'EQ','NE','GT','GE','LT','LE','BT','CP' ...
"       LOW    = '1000.00'
"       HIGH   = '5000.00'  (yalnızca BT — between için)

" Klasik yaklaşım: alana göre ranges'ları çıkar ve SELECT ... WHERE field IN ranges kullan

DATA lt_price_range  TYPE RANGE OF kbetr.
DATA lt_name_range   TYPE RANGE OF maktx.

LOOP AT it_filter_select_options INTO DATA(ls_sel_opt).
  CASE ls_sel_opt-property.
    WHEN 'Price'.
      lt_price_range = CORRESPONDING #( ls_sel_opt-select_options ).
    WHEN 'Name'.
      lt_name_range  = CORRESPONDING #( ls_sel_opt-select_options ).
  ENDCASE.
ENDLOOP.

" Bunları SELECT'te kullanın:
SELECT a~matnr AS product_id,
       t~maktx AS name,
       a~stprs AS price,
       a~waers AS currency,
       a~ersda AS created_on
  INTO CORRESPONDING FIELDS OF TABLE @lt_products
  FROM mara AS a
  LEFT OUTER JOIN makt AS t
    ON  t~matnr = a~matnr
    AND t~spras = @lv_langiso
 WHERE a~lvorm  = space
   AND a~stprs IN @lt_price_range     " ← filter uygulandı!
   AND t~maktx IN @lt_name_range      " ← filter uygulandı!
 ORDER BY a~matnr
 UP TO @lv_top ROWS
 OFFSET @lv_skip ROWS.
```

> 💡 Bir ranges tablosu boşsa (istemci o alanda filter uygulamadıysa), `field IN boş_ranges_tablosu` **her şeyle** eşleşir — ki bu tam olarak istediğiniz şeydir. `IF lt_price_range IS NOT INITIAL` şeklinde özel bir korumaya gerek yoktur.

> ⚠️ **C#/Python tuzağı:** Ranges tablosu satırları SQL değil ABAP SIGN/OPTION gösterimini kullanır. `EQ` = `=`, `NE` = `<>`, `GT` = `>`, `GE` = `>=`, `LT` = `<`, `LE` = `<=`, `BT` = `BETWEEN`. `CP` = içerir kalıbı (`*` joker karakteriyle). OData çerçevesi `$filter=Price ge 1000` ifadesini zaten sizin için SIGN=I, OPTION=GE, LOW='1000' biçimine çevirmiştir.

### Tip güvenli çıkarım için get_filter_select_options() kullanma

Alternatif olarak, aynı veriyi OO arayüzü üzerinden döndüren request context yardımcısını çağırabilirsiniz:

```abap
DATA(lt_filter_so) = io_tech_request_context->get_filter(
    )->get_filter_select_options( ).
```

Bu, `it_filter_select_options` ile içerik olarak özdeştir ama arayüz üzerinden çekilir — ranges'ları çıkarmadan önce sonradan işleme veya günlükleme yapmak istediğinizde kullanışlıdır.

### OData'nın desteklediği $filter operatörleri

| OData operatörü | SQL karşılığı | ABAP OPTION |
|---|---|---|
| `eq` | `=` | `EQ` |
| `ne` | `<>` | `NE` |
| `gt` | `>` | `GT` |
| `ge` | `>=` | `GE` |
| `lt` | `<` | `LT` |
| `le` | `<=` | `LE` |
| `and` | `AND` | Birden fazla ranges satırı (SIGN=I) |
| `or` | `OR` | Birden fazla ranges satırı (SIGN=I) |
| `substringof('x', field)` | `LIKE '%x%'` | OPTION=`CP`, LOW=`*x*` |
| `startswith(field,'x')` | `LIKE 'x%'` | OPTION=`CP`, LOW=`x*` |

---

## 24.2 Serbest metin arama dizesi ($search)

### 1️⃣ Benzetme

`$search=lathe`, bir arama kutusu gibidir — istemci tek bir metin dizesi gönderir ve siz bunu ilgili metin alanlarında aramanızı bekler. SQL WHERE değildir; daha çok hafif bir tam metin aramasıdır. Hangi alanların aranacağına siz karar verirsiniz.

### 2️⃣ Bunu zaten biliyorsun

```python
# FastAPI — istediğiniz şekilde uygulayabileceğiniz bir q= arama parametresi
@app.get("/ProductSet")
def get_products(q: str | None = None):
    query = db.query(ProductModel)
    if q:
        query = query.filter(
            or_(ProductModel.name.ilike(f"%{q}%"),
                ProductModel.product_id.ilike(f"%{q}%"))
        )
    return query.all()
```

### 3️⃣ ABAP'taki karşılığı

```abap
" Serbest metin arama ifadesini oku
DATA(lv_search) = io_tech_request_context->get_search_expression( ).

" Metin alanlarında joker karakterli eşleme olarak uygula
DATA lt_search_range TYPE RANGE OF maktx.

IF lv_search IS NOT INITIAL.
  " İÇERİR tarzı eşleme için joker karakterlerle sar
  APPEND VALUE #(
    sign   = 'I'
    option = 'CP'
    low    = |*{ lv_search }*|    " '*lathe*' → LIKE '%lathe%'
  ) TO lt_search_range.
ENDIF.

" SELECT'e ekle:
"   AND ( t~maktx IN @lt_search_range
"      OR a~matnr IN @lt_search_range_matnr )
```

> ⚠️ **C#/Python tuzağı:** SAP'ın OData çerçevesi `$search`'ü otomatik olarak hiçbir alana bağlamaz — arama mantığını siz implemente edersiniz. Çerçeve ham dizeyi yalnızca `get_search_expression( )` aracılığıyla geçirir. Bu metodu çağırmazsanız `$search` parametresi sessizce görmezden gelinir. Servisinizin hangi alanları aradığını belgelediğinizden emin olun.

---

## 24.3 Sayfalama & sıralama — $top, $skip, $orderby, $inlinecount

### 1️⃣ Benzetme

Bu, herhangi bir REST API'daki sayfalama ile tamamen aynıdır — bunu C# veya Python'da düzinelerce kez yaptınız. `$top` = sayfa boyutu, `$skip` = ofset, `$orderby` = sıralama sütunu ve yönü, `$inlinecount=allpages` = "sayfa numaralarını gösterebilmem için toplam sayıyı da söyle".

### 2️⃣ Bunu zaten biliyorsun

```csharp
// Microsoft.AspNetCore.OData ile ASP.NET — tanıdık kalıp:
[HttpGet]
[EnableQuery(MaxTop = 500)]
public IQueryable<Product> Get() => _db.Products;
// Çerçeve $top/$skip/$orderby'yi otomatik uygular.

// Manuel karşılığı:
public PagedResult<Product> GetPaged(int top, int skip, string orderBy)
{
    var query = _db.Products.AsQueryable();

    query = orderBy switch
    {
        "Price asc"  => query.OrderBy(p => p.Price),
        "Price desc" => query.OrderByDescending(p => p.Price),
        "Name asc"   => query.OrderBy(p => p.Name),
        _            => query.OrderBy(p => p.ProductId)
    };

    return new PagedResult<Product>
    {
        TotalCount = query.Count(),
        Items      = query.Skip(skip).Take(top).ToList()
    };
}
```

### 3️⃣ ABAP'taki karşılığı

#### $top ve $skip (IS_PAGING)

```abap
" IS_PAGING, /IWBEP/S_MGW_PAGING tipindedir
"   IS_PAGING-top   = $top'un tamsayı değeri  (gönderilmediyse 0)
"   IS_PAGING-skip  = $skip'in tamsayı değeri (gönderilmediyse 0)

DATA(lv_top)  = COND i( WHEN is_paging-top  > 0
                         THEN CONV i( is_paging-top )
                         ELSE 100 ).                  " varsayılan sayfa boyutu
DATA(lv_skip) = CONV i( is_paging-skip ).

" SELECT'te uygulama:
" UP TO @lv_top ROWS OFFSET @lv_skip ROWS
```

#### $orderby (IT_ORDER)

```abap
" IT_ORDER, /IWBEP/T_MGW_SORTING_ORDER tipindedir
" Her satır:
"   PROPERTY  = 'Price'
"   ORDER     = /IWBEP/IF_MGW_APPL_SRV_RUNTIME=>GC_SORT_ORDER-ascending
"               veya GC_SORT_ORDER-descending

" IT_ORDER'dan dinamik ORDER BY dizesi oluştur
DATA lv_order_by TYPE string.

LOOP AT it_order INTO DATA(ls_order).
  IF lv_order_by IS NOT INITIAL.
    lv_order_by = lv_order_by && ', '.
  ENDIF.

  DATA(lv_field) = SWITCH string( ls_order-property
    WHEN 'ProductId' THEN 'a~matnr'
    WHEN 'Price'     THEN 'a~stprs'
    WHEN 'Name'      THEN 't~maktx'
    ELSE                  'a~matnr' ).

  DATA(lv_dir) = SWITCH string( ls_order-order
    WHEN /iwbep/if_mgw_appl_srv_runtime=>gc_sort_order-descending THEN ' DESCENDING'
    ELSE ' ASCENDING' ).

  lv_order_by = lv_order_by && lv_field && lv_dir.
ENDLOOP.

IF lv_order_by IS INITIAL.
  lv_order_by = 'a~matnr ASCENDING'.   " varsayılan sıralama
ENDIF.
```

> ⚠️ **C#/Python tuzağı:** ABAP Open SQL, eski sürümlerde `ORDER BY` koşulunda düz bir string değişkeni **desteklemez**. Gerçek anlamda dinamik sıralama için iki seçenek vardır: (1) `cl_abap_dyn_prog=>check_column_name( )` / `adbc` dinamik SQL yaklaşımı ya da (2) her şeyi SELECT edip ardından iç tabloyu `SORT it_result BY (lv_fieldname)` ile sıralamak. 2. seçenek daha güvenlidir ve pratikte yaygındır — bunun maliyeti, `UP TO N ROWS` sonrası sıralama yaptığınız için sayfalama + sıralamayı dikkatli yönetmeniz gerektiğidir (mümkünse önce sıralayın veya dinamik ORDER BY'nin tam tarama gerektirdiğini kabul edin, ardından manuel dilim alın).

#### $inlinecount=allpages

```abap
" Ana SELECT'ten sonra COUNT(*) çalıştırın ve atayın:
IF io_tech_request_context->get_inlinecount( ) = abap_true.
  SELECT COUNT(*) INTO @DATA(lv_total)
    FROM mara AS a
    LEFT OUTER JOIN makt AS t
      ON  t~matnr = a~matnr
      AND t~spras = @lv_langiso
   WHERE a~lvorm = space
     AND a~stprs IN @lt_price_range
     AND t~maktx IN @lt_name_range.

  es_response_context-inlinecount = lv_total.
ENDIF.
```

İstemci JSON'da `__count` alanını alır:

```json
{
  "d": {
    "__count": "342",
    "results": [ ... ]
  }
}
```

Fiori SmartTable bunu "342 öğe (1-25 gösteriliyor)" biçiminde render etmek için kullanır.

---

## 24.4 Tüm query seçenekleriyle tam GET_ENTITYSET

İşte beş query mekanizmasının tümünü birlikte yöneten, üretime hazır `PRODUCTSET_GET_ENTITYSET`:

```abap
METHOD productset_get_entityset.
  "======================================================================
  " GET /ProductSet
  " Desteklenenler: $filter (Price, Name), $search, $top, $skip,
  "                 $orderby (ProductId, Price, Name), $inlinecount=allpages
  "======================================================================

  CONSTANTS c_default_page TYPE i VALUE 100.
  CONSTANTS c_max_page      TYPE i VALUE 500.

  " ── Dil ──────────────────────────────────────────────────────────────
  DATA(lv_langiso) = cl_abap_syst=>get_language( ).

  " ── Sayfalama ─────────────────────────────────────────────────────────
  DATA(lv_top) = COND i(
      WHEN is_paging-top > 0 AND is_paging-top <= c_max_page
      THEN CONV i( is_paging-top )
      WHEN is_paging-top > c_max_page
      THEN c_max_page
      ELSE c_default_page ).
  DATA(lv_skip) = CONV i( is_paging-skip ).

  " ── Filter — filtrelenebilir özellik başına ranges çıkar ──────────────
  DATA lt_price_range  TYPE RANGE OF kbetr.
  DATA lt_name_range   TYPE RANGE OF maktx.
  DATA lt_matno_range  TYPE RANGE OF matnr.

  LOOP AT it_filter_select_options INTO DATA(ls_sel_opt).
    CASE ls_sel_opt-property.
      WHEN 'Price'.
        lt_price_range = CORRESPONDING #( ls_sel_opt-select_options ).
      WHEN 'Name'.
        lt_name_range  = CORRESPONDING #( ls_sel_opt-select_options ).
      WHEN 'ProductId'.
        lt_matno_range = CORRESPONDING #( ls_sel_opt-select_options ).
    ENDCASE.
  ENDLOOP.

  " ── Serbest metin arama ───────────────────────────────────────────────
  DATA lt_search_name_range TYPE RANGE OF maktx.
  DATA(lv_search) = io_tech_request_context->get_search_expression( ).

  IF lv_search IS NOT INITIAL.
    " Name alanında ara (diğer metin alanlarına genişletilebilir)
    APPEND VALUE #(
        sign   = 'I'
        option = 'CP'
        low    = |*{ lv_search }*|
    ) TO lt_search_name_range.
  ELSE.
    " Arama yoksa geçir (boş range = SELECT'teki tüm satırlarla eşleşir)
    " Yapılacak bir şey yok — lt_search_name_range boş kalır
  ENDIF.

  " ── Ana SELECT ────────────────────────────────────────────────────────
  DATA lt_products TYPE TABLE OF zcl_zmy_product_srv_mpc=>ts_product
       WITH EMPTY KEY.

  " Not: arama VE ad filter'ı burada AND mantığıyla çalışır
  " (her ikisi de gönderildiyse AND uygularız — iş kuralınıza göre ayarlayın)
  SELECT a~matnr AS product_id,
         t~maktx AS name,
         a~stprs AS price,
         a~waers AS currency,
         a~ersda AS created_on
    INTO CORRESPONDING FIELDS OF TABLE @lt_products
    FROM mara AS a
    LEFT OUTER JOIN makt AS t
      ON  t~matnr = a~matnr
      AND t~spras = @lv_langiso
   WHERE a~lvorm = space
     AND a~matnr IN @lt_matno_range
     AND a~stprs IN @lt_price_range
     AND (   t~maktx IN @lt_name_range
          OR t~maktx IN @lt_search_name_range ).

  " ── Sıralama (SELECT sonrası — güvenli yaklaşım) ──────────────────────
  IF it_order IS NOT INITIAL.
    DATA(ls_first_order) = it_order[ 1 ].

    DATA(lv_sort_field) = SWITCH string( ls_first_order-property
        WHEN 'ProductId' THEN 'PRODUCT_ID'
        WHEN 'Price'     THEN 'PRICE'
        WHEN 'Name'      THEN 'NAME'
        ELSE                  'PRODUCT_ID' ).

    DATA(lv_sort_descending) = xsdbool(
        ls_first_order-order =
        /iwbep/if_mgw_appl_srv_runtime=>gc_sort_order-descending ).

    SORT lt_products BY (lv_sort_field) ASCENDING.  " varsayılan artan

    IF lv_sort_descending = abap_true.
      SORT lt_products BY (lv_sort_field) DESCENDING.
    ENDIF.
  ELSE.
    " Varsayılan: ProductId'ye göre artan sırala
    SORT lt_products BY product_id ASCENDING.
  ENDIF.

  " ── Sayfalama dilimi (sıralama SELECT sonrası olduğu için) ─────────────
  DATA(lv_total_before_page) = lines( lt_products ).

  " Dilim: ilk lv_skip satırı atla, ardından lv_top satır al
  DATA lt_paged LIKE lt_products.
  DATA(lv_start) = lv_skip + 1.
  DATA(lv_end)   = lv_skip + lv_top.

  LOOP AT lt_products INTO DATA(ls_prod)
      FROM lv_start TO lv_end.
    APPEND ls_prod TO lt_paged.
  ENDLOOP.

  " ── Veriyi döndür ─────────────────────────────────────────────────────
  ct_data = CORRESPONDING #( lt_paged ).

  " ── Satır içi sayı (sayfalama diliminden ÖNCE, filterdan SONRA toplam) ─
  IF io_tech_request_context->get_inlinecount( ) = abap_true.
    es_response_context-inlinecount = lv_total_before_page.
  ENDIF.

ENDMETHOD.
```

### Değiş tokuş: SELECT-sonra-sırala vs. SQL'de ORDER BY

Yukarıdaki implementasyon ABAP'ta SELECT sonrasında sıralar. Bu güvenli, taşınabilir yaklaşımdır — dinamik SQL riski yoktur. Dezavantajı: çok büyük tablolarda dilimleme öncesinde gereğinden fazla satır çekilir.

Pratikte üretim sistemlerinde:

```
küçük-orta tablolar (< 50.000 satır)  → ABAP'ta sırala, ABAP'ta dilimle — sorun yok
büyük tablolar (> 100.000 satır)       → Sütun doğrulaması için cl_abap_dyn_prog kullanan
                                          dinamik SQL ile ORDER BY'yi SELECT'e aktar
```

SEGW/SEGW eğitim kapsamı için çoğu projede ABAP sıralama yaklaşımını görürsünüz.

> 🧭 **İş hayatında:** Fiori SmartTable ilk yüklenmede her zaman `$top=30&$skip=0&$inlinecount=allpages`, aşağı kaydırmada ise `$top=30&$skip=30` gönderir. Sayfalamayı yanlış implemente ederseniz (inlinecount döndürmeyi unutmak veya sıralama öncesinde `UP TO N ROWS` uygulamak), kullanıcılar 2. sayfada tekrar eden satırlar ya da tablo başlığında yanlış toplamlar görür. Bu hataları `/IWFND/GW_CLIENT`'ta 5'lik sayfa boyutuyla hızlıca yakalamak için her zaman o ayarla test edin.

---

### Tam metodu test etmek için örnek URL'ler

```http
" Tüm ürünler, varsayılan sayfalama:
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet?$format=json

" Fiyat aralığına göre filter:
GET /ProductSet?$filter=Price ge 1000 and Price le 5000&$format=json

" Serbest metin arama:
GET /ProductSet?$search=lathe&$format=json

" Sıralama ve sayı ile sayfalama:
GET /ProductSet?$top=10&$skip=0&$orderby=Price desc&$inlinecount=allpages&$format=json

" Birleştirilmiş — filter + arama + sayfalama:
GET /ProductSet?$filter=Price ge 500&$search=mill&$top=5&$skip=0
    &$orderby=Name asc&$inlinecount=allpages&$format=json
```

---

## 🧠 Özet

- **`IT_FILTER_SELECT_OPTIONS`**, alan başına ABAP ranges tabloları verir; bunları doğrudan `SELECT WHERE field IN range` içine bırakabilirsiniz. Boş range = her şeyle eşleşir — korumaya gerek yok.
- **`$search`**, `io_tech_request_context->get_search_expression( )` aracılığıyla düz bir string olarak gelir. Hangi alanlar üzerinde joker eşleme yapacağınıza siz karar verirsiniz.
- **`IS_PAGING`**, `-top` ve `-skip` alanlarına sahiptir; bunları `UP TO n ROWS OFFSET m ROWS` (ABAP 7.50+) veya eski sistemler için SELECT sonrası döngü dilimleme ile kullanın.
- **`IT_ORDER`**, özellik + yön bilgisi verir. En güvenli yaklaşım: bellek içi iç tabloyu `SORT lt BY (dinamik_alan_adı) ASCENDING/DESCENDING` ile sıralamak.
- **`$inlinecount=allpages`** → `io_tech_request_context->get_inlinecount( )` ile kontrol edin, ardından `es_response_context-inlinecount`'u filterdan *sonra* sayfalamadan *önce* toplam satır sayısına ayarlayın.
- Tam metot gövdesi beş mekanizmanın tümünü bir arada yönetir — her biri toplamsal ve bağımsızdır.

---

*[← İçindekiler](../content.md) | [← Önceki: Veri Okuma: GET_ENTITY & GET_ENTITYSET](23-odata-read-crud-get-entity.md) | [Sonraki: Create, Update & Delete →](25-odata-create-update-delete.md)*
