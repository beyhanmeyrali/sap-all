# Kısım 25: Create, Update & Delete

*POST, PUT/PATCH ve DELETE — istek gövdesini okuma, veritabanına yazma, hataları yönetme ve transaction'larınızı temiz tutma.*

---

## ☕ Okuma tarafı ısınmaydı. Şimdi iş ciddileşiyor.

Kısımlar 23 ve 24 veri okumayı — güvenli, yan etkisiz yolu — ele alıyordu. Bu kısımda veri yazıyorsunuz: kayıt oluşturma, güncelleme, silme, girdileri doğrulama ve bir şeyler ters giderse geri alma.

Her şey `ZMY_PRODUCT_SRV_DPC_EXT`'te, üç yeni metodda yaşar: `PRODUCTSET_CREATE_ENTITY`, `PRODUCTSET_UPDATE_ENTITY` ve `PRODUCTSET_DELETE_ENTITY`.

`Product` entity'si için üçünü de oluşturacağız; kısımın sonunda doğrulama ve uygun hata yönetimi ekleyeceğiz; böylece servisiniz gerçek bir API gibi davranacak.

---

## 25.1 CREATE_ENTITY — POST'u yönetme

### 1️⃣ Benzetme

JSON gövdeli `POST /ProductSet`, Web API'deki `[HttpPost]` ile tamamen aynıdır. Çerçeve gövdeyi entity şeklinize deserialize eder. Siz doğrularsınız, veritabanına yazarsınız ve oluşturulan kaydı (genellikle sunucu tarafından atanan anahtar doldurulmuş şekilde) döndürürsünüz.

### 2️⃣ Bunu zaten biliyorsun

```csharp
[HttpPost]
public async Task<ActionResult<ProductDto>> Create([FromBody] ProductDto dto)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var product = new Product
    {
        ProductId = dto.ProductId,
        Name      = dto.Name,
        Price     = dto.Price,
        Currency  = dto.Currency,
        CreatedOn = DateTime.UtcNow
    };

    _db.Products.Add(product);
    await _db.SaveChangesAsync();

    return CreatedAtAction(nameof(GetById),
                           new { id = product.ProductId },
                           MapToDto(product));
}
```

```python
@app.post("/ProductSet", status_code=201, response_model=Product)
def create_product(payload: Product, db: Session = Depends(get_db)):
    if db.query(ProductModel).filter_by(product_id=payload.product_id).first():
        raise HTTPException(400, "Product already exists")
    db_product = ProductModel(**payload.dict())
    db.add(db_product)
    db.commit()
    db.refresh(db_product)
    return db_product
```

### 3️⃣ ABAP'taki karşılığı — CREATE_ENTITY implementasyonu

İstemci JSON gövdesiyle bir POST gönderir:

```http
POST /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet
Content-Type: application/json

{
  "ProductId": "Z-NEW-001",
  "Name": "Hydraulic Press H900",
  "Price": "22500.00",
  "Currency": "EUR"
}
```

`DPC_EXT`'te gövdeyi `io_data_provider->read_entry_data( )` ile okursunuz:

