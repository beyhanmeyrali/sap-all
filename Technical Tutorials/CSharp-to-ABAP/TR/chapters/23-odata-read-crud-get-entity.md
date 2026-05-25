# Kısım 23: Veri Okuma: GET_ENTITY & GET_ENTITYSET

*Product entity'si için tam çalışan, salt okunur bir OData servisi oluşturma — SEGW modelinden canlı JSON yanıtına kadar.*

---

## ☕ Her şeyin yerine oturduğu kısım bu

Kısımlar 21 ve 22 hazırlık niteliğindeydi: söz dizimi, iskele oluşturma, metot imzaları. Bu kısım gerçek ABAP yazdığınız, gerçek bir HTTP isteği attığınız ve gerçek JSON aldığınız ilk kısımdır. `Product` entity'si için okuma yolunu uçtan uca oluşturuyoruz.

Bu kısımın sonunda şunlara sahip olacaksınız:
- `MARA` + `MAKT` üzerinde gerçekçi SELECT'lerle koleksiyon döndüren `PRODUCTSET_GET_ENTITYSET`
- Anahtara göre tek bir ürün döndüren `PRODUCTSET_GET_ENTITY`
- `/IWFND/GW_CLIENT`'ta çalışan bir test URL'si

---

## 23.1 GET_ENTITYSET — koleksiyonu döndürme

### 1️⃣ Benzetme

`GET_ENTITYSET`, Web API'nizdeki `GET /api/products` endpoint'idir. İstemci çağırır, siz bir liste döndürürsünüz. Fark şu ki "controller"ınız yeniden tanımlanmış bir ABAP metodudur ve `IEnumerable<T>` döndürmek yerine bir iç tablo doldurarak veriyi geri verirsiniz.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# Web API
[HttpGet]
public async Task<IEnumerable<ProductDto>> GetAll(
    [FromQuery] int top = 100,
    [FromQuery] int skip = 0)
{
    return await _db.Products
        .Join(_db.ProductDescriptions,
              p => p.ProductId,
              d => d.ProductId,
              (p, d) => new ProductDto
              {
                  ProductId  = p.ProductId,
                  Name       = d.Name,
                  Price      = p.Price,
                  Currency   = p.Currency,
                  CreatedOn  = p.CreatedOn
              })
        .Skip(skip)
        .Take(top)
        .ToListAsync();
}
```

```python
# FastAPI
@app.get("/ProductSet", response_model=list[Product])
def get_products(top: int = 100, skip: int = 0, db: Session = Depends(get_db)):
    return (db.query(ProductModel)
              .join(ProductDescModel)
              .offset(skip)
              .limit(top)
              .all())