```abap
METHOD productset_create_entity.
  "======================================================================
  " POST /ProductSet — Yeni ürün oluştur
  "======================================================================

  " ── 1. İstek gövdesini entity yapısına oku ───────────────────────────
  " io_data_provider, /IWBEP/IF_MGW_ENTRY_DP tipindedir
  " JSON/XML gövdesini sizin için deserialize eder — JSON ayrıştırıcı yazmazsınız.

  DATA ls_product TYPE zcl_zmy_product_srv_mpc=>ts_product.

  io_data_provider->read_entry_data(
    IMPORTING
      es_data = ls_product ).         " ← gövde yapınıza eşlendi

  " ── 2. Girdiyi doğrula ───────────────────────────────────────────────
  " (doğrulama yardımcısı için bölüm 25.4'e bakın — burada satır içi çağırıyoruz)

  IF ls_product-product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId is required'.
  ENDIF.

  IF ls_product-price <= 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'Price must be greater than zero'.
  ENDIF.

  IF ls_product-currency IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'Currency is required'.
  ENDIF.

  " ── 3. Yinelenen kayıt kontrolü ──────────────────────────────────────
  SELECT SINGLE matnr FROM mara
    INTO @DATA(lv_existing)
   WHERE matnr = @ls_product-product_id.

  IF sy-subrc = 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = |Product '{ ls_product-product_id }' already exists|.
  ENDIF.

  " ── 4. Veritabanı kaydını hazırla ─────────────────────────────────────
  DATA ls_mara TYPE mara.
  ls_mara-matnr = ls_product-product_id.
  ls_mara-stprs = ls_product-price.
  ls_mara-waers = ls_product-currency.
  ls_mara-ersda = sy-datum.           " sunucu tarafı oluşturma tarihi

  " Gerçek bir projede, doğrudan INSERT yerine bir BAPI veya
  " function module çağırırsınız — aşağıdaki "İş hayatında" notuna bakın.
  INSERT mara FROM @ls_mara.

  IF sy-subrc <> 0.
    ROLLBACK WORK.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-server_error
        message          = |Failed to insert product '{ ls_product-product_id }'|.
  ENDIF.

  " ── 5. MAKT'a açıklama ekle ──────────────────────────────────────────
  IF ls_product-name IS NOT INITIAL.
    DATA ls_makt TYPE makt.
    ls_makt-matnr = ls_product-product_id.
    ls_makt-spras = cl_abap_syst=>get_language( ).
    ls_makt-maktx = ls_product-name.
    INSERT makt FROM @ls_makt.
  ENDIF.

  " ── 6. Commit ─────────────────────────────────────────────────────────
  COMMIT WORK AND WAIT.

  " ── 7. Oluşturulan entity'yi döndür (doğruluk için DB'den yeniden oku) ─
  er_entity-product_id  = ls_mara-matnr.
  er_entity-name        = ls_product-name.
  er_entity-price       = ls_mara-stprs.
  er_entity-currency    = ls_mara-waers.
  er_entity-created_on  = ls_mara-ersda.

ENDMETHOD.
```

> 🧭 **İş hayatında:** Gerçek bir SAP projesinde `MARA`'ya neredeyse hiç doğrudan INSERT yapmazsınız. Malzemeler `BAPI_MATERIAL_SAVEDATA` veya `MM_MATERIAL_MAINTAIN_OSP` aracılığıyla oluşturulur. Doğrudan INSERT'ler SAP uygulama katmanını (numara aralıkları, user exit'ler, deftere nakil mantığı) atlar. Yukarıdaki doğrudan INSERT yalnızca öğretim netliği için gösterilmiştir — gerçek projenizde bir BAPI veya business object metodu çağıracaksınız ve sonucu geri döndüreceksiniz. `io_data_provider->read_entry_data( )` + doğrulama + BAPI çağrısı + `COMMIT WORK` kalıbı ne olursa olsun aynı kalır.

> ⚠️ **C#/Python tuzağı:** ABAP veritabanı transaction'ları her ifadeden sonra **otomatik olarak commit edilmez**. `COMMIT WORK` (ya da devam etmeden önce commit'in tam olarak kalıcı hale geldiğinden emin olmak istiyorsanız `COMMIT WORK AND WAIT`) çağırmanız gerekir. `COMMIT WORK`'ü unutmak yeni başlayanlar için çok yaygın bir hatadır — INSERT başarılı olur, metot HTTP 201 döndürür ama diyalog adımı sona erdiğinde LUW (Logical Unit of Work) geri alındığı için veritabanında hiçbir şey olmaz.

---

## 25.2 UPDATE_ENTITY — PUT ve PATCH/MERGE'i yönetme

### 1️⃣ Benzetme

`PUT /ProductSet('P-100')` = kaydın tamamını değiştir.
`PATCH /ProductSet('P-100')` = yalnızca gönderilen alanları güncelle.

OData v2'de standart PATCH fiili aslında `MERGE` olarak adlandırılır (PATCH standartlaşmadan önce SAP'ın tanımladığı bir HTTP uzantısı). Modern istemciler `PATCH` gönderir; SAP her ikisini de `UPDATE_ENTITY`'ye yönlendirir. Gerektiğinde metot içinde ayırt edebilirsiniz.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// PUT — tam değiştirme
[HttpPut("{id}")]
public IActionResult Put(string id, [FromBody] ProductDto dto)
{
    var product = _db.Products.Find(id);
    if (product is null) return NotFound();

    product.Name     = dto.Name;
    product.Price    = dto.Price;
    product.Currency = dto.Currency;
    _db.SaveChanges();
    return NoContent();
}

// PATCH — kısmi güncelleme
[HttpPatch("{id}")]
public IActionResult Patch(string id, [FromBody] JsonPatchDocument<ProductDto> patch)
{
    var product = _db.Products.Find(id);
    if (product is null) return NotFound();
    patch.ApplyTo(product, ModelState);
    _db.SaveChanges();
    return NoContent();
}
```

### 3️⃣ ABAP'taki karşılığı — UPDATE_ENTITY implementasyonu

```abap
METHOD productset_update_entity.
  "======================================================================
  " PUT  /ProductSet('P-100')       — tam değiştirme
  " PATCH/MERGE /ProductSet('P-100') — kısmi güncelleme (aynı metot)
  "======================================================================

  " ── 1. Anahtarı oku ────────────────────────────────────────────────────
  DATA(lv_product_id) = VALUE matnr(
      OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).

  IF lv_product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId key is required'.
  ENDIF.

  " ── 2. Kaydın var olduğunu kontrol et ────────────────────────────────
  SELECT SINGLE matnr, stprs, waers, ersda
    INTO @DATA(ls_mara_db)
    FROM mara
   WHERE matnr = @lv_product_id.

  IF sy-subrc <> 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-not_found
        message          = |Product '{ lv_product_id }' not found|.
  ENDIF.

  " ── 3. İstek gövdesini oku ───────────────────────────────────────────
  DATA ls_product TYPE zcl_zmy_product_srv_mpc=>ts_product.
  io_data_provider->read_entry_data( IMPORTING es_data = ls_product ).

  " ── 4. PUT vs PATCH/MERGE'i ayırt et ─────────────────────────────────
  DATA(lv_http_method) = io_tech_request_context->get_http_method( ).

  " PUT (tam değiştirme) için: gövdedeki her alan DB değerinin yerini alır.
  " PATCH/MERGE (kısmi) için: yalnızca gönderilen/initial olmayan alanlar güncellenir.

  DATA ls_mara_upd TYPE mara.
  ls_mara_upd-matnr = lv_product_id.

  CASE lv_http_method.
    WHEN 'PUT'.
      " Tam değiştirme — zorunlu alanları doğrula
      IF ls_product-price <= 0.
        RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
          EXPORTING
            textid           = /iwbep/cx_mgw_busi_exception=>business_error
            http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
            message          = 'Price must be greater than zero'.
      ENDIF.
      ls_mara_upd-stprs = ls_product-price.
      ls_mara_upd-waers = ls_product-currency.

    WHEN 'PATCH' OR 'MERGE'.
      " Kısmi güncelleme — yalnızca gönderilen alanları uygula (initial olmayan)
      ls_mara_upd-stprs = COND #( WHEN ls_product-price    > 0
                                   THEN ls_product-price
                                   ELSE ls_mara_db-stprs ).
      ls_mara_upd-waers = COND #( WHEN ls_product-currency IS NOT INITIAL
                                   THEN ls_product-currency
                                   ELSE ls_mara_db-waers ).

    WHEN OTHERS.
      ls_mara_upd-stprs = ls_product-price.
      ls_mara_upd-waers = ls_product-currency.
  ENDCASE.

  " ── 5. MARA kaydını güncelle ──────────────────────────────────────────
  UPDATE mara SET stprs = @ls_mara_upd-stprs
                  waers = @ls_mara_upd-waers
   WHERE matnr = @lv_product_id.

  IF sy-subrc <> 0.
    ROLLBACK WORK.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-server_error
        message          = |Failed to update product '{ lv_product_id }'|.
  ENDIF.

  " ── 6. MAKT'ta açıklamayı güncelle (gönderildiyse) ────────────────────
  IF ls_product-name IS NOT INITIAL.
    DATA(lv_langiso) = cl_abap_syst=>get_language( ).

    UPDATE makt SET maktx = @ls_product-name
     WHERE matnr = @lv_product_id
       AND spras = @lv_langiso.

    IF sy-subrc <> 0.
      " Satır mevcut olmayabilir — bunun yerine INSERT et
      INSERT INTO makt VALUES @(
          VALUE makt(
              matnr = lv_product_id
              spras = lv_langiso
              maktx = ls_product-name ) ).
    ENDIF.
  ENDIF.

  " ── 7. Commit ─────────────────────────────────────────────────────────
  COMMIT WORK AND WAIT.

  " ── 8. Güncellenen entity'yi döndür ───────────────────────────────────
  er_entity-product_id  = lv_product_id.
  er_entity-name        = ls_product-name.
  er_entity-price       = ls_mara_upd-stprs.
  er_entity-currency    = ls_mara_upd-waers.
  er_entity-created_on  = ls_mara_db-ersda.