```

### 3️⃣ ABAP'taki karşılığı — tam GET_ENTITYSET implementasyonu

`SE24` → `ZMY_PRODUCT_SRV_DPC_EXT` sınıfı → Methods sekmesi → `PRODUCTSET_GET_ENTITYSET`'i bulun → sağ tıklayın **Redefine**.

```abap
METHOD productset_get_entityset.
  "--------------------------------------------------------------------
  " GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet
  " MARA + MAKT birleşiminden ürün listesini döndürür.
  "--------------------------------------------------------------------

  " ── Tipler ──────────────────────────────────────────────────────────
  TYPES: BEGIN OF lty_result,
           product_id TYPE matnr,
           name       TYPE maktx,
           price      TYPE p DECIMALS 2,
           currency   TYPE waers,
           created_on TYPE dats,
         END OF lty_result.

  " ── Yerel veri ─────────────────────────────────────────────────────
  DATA lt_result   TYPE TABLE OF lty_result.
  DATA lv_top      TYPE i.
  DATA lv_skip     TYPE i.
  DATA lv_langiso  TYPE laiso.

  " ── 1. MAKT için dili belirle ────────────────────────────────────────
  lv_langiso = cl_abap_syst=>get_language( ).         " giriş dili

  " ── 2. Sayfalama parametrelerini oku ─────────────────────────────────
  lv_top  = COND #( WHEN is_paging-top  > 0 THEN is_paging-top  ELSE 100 ).
  lv_skip = is_paging-skip.

  " ── 3. SELECT — MARA (başlık) + MAKT (açıklamalar) ──────────────────
  "    Kısım 24'te bu tablonun üstüne filtreler ve sayfalama ekleyeceğiz.
  SELECT a~matnr  AS product_id,
         t~maktx  AS name,
         a~stprs  AS price,          " MARA'dan standart fiyat (demo alanı)
         a~waers  AS currency,
         a~ersda  AS created_on
    INTO CORRESPONDING FIELDS OF TABLE @lt_result
    FROM mara AS a
    LEFT OUTER JOIN makt AS t
      ON  t~matnr = a~matnr
      AND t~spras = @lv_langiso
   WHERE a~lvorm = space                  " silinmek üzere işaretlenmemiş
   ORDER BY a~matnr
   UP TO @lv_top ROWS
   OFFSET @lv_skip ROWS.

  " ── 4. MPC tarafından üretilen entity yapısına eşle ─────────────────
  "    Üretilen tip: zcl_zmy_product_srv_mpc=>ts_product
  DATA ls_entity TYPE zcl_zmy_product_srv_mpc=>ts_product.

  LOOP AT lt_result INTO DATA(ls_row).
    CLEAR ls_entity.
    ls_entity-product_id  = ls_row-product_id.
    ls_entity-name        = ls_row-name.
    ls_entity-price       = ls_row-price.
    ls_entity-currency    = ls_row-currency.
    ls_entity-created_on  = ls_row-created_on.
    APPEND ls_entity TO ct_data.
  ENDLOOP.

  " ── 5. Satır içi sayı (sayfalama öncesi toplam satır) ────────────────
  "    Yalnızca istemci $inlinecount=allpages ile istediyse zahmet edin
  IF io_tech_request_context->get_inlinecount( ) = abap_true.
    SELECT COUNT(*) INTO @DATA(lv_total)
      FROM mara
     WHERE lvorm = space.
    es_response_context-inlinecount = lv_total.
  ENDIF.

ENDMETHOD.
```

> 💡 **`OFFSET ... ROWS`**, ABAP 7.50+ Open SQL sözdizimidir. Daha eski sistemlerde SELECT sonrasında satırları manuel atlayan bir döngü kullanın veya `@lv_skip` değişkeniyle yardımcı değişken yaklaşımını deneyin. Sistem sürümünüzü her zaman kontrol edin (`SE38 → ABAP düzenleyici → menü System → Status`).

> ⚠️ **C#/Python tuzağı:** `ct_data`, genel bir tablo (`ANY TABLE`) olarak tiplenmiştir — çerçeve onu dinamik olarak entity type'a eşler. İki yoldan biriyle doldurabilirsiniz: `zcl_..._mpc=>ts_product`'a eşleyerek (en güvenlisi — derleyici alan adlarınızı kontrol eder) ya da `CORRESPONDING #( )` kullanarak. `ct_data`'yı doğrudan entity tablosuymuş gibi kullanmayın — önce tiplenmiş bir yerel tablo doldurun, ardından `ct_data = CORRESPONDING #( lt_typed )` yapın.

---

## 23.2 GET_ENTITY — anahtara göre tek kayıt döndürme

### 1️⃣ Benzetme

`GET_ENTITY`, `GET /api/products/{id}` ile aynıdır. Bir anahtar, bir kayıt. Bulunamazsa 404 fırlatın.

### 2️⃣ Bunu zaten biliyorsun

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetById(string id)
{
    var product = await _db.Products
        .Include(p => p.Description)
        .FirstOrDefaultAsync(p => p.ProductId == id);

    if (product is null)
        return NotFound($"Product '{id}' not found.");

    return Ok(MapToDto(product));
}
```

```python
@app.get("/ProductSet/{product_id}", response_model=Product)
def get_product(product_id: str, db: Session = Depends(get_db)):
    product = db.query(ProductModel).filter_by(product_id=product_id).first()
    if product is None:
        raise HTTPException(status_code=404, detail=f"Product {product_id} not found")
    return product
```

### 3️⃣ ABAP'taki karşılığı — tam GET_ENTITY implementasyonu

```abap
METHOD productset_get_entity.
  "--------------------------------------------------------------------
  " GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')
  " Anahtarına göre tek bir ürün döndürür.
  "--------------------------------------------------------------------

  " ── 1. IT_KEY_TAB'dan anahtarı oku ──────────────────────────────────
  "    IT_KEY_TAB, /IWBEP/T_MGW_TECH_PAIRS tipindedir
  DATA(lv_product_id) = VALUE matnr(
      OPTIONAL
      ( it_key_tab[ name = 'ProductId' ]-value ) ).

  IF lv_product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId key is required'.
  ENDIF.

  " ── 2. MARA + MAKT'tan SELECT et ─────────────────────────────────────
  DATA lv_langiso TYPE laiso.
  lv_langiso = cl_abap_syst=>get_language( ).

  DATA ls_mara TYPE mara.
  DATA ls_makt TYPE makt.

  SELECT SINGLE * FROM mara INTO @ls_mara WHERE matnr = @lv_product_id.
  IF sy-subrc <> 0.
    " Standart OData "bulunamadı" exception'ı — çerçeve HTTP 404 döndürür
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-not_found
        message          = |Product '{ lv_product_id }' does not exist|.
  ENDIF.

  SELECT SINGLE maktx FROM makt
    INTO @DATA(lv_name)
   WHERE matnr = @lv_product_id
     AND spras = @lv_langiso.

  " ── 3. Çıktı entity'sini doldur ──────────────────────────────────────
  er_entity-product_id  = ls_mara-matnr.
  er_entity-name        = lv_name.
  er_entity-price       = ls_mara-stprs.
  er_entity-currency    = ls_mara-waers.
  er_entity-created_on  = ls_mara-ersda.

ENDMETHOD.
```

> 🧭 **İş hayatında:** Gerçek projelerde `er_entity`, `TYPE REF TO data` (genel bir veri referansı) olarak tanımlanmıştır. Çerçeve onu doğru MPC yapısına dereferans eder. Yeniden tanımlanan metot gövdesinde `zcl_..._mpc=>ts_product` olarak tiplenmiş göreceksiniz — normal bir yapı gibi doldurun.

---

## 23.3 Anahtarı okuma — it_key_tab ve get_keys()

Anahtar değerini çıkarmanın iki eşdeğer yolu vardır. İkisini de bilin — işte iki farklı yaklaşım.

### A Kalıbı: Doğrudan tablo okuma (en basit, en yaygın)

```abap
" Doğrudan import parametre tablosundan okur
DATA(lv_id) = it_key_tab[ name = 'ProductId' ]-value.

" Güvenli variant OPTIONAL ile (anahtar bulunamazsa exception yok)
DATA(lv_id_safe) = VALUE #( OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).
```

### B Kalıbı: io_tech_request_context üzerinde get_keys() kullanma

```abap
" OO yolu — anahtar hakkında daha zengin metadata istiyorsanız kullanışlı
io_tech_request_context->get_keys(
  IMPORTING
    et_key_tab = DATA(lt_keys) ).

DATA(ls_key) = VALUE /iwbep/s_mgw_tech_pair(
    OPTIONAL
    ( lt_keys[ name = 'ProductId' ] ) ).

DATA(lv_product_id) = ls_key-value.
```

### Bileşik anahtarlar (birden fazla anahtar alanı)

İki anahtarı olan `SalesOrderItem` için (`SalesOrderId` ve `ItemNumber`):

```abap
DATA(lv_order_id) = it_key_tab[ name = 'SalesOrderId' ]-value.
DATA(lv_item_no)  = it_key_tab[ name = 'ItemNumber'   ]-value.
```

> ⚠️ **C#/Python tuzağı:** `IT_KEY_TAB`'daki `name` alanı, ABAP alan adı (`product_id`) değil, **OData özellik adıdır** (SEGW'de tanımlandığı şekliyle camelCase — ör. `'ProductId'`). Bunlar çoğunlukla büyük-küçük harf açısından farklıdır. Boş bir değer alırsanız SEGW modelindeki tam özellik adını kontrol edin.

---

## 23.4 Uçtan uca tam salt okunur servis

Her şeyi bir araya getirelim. `Product` için tam çalışan bir okuma servisi için ihtiyacınız olan her şey aşağıdadır.

### SEGW'de yaptıklarınız (kısım 21'den özet)

1. `ZMY_PRODUCT_SRV` projesi oluşturuldu
2. `Product` Entity Type'ı `ProductId` (anahtar), `Name`, `Price`, `Currency`, `CreatedOn` özellikleriyle tanımlandı
3. Oluşturma çalıştırıldı — `ZMY_PRODUCT_SRV_DPC_EXT`, `ZMY_PRODUCT_SRV_MPC_EXT` sınıfları elde edildi
4. `/IWFND/MAINT_SERVICE`'e kaydedildi

### `ZMY_PRODUCT_SRV_DPC_EXT` içindeki iki metot (tam, temiz versiyon)

```abap
CLASS zcl_zmy_product_srv_dpc_ext DEFINITION
  INHERITING FROM zcl_zmy_product_srv_dpc
  PUBLIC FINAL CREATE PUBLIC.

  PUBLIC SECTION.
    METHODS productset_get_entityset REDEFINITION.
    METHODS productset_get_entity    REDEFINITION.