ENDMETHOD.
```

> 💡 **PATCH semantiği notu:** OData v2'nin MERGE/PATCH'i aslında gövdede *hangi* alanların dahil edildiğini, hangilerinin atlandığını size söylemez — yalnızca gönderilen alanları iletir ama ABAP'ta bunlar atlanmış alanlar için varsayılan/initial değerlerle yapınıza gelir. Yukarıda gösterilen kalıp (`COND #( WHEN ls_product-price > 0 THEN ... ELSE DB değerini koru )`) çoğu ekibin kullandığı pratik geçici çözümdür. Daha sağlam kısmi güncelleme desteği için ham HTTP gövdesini string olarak okuyun ve kendiniz ayrıştırın — ama bu uç bir durumdur.

---

## 25.3 DELETE_ENTITY — DELETE'i yönetme

### 1️⃣ Benzetme

`DELETE /ProductSet('P-100')` — anahtarı oku, kaydı sil. Başarılı olursa HTTP 204 No Content döndür. Kayıt mevcut değilse 404 döndür.

### 2️⃣ Bunu zaten biliyorsun

```csharp
[HttpDelete("{id}")]
public IActionResult Delete(string id)
{
    var product = _db.Products.Find(id);
    if (product is null) return NotFound();
    _db.Products.Remove(product);
    _db.SaveChanges();
    return NoContent();   // HTTP 204
}
```

### 3️⃣ ABAP'taki karşılığı — DELETE_ENTITY implementasyonu

```abap
METHOD productset_delete_entity.
  "======================================================================
  " DELETE /ProductSet('P-100')
  "======================================================================

  " ── 1. Anahtarı oku ────────────────────────────────────────────────────
  DATA(lv_product_id) = VALUE matnr(
      OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).

  IF lv_product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId is required'.
  ENDIF.

  " ── 2. Kaydın var olduğunu kontrol et ─────────────────────────────────
  SELECT SINGLE matnr FROM mara
    INTO @DATA(lv_check)
   WHERE matnr = @lv_product_id.

  IF sy-subrc <> 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-not_found
        message          = |Product '{ lv_product_id }' not found|.
  ENDIF.

  " ── 3. İş kuralı — ör. satış siparişlerinde referans alınıyorsa silinemez
  SELECT SINGLE vbeln FROM vbap
    INTO @DATA(lv_so_ref)
   WHERE matnr = @lv_product_id.

  IF sy-subrc = 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = |Product '{ lv_product_id }' cannot be deleted: referenced in sales order { lv_so_ref }|.
  ENDIF.

  " ── 4. MAKT ve MARA'dan sil ───────────────────────────────────────────
  DELETE FROM makt WHERE matnr = @lv_product_id.
  DELETE FROM mara WHERE matnr = @lv_product_id.

  IF sy-subrc <> 0.
    ROLLBACK WORK.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-server_error
        message          = |Failed to delete product '{ lv_product_id }'|.
  ENDIF.

  " ── 5. Commit ─────────────────────────────────────────────────────────
  COMMIT WORK AND WAIT.

  " DELETE_ENTITY'nin dolduracağı ER_ENTITY yoktur — çerçeve HTTP 204 döndürür.

ENDMETHOD.
```