ENDCLASS.

CLASS zcl_zmy_product_srv_dpc_ext IMPLEMENTATION.

"======================================================================
" GET /ProductSet  — koleksiyonu döndür
"======================================================================
  METHOD productset_get_entityset.

    CONSTANTS c_max_rows TYPE i VALUE 500.    " güvenlik sınırı

    " MAKT için giriş dilini belirle
    DATA(lv_langiso) = cl_abap_syst=>get_language( ).

    " Sayfalama
    DATA(lv_top)  = COND i( WHEN is_paging-top  > 0
                             THEN CONV i( is_paging-top )
                             ELSE c_max_rows ).
    DATA(lv_skip) = CONV i( is_paging-skip ).

    " Üretilen MPC yapısıyla eşleşen sonuç tipi
    DATA lt_products TYPE TABLE OF zcl_zmy_product_srv_mpc=>ts_product.

    " Ana SELECT — MARA başlık + MAKT açıklaması
    SELECT a~matnr  AS product_id,
           t~maktx  AS name,
           a~stprs  AS price,
           a~waers  AS currency,
           a~ersda  AS created_on
      INTO CORRESPONDING FIELDS OF TABLE @lt_products
      FROM mara AS a
      LEFT OUTER JOIN makt AS t
        ON  t~matnr = a~matnr
        AND t~spras = @lv_langiso
     WHERE a~lvorm = space
     ORDER BY a~matnr
     UP TO @lv_top ROWS
     OFFSET @lv_skip ROWS.

    ct_data = CORRESPONDING #( lt_products ).

    " Satır içi sayı (sayfalama öncesi toplam)
    IF io_tech_request_context->get_inlinecount( ) = abap_true.
      SELECT COUNT(*) INTO @DATA(lv_total)
        FROM mara WHERE lvorm = space.
      es_response_context-inlinecount = lv_total.
    ENDIF.

  ENDMETHOD.

"======================================================================
" GET /ProductSet('P-100')  — anahtara göre tek entity döndür
"======================================================================
  METHOD productset_get_entity.

    " Anahtarı çıkar
    DATA(lv_product_id) = VALUE matnr(
        OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).

    IF lv_product_id IS INITIAL.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid           = /iwbep/cx_mgw_busi_exception=>business_error
          http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
          message          = 'ProductId is required'.
    ENDIF.

    " Dil
    DATA(lv_langiso) = cl_abap_syst=>get_language( ).

    " Başlığı oku
    SELECT SINGLE matnr, stprs, waers, ersda
      INTO @DATA(ls_mara)
      FROM mara
     WHERE matnr = @lv_product_id.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid           = /iwbep/cx_mgw_busi_exception=>business_error
          http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-not_found
          message          = |Product '{ lv_product_id }' not found|.
    ENDIF.

    " Açıklamayı oku
    SELECT SINGLE maktx INTO @DATA(lv_name)
      FROM makt
     WHERE matnr = @lv_product_id
       AND spras = @lv_langiso.

    " Çıktıyı doldur
    er_entity-product_id  = ls_mara-matnr.
    er_entity-name        = lv_name.
    er_entity-price       = ls_mara-stprs.
    er_entity-currency    = ls_mara-waers.
    er_entity-created_on  = ls_mara-ersda.

  ENDMETHOD.