> 🧭 **İş hayatında:** SAP'ta gerçek silme işlemi neredeyse hiç yapmazsınız. Çoğu entity "mantıksal olarak silinir" — bir silme bayrağı ayarlanır (`MARA-LVORM = 'X'`) ve aşağı akış süreçleri (arşivleme, SARA) fiziksel silmeyi daha sonra yönetir. Kalıbı buna göre uyarlayın: `DELETE FROM mara` yerine `UPDATE mara SET lvorm = 'X'` yapın.

---

## 25.4 Doğrulama, /IWBEP/CX_MGW_BUSI_EXCEPTION fırlatma ve commit/rollback disiplini

### Exception sınıfı

`/IWBEP/CX_MGW_BUSI_EXCEPTION`, beklenen iş hataları için bir DPC_EXT metodundan kaçmasına izin vermeniz gereken *tek* exception'dır. Çerçeve bunu yakalar ve iyi biçimlendirilmiş bir OData hata yanıtına dönüştürür.

```abap
" HTTP 400 — istemci hatalı girdi gönderdi
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
    message          = 'Açıklayıcı mesaj buraya'.

" HTTP 404 — kayıt bulunamadı
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-not_found
    message          = 'Product not found'.

" HTTP 409 — çakışma (yinelenen, kilitli kayıt)
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-conflict
    message          = 'Record already exists'.

" HTTP 500 — beklenmedik sunucu hatası (dikkatli kullanın)
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-server_error
    message          = 'Unexpected error during processing'.
```

### Mesaj container'ı aracılığıyla birden fazla mesaj

3 geçersiz alanı olan bir form gibi, aynı anda birden fazla doğrulama hatasını raporlamak istediğinizde, hemen fırlatmak yerine mesaj container'ını kullanın:

```abap
METHOD productset_create_entity.

  DATA lt_messages TYPE /iwbep/t_mgw_business_msg.
  DATA ls_msg      TYPE /iwbep/s_mgw_business_msg.
  DATA lv_errors   TYPE abap_bool.

  " Önce tüm hataları topla
  IF ls_product-product_id IS INITIAL.
    ls_msg-msgty = 'E'.
    ls_msg-msgid = 'ZMY_MSGS'.
    ls_msg-msgno = '001'.
    ls_msg-msgv1 = 'ProductId is required'.
    APPEND ls_msg TO lt_messages.
    lv_errors = abap_true.
  ENDIF.

  IF ls_product-price <= 0.
    CLEAR ls_msg.
    ls_msg-msgty = 'E'.
    ls_msg-msgid = 'ZMY_MSGS'.
    ls_msg-msgno = '002'.
    ls_msg-msgv1 = 'Price must be greater than zero'.
    APPEND ls_msg TO lt_messages.
    lv_errors = abap_true.
  ENDIF.

  " Tüm mesajlarla birlikte fırlat
  IF lv_errors = abap_true.
    mo_context->get_message_container(
      )->add_messages_from_bapi(
          it_bapi_messages = CORRESPONDING #( lt_messages ) ).
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        message_container = mo_context->get_message_container( ).
  ENDIF.

  " ... INSERT ile devam et ...

ENDMETHOD.
```

### COMMIT / ROLLBACK disiplini

Kurallar basit ve kesindir:

```
Kural 1: COMMIT WORK'ü yalnızca bir yazma metodunda çağırın (CREATE/UPDATE/DELETE).
         GET_ENTITY veya GET_ENTITYSET'te ASLA çağırmayın.

Kural 2: Tüm veritabanı yazmaları başarıyla tamamlandıktan SONRA COMMIT WORK çağırın.
         Herhangi bir yazma başarısız olursa → ROLLBACK WORK → exception fırlat.

Kural 3: ER_ENTITY içinde entity'yi döndürmeden önce verinin kalıcı hale
         geldiğinden emin olmak için COMMIT WORK AND WAIT kullanın
         (sadece COMMIT WORK değil) — bu commit'i senkron yapar.

Kural 4: Bir BAPI çağırıyorsanız, commit öncesinde RETURN tablosunu hatalara karşı kontrol edin:
         - Herhangi bir MSGTY = 'E' veya 'A' varsa → ROLLBACK WORK → exception fırlat.
         - Tüm MSGTY = 'S' veya 'W' ise → COMMIT WORK AND WAIT.
```

```abap
" BAPI kalıbı (gerçek projelerde yapacağınız):
CALL FUNCTION 'BAPI_MATERIAL_SAVEDATA'
  EXPORTING
    headdata        = ls_headdata
  TABLES
    returnmessages  = lt_return.

" Hataları kontrol et
IF line_exists( lt_return[ type = 'E' ] ) OR
   line_exists( lt_return[ type = 'A' ] ).
  ROLLBACK WORK.
  DATA(ls_err) = lt_return[ type = 'E' ].
  RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
    EXPORTING
      textid           = /iwbep/cx_mgw_busi_exception=>business_error
      http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
      message          = |{ ls_err-message }|.
ENDIF.

CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
  EXPORTING
    wait = abap_true.             " COMMIT WORK AND WAIT ile eşdeğer
```

> ⚠️ **C#/Python tuzağı:** Klasik SEGW DPC_EXT'te otomatik unit-of-work / transaction kapsamı yoktur. Yaptığınız her veritabanı yazması SAP LUW'sunda (Logical Unit of Work — tek bir örtük transaction) birikir. `COMMIT WORK` hepsini sonuçlandırır. `ROLLBACK WORK` son commit'ten bu yana her şeyi geri alır. Metodunuz birden fazla INSERT yapıyorsa ve üçüncüsü başarısız olursa, exception'ı fırlatmadan önce ilk ikisini geri almak için `ROLLBACK WORK` çağırın. Rollback'i unutmak veritabanında kısmi veri bırakır.

> ⚠️ **C#/Python tuzağı:** Bir okuma metodunda (`GET_ENTITY`, `GET_ENTITYSET`) asla `COMMIT WORK` çağırmayın. Bazı geliştiriciler yanlışlıkla önceki bir diyalog adımından açık bir LUW commit eder. SEGW/OData çağrılarında bu genellikle sorun çıkarmaz (her HTTP çağrısı taze bir diyalog adımıdır) ama iyi bir alışkanlıktır.

### Yeniden kullanılabilir doğrulama yardımcısı

CREATE ve UPDATE'in paylaşabilmesi için doğrulamayı özel bir metoda çıkarın:

```abap
" ZMY_PRODUCT_SRV_DPC_EXT içinde — özel metot ZM_VALIDATE_PRODUCT

METHOD zm_validate_product.
  "--------------------------------------------------------------
  " IS_PRODUCT'ı doğrular. Doğrulama başarısız olursa
  " /IWBEP/CX_MGW_BUSI_EXCEPTION fırlatır.
  " IS_FOR_CREATE = abap_true  → ProductId initial olmamalı
  " IS_FOR_CREATE = abap_false → ProductId initial olabilir (UPDATE)
  "--------------------------------------------------------------
  IMPORTING
    is_product    TYPE zcl_zmy_product_srv_mpc=>ts_product
    is_for_create TYPE abap_bool DEFAULT abap_true.

  IF is_for_create = abap_true AND is_product-product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId is required for creation'.
  ENDIF.

  IF is_product-price < 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'Price cannot be negative'.
  ENDIF.

  IF is_product-currency IS NOT INITIAL.
    " Para birimini T006A'ya (SAP para birimi tablosu) karşı doğrula
    SELECT SINGLE waers FROM tcurc
      INTO @DATA(lv_cur)
     WHERE waers = @is_product-currency.
    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid           = /iwbep/cx_mgw_busi_exception=>business_error
          http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
          message          = |Currency '{ is_product-currency }' is not valid|.
    ENDIF.
  ENDIF.

ENDMETHOD.
```