ENDCLASS.
```

### /IWFND/GW_CLIENT'ta test etme

`/IWFND/GW_CLIENT` transaction'ı sizin Postman'ınızdır. Nasıl kullanılır:

1. `/IWFND/GW_CLIENT`'ı çalıştırın
2. **Request URI** alanına göreli yolu girin:
   - Koleksiyon: `/sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet?$format=json`
   - Tek entity: `/sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')?$format=json`
3. **Execute**'e (F8) tıklayın
4. Alt bölümdeki yanıtı izleyin

### Görmesi beklenenler — koleksiyon yanıtı

```http
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet?$format=json&$top=2
```

```json
{
  "d": {
    "results": [
      {
        "__metadata": {
          "id": "https://mydev/.../ProductSet('P-100')",
          "uri": "https://mydev/.../ProductSet('P-100')",
          "type": "ZMY_PRODUCT_SRV.Product"
        },
        "ProductId": "P-100",
        "Name": "Precision Lathe T500",
        "Price": "4250.00",
        "Currency": "EUR",
        "CreatedOn": "/Date(1748131200000)/"
      },
      {
        "__metadata": { ... },
        "ProductId": "P-101",
        "Name": "CNC Mill X200",
        "Price": "18900.00",
        "Currency": "EUR",
        "CreatedOn": "/Date(1748217600000)/"
      }
    ]
  }
}
```

### Görmesi beklenenler — tek entity yanıtı

```http
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')?$format=json
```

```json
{
  "d": {
    "__metadata": {
      "id": "https://mydev/.../ProductSet('P-100')",
      "uri": "https://mydev/.../ProductSet('P-100')",
      "type": "ZMY_PRODUCT_SRV.Product"
    },
    "ProductId": "P-100",
    "Name": "Precision Lathe T500",
    "Price": "4250.00",
    "Currency": "EUR",
    "CreatedOn": "/Date(1748131200000)/"
  }
}
```

### 404 nasıl görünür

```http
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('DOESNOTEXIST')?$format=json
HTTP/1.1 404 Not Found
```

```json
{
  "error": {
    "code": "SY/530",
    "message": {
      "lang": "en",
      "value": "Product 'DOESNOTEXIST' not found"
    },
    "innererror": { ... }
  }
}
```

> 🧭 **İş hayatında:** Fiori uygulaması genel bir "Bir hata oluştu" mesajı gösterdiğinde `/IWFND/ERROR_LOG`'u (ya da `/IWFND/APPS_LOG`'u) açın. Bunlar sunucu tarafındaki OData hata günlükleridir — HTTP yanıtı ayıklanmış olsa bile tam exception yığın izini gösterirler. Her OData hata ayıklama oturumunda bakacağınız ilk yerdir.

---

## 🧠 Özet

- **`GET_ENTITYSET`** = koleksiyon endpoint'i. Tiplenmiş bir yerel tabloya SELECT edin, `ct_data`'ya kopyalayın, isteğe bağlı olarak `es_response_context-inlinecount` ayarlayın.
- **`GET_ENTITY`** = tek kayıt endpoint'i. Anahtarı `it_key_tab`'dan okuyun, tek SELECT yapın, `er_entity`'yi doldurun, bulunamazsa 404 exception'ı fırlatın.
- **Anahtar çıkarma:** `it_key_tab[ name = 'PropertyName' ]-value` (doğrudan tablo ifadesi) — en yaygın kalıp. `name`, ABAP alan adı değil OData özellik adını (SEGW'deki gibi) kullanır.
- **Bulunamadı işleme:** `http_status_code = gcs_http_status_codes-not_found` ile `/IWBEP/CX_MGW_BUSI_EXCEPTION` fırlatın. Çerçeve bunu düzgün bir OData hata gövdesiyle HTTP 404'e dönüştürür.
- **Test aracı:** `/IWFND/GW_CLIENT`, SAP sistemi içindeki Postman'dır. Her test turunda kullanın.
- **Hata ayıklama aracı:** `/IWFND/ERROR_LOG`, OData çağrıları için tam sunucu tarafı exception izlerini gösterir.

---

*[← İçindekiler](../content.md) | [← Önceki: Servis Metotları & Import Parametreleri](22-odata-methods-and-parameters.md) | [Sonraki: Arama & Query Seçenekleri →](24-odata-search-and-query-options.md)*