Her iki `CREATE_ENTITY` ve `UPDATE_ENTITY`'nin en üstünde çağırın:

```abap
zm_validate_product(
    is_product    = ls_product
    is_for_create = abap_true ).
```

---

## ☕ Yazma tarafının tam resmi

```mermaid
sequenceDiagram
    participant Client as Fiori / HTTP istemcisi
    participant FW as OData Framework
    participant DPC as DPC_EXT metodu
    participant DB as SAP Veritabanı

    Client->>FW: POST /ProductSet (JSON gövde)
    FW->>DPC: CREATE_ENTITY(io_data_provider, ...)
    DPC->>DPC: read_entry_data() → ls_product
    DPC->>DPC: doğrula
    alt doğrulama başarısız
        DPC-->>FW: CX_MGW_BUSI_EXCEPTION fırlat
        FW-->>Client: HTTP 400 + OData hata gövdesi
    end
    DPC->>DB: INSERT mara
    DPC->>DB: INSERT makt
    DPC->>DB: COMMIT WORK AND WAIT
    DPC-->>FW: er_entity'yi doldur
    FW-->>Client: HTTP 201 + oluşturulan entity JSON

    style DPC fill:#4CAF50,color:white
```

---

## 🧠 Özet

- **`CREATE_ENTITY`**: `io_data_provider->read_entry_data( IMPORTING es_data = ls_struct )` ile gövdeyi okuyun, doğrulayın, INSERT yapın, `COMMIT WORK AND WAIT` çağırın, `er_entity`'yi doldurun. Çerçeve HTTP 201 döndürür.
- **`UPDATE_ENTITY`**: `it_key_tab`'dan anahtarı okuyun, varlığı kontrol edin (yoksa 404), gövdeyi okuyun, `io_tech_request_context->get_http_method( )` ile PUT'u PATCH'ten ayırt edin, UPDATE yapın, commit edin, `er_entity`'yi doldurun. Çerçeve HTTP 200 döndürür.
- **`DELETE_ENTITY`**: anahtarı okuyun, varlığı kontrol edin (404), iş kurallarını uygulayın, DELETE yapın, commit edin. Dolduracağınız `er_entity` yoktur — çerçeve HTTP 204 döndürür.
- **Hata yönetimi**: her zaman uygun bir HTTP durum koduyla `/IWBEP/CX_MGW_BUSI_EXCEPTION` fırlatın. Düz `CX_ROOT` exception'larının kaçmasına asla izin vermeyin — bunlar ayrıntısız genel 500'lere neden olur.
- **COMMIT / ROLLBACK kuralı**: tüm yazmalar başarıyla tamamlandıktan sonra `COMMIT WORK AND WAIT`; herhangi bir yazma kısmen başarılı olduysa exception fırlatmadan önce `ROLLBACK WORK`.
- **Gerçek projeler BAPI kullanır**: INSERT/UPDATE/DELETE'i bir BAPI çağrısına sarın, RETURN tablosunu kontrol edin, ardından çıplak `COMMIT WORK` yerine `BAPI_TRANSACTION_COMMIT` / `BAPI_TRANSACTION_ROLLBACK` kullanın.

---

*[← İçindekiler](../content.md) | [← Önceki: Arama & Query Seçenekleri](24-odata-search-and-query-options.md) | [Sonraki: Association'lar →](26-odata-associations.md)*
