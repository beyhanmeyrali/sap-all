# Chapter 25: Create, Update & Delete

*POST, PUT/PATCH, and DELETE — reading the request body, writing to the database, handling errors, and keeping your transactions clean.*

---

## ☕ The read side was warm-up. Now it gets real.

Chapters 23 and 24 were about reading data — the safe, no-side-effects path. This chapter is where you write data: create records, update them, delete them, validate inputs, and roll things back if something goes wrong.

Everything here lives in `ZMY_PRODUCT_SRV_DPC_EXT`, in three new methods: `PRODUCTSET_CREATE_ENTITY`, `PRODUCTSET_UPDATE_ENTITY`, and `PRODUCTSET_DELETE_ENTITY`.

We'll build all three for the `Product` entity, and by the end of the chapter add validation and proper error handling so your service behaves like a real API.

---

## 25.1 CREATE_ENTITY — handling POST

### 1️⃣ The analogy

`POST /ProductSet` with a JSON body is exactly `[HttpPost]` in Web API. The framework deserialises the body into your entity shape. You validate it, write it to the database, and return the created record (usually with the server-assigned key filled in).

### 2️⃣ You already know this

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

### 3️⃣ The ABAP way — CREATE_ENTITY implementation

The client sends a POST with a JSON body:

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

In `DPC_EXT` you read the body via `io_data_provider->read_entry_data( )`:

```abap
METHOD productset_create_entity.
  "======================================================================
  " POST /ProductSet — Create a new product
  "======================================================================

  " ── 1. Read the request body into the entity structure ───────────────
  " io_data_provider is of type /IWBEP/IF_MGW_ENTRY_DP
  " It deserialises the JSON/XML body for you — you never write a JSON parser.

  DATA ls_product TYPE zcl_zmy_product_srv_mpc=>ts_product.

  io_data_provider->read_entry_data(
    IMPORTING
      es_data = ls_product ).         " ← body mapped into your structure

  " ── 2. Validate input ────────────────────────────────────────────────
  " (see section 25.4 for the validation helper — we call it inline here)

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

  " ── 3. Check for duplicates ───────────────────────────────────────────
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

  " ── 4. Prepare the database record ───────────────────────────────────
  DATA ls_mara TYPE mara.
  ls_mara-matnr = ls_product-product_id.
  ls_mara-stprs = ls_product-price.
  ls_mara-waers = ls_product-currency.
  ls_mara-ersda = sy-datum.           " server-side creation date

  " In a real project you'd call a BAPI or function module instead of
  " inserting directly — see the "On the job" note below.
  INSERT mara FROM @ls_mara.

  IF sy-subrc <> 0.
    ROLLBACK WORK.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-server_error
        message          = |Failed to insert product '{ ls_product-product_id }'|.
  ENDIF.

  " ── 5. Insert description into MAKT ──────────────────────────────────
  IF ls_product-name IS NOT INITIAL.
    DATA ls_makt TYPE makt.
    ls_makt-matnr = ls_product-product_id.
    ls_makt-spras = cl_abap_syst=>get_language( ).
    ls_makt-maktx = ls_product-name.
    INSERT makt FROM @ls_makt.
  ENDIF.

  " ── 6. Commit ─────────────────────────────────────────────────────────
  COMMIT WORK AND WAIT.

  " ── 7. Return the created entity (re-read from DB for accuracy) ───────
  er_entity-product_id  = ls_mara-matnr.
  er_entity-name        = ls_product-name.
  er_entity-price       = ls_mara-stprs.
  er_entity-currency    = ls_mara-waers.
  er_entity-created_on  = ls_mara-ersda.

ENDMETHOD.
```

> 🧭 **On the job:** In a real SAP project you almost never INSERT directly into `MARA`. Materials are created via `BAPI_MATERIAL_SAVEDATA` or `MM_MATERIAL_MAINTAIN_OSP`. Direct inserts bypass the SAP application layer (number ranges, user exits, posting logic). The direct INSERT above is shown for teaching clarity — your actual project will call a BAPI or a business object method here, and you hand the result back. The `io_data_provider->read_entry_data( )` + validation + BAPI call + `COMMIT WORK` pattern stays the same regardless.

> ⚠️ **C#/Python gotcha:** ABAP database transactions are **NOT** auto-committed after each statement. You must call `COMMIT WORK` (or `COMMIT WORK AND WAIT` if you want to be sure the commit is fully persisted before continuing). Forgetting `COMMIT WORK` is a very common bug for newcomers — the INSERT succeeds, the method returns HTTP 201, but nothing is in the database because the LUW (Logical Unit of Work) was rolled back when the dialog step ended.

---

## 25.2 UPDATE_ENTITY — handling PUT and PATCH/MERGE

### 1️⃣ The analogy

`PUT /ProductSet('P-100')` = replace the whole record.
`PATCH /ProductSet('P-100')` = update only supplied fields.

In OData v2, the standard PATCH verb is actually called `MERGE` (an HTTP extension SAP introduced before PATCH was standardised). Modern clients send `PATCH`; SAP forwards both to `UPDATE_ENTITY`. You can distinguish them inside the method if you need to.

### 2️⃣ You already know this

```csharp
// PUT — full replacement
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

// PATCH — partial update
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

### 3️⃣ The ABAP way — UPDATE_ENTITY implementation

```abap
METHOD productset_update_entity.
  "======================================================================
  " PUT  /ProductSet('P-100')  — full replacement
  " PATCH/MERGE /ProductSet('P-100') — partial update (same method)
  "======================================================================

  " ── 1. Read the key ───────────────────────────────────────────────────
  DATA(lv_product_id) = VALUE matnr(
      OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).

  IF lv_product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId key is required'.
  ENDIF.

  " ── 2. Check record exists ────────────────────────────────────────────
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

  " ── 3. Read the request body ─────────────────────────────────────────
  DATA ls_product TYPE zcl_zmy_product_srv_mpc=>ts_product.
  io_data_provider->read_entry_data( IMPORTING es_data = ls_product ).

  " ── 4. Distinguish PUT vs PATCH/MERGE ────────────────────────────────
  DATA(lv_http_method) = io_tech_request_context->get_http_method( ).

  " For PUT (full replacement): every field from body replaces DB value.
  " For PATCH/MERGE (partial):  only supplied/non-initial fields are updated.

  DATA ls_mara_upd TYPE mara.
  ls_mara_upd-matnr = lv_product_id.

  CASE lv_http_method.
    WHEN 'PUT'.
      " Full replacement — validate required fields
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
      " Partial update — apply only fields that were sent (non-initial)
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

  " ── 5. UPDATE the MARA record ────────────────────────────────────────
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

  " ── 6. Update description in MAKT (if supplied) ───────────────────────
  IF ls_product-name IS NOT INITIAL.
    DATA(lv_langiso) = cl_abap_syst=>get_language( ).

    UPDATE makt SET maktx = @ls_product-name
     WHERE matnr = @lv_product_id
       AND spras = @lv_langiso.

    IF sy-subrc <> 0.
      " Row might not exist — insert instead
      INSERT INTO makt VALUES @(
          VALUE makt(
              matnr = lv_product_id
              spras = lv_langiso
              maktx = ls_product-name ) ).
    ENDIF.
  ENDIF.

  " ── 7. Commit ─────────────────────────────────────────────────────────
  COMMIT WORK AND WAIT.

  " ── 8. Return updated entity ──────────────────────────────────────────
  er_entity-product_id  = lv_product_id.
  er_entity-name        = ls_product-name.
  er_entity-price       = ls_mara_upd-stprs.
  er_entity-currency    = ls_mara_upd-waers.
  er_entity-created_on  = ls_mara_db-ersda.

ENDMETHOD.
```

> 💡 **PATCH semantics note:** OData v2's MERGE/PATCH doesn't actually tell you *which* fields were included in the body vs. which were omitted — it sends only the supplied fields, but in ABAP they land in your structure with default/initial values for the omitted ones. The pattern shown above (`COND #( WHEN ls_product-price > 0 THEN ... ELSE keep DB value )`) is the practical workaround most teams use. For more robust partial-update support, read the raw HTTP body as a string and parse it yourself — but that's an edge case.

---

## 25.3 DELETE_ENTITY — handling DELETE

### 1️⃣ The analogy

`DELETE /ProductSet('P-100')` — read the key, delete the record. Return HTTP 204 No Content on success. If the record doesn't exist, return 404.

### 2️⃣ You already know this

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

### 3️⃣ The ABAP way — DELETE_ENTITY implementation

```abap
METHOD productset_delete_entity.
  "======================================================================
  " DELETE /ProductSet('P-100')
  "======================================================================

  " ── 1. Read the key ───────────────────────────────────────────────────
  DATA(lv_product_id) = VALUE matnr(
      OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).

  IF lv_product_id IS INITIAL.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid           = /iwbep/cx_mgw_busi_exception=>business_error
        http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
        message          = 'ProductId is required'.
  ENDIF.

  " ── 2. Check that the record exists ──────────────────────────────────
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

  " ── 3. Business rule — e.g., cannot delete if referenced in sales orders
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

  " ── 4. Delete from MAKT and MARA ─────────────────────────────────────
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

  " DELETE_ENTITY has no ER_ENTITY to fill — the framework returns HTTP 204.

ENDMETHOD.
```

> 🧭 **On the job:** You'll rarely do hard DELETEs in SAP. Most entities are "logically deleted" — a deletion flag is set (`MARA-LVORM = 'X'`), and downstream processes (archiving, SARA) handle the physical removal later. Adapt the pattern: instead of `DELETE FROM mara`, do `UPDATE mara SET lvorm = 'X'`.

---

## 25.4 Validation, raising /IWBEP/CX_MGW_BUSI_EXCEPTION, and commit/rollback discipline

### The exception class

`/IWBEP/CX_MGW_BUSI_EXCEPTION` is the *only* exception you should let escape from a DPC_EXT method for expected business errors. The framework catches it and converts it into a well-formed OData error response.

```abap
" HTTP 400 — client sent bad input
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
    message          = 'Descriptive message here'.

" HTTP 404 — record not found
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-not_found
    message          = 'Product not found'.

" HTTP 409 — conflict (duplicate, locked record)
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-conflict
    message          = 'Record already exists'.

" HTTP 500 — unexpected server error (use sparingly)
RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
  EXPORTING
    textid           = /iwbep/cx_mgw_busi_exception=>business_error
    http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-server_error
    message          = 'Unexpected error during processing'.
```

### Multiple messages via the message container

When you have multiple validation errors to report at once (like a form with 3 invalid fields), use the message container instead of raising immediately:

```abap
METHOD productset_create_entity.

  DATA lt_messages TYPE /iwbep/t_mgw_business_msg.
  DATA ls_msg      TYPE /iwbep/s_mgw_business_msg.
  DATA lv_errors   TYPE abap_bool.

  " Collect all errors first
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

  " Raise with all messages at once
  IF lv_errors = abap_true.
    mo_context->get_message_container(
      )->add_messages_from_bapi(
          it_bapi_messages = CORRESPONDING #( lt_messages ) ).
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        message_container = mo_context->get_message_container( ).
  ENDIF.

  " ... continue with INSERT ...

ENDMETHOD.
```

### COMMIT / ROLLBACK discipline

The rules are simple and absolute:

```
Rule 1: Only call COMMIT WORK in a write method (CREATE/UPDATE/DELETE).
        NEVER in GET_ENTITY or GET_ENTITYSET.

Rule 2: Call COMMIT WORK AFTER all database writes succeed.
        If any write fails → ROLLBACK WORK → raise exception.

Rule 3: Use COMMIT WORK AND WAIT (not just COMMIT WORK) so the
        commit is synchronous — you know the data is persisted before
        you return the entity in ER_ENTITY.

Rule 4: If you call a BAPI, check RETURN table for errors BEFORE commit:
        - If any MSGTY = 'E' or 'A' → ROLLBACK WORK → raise exception.
        - If all MSGTY = 'S' or 'W' → COMMIT WORK AND WAIT.
```

```abap
" The BAPI pattern (what you'll do in real projects):
CALL FUNCTION 'BAPI_MATERIAL_SAVEDATA'
  EXPORTING
    headdata        = ls_headdata
  TABLES
    returnmessages  = lt_return.

" Check for errors
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
    wait = abap_true.             " equivalent to COMMIT WORK AND WAIT
```

> ⚠️ **C#/Python gotcha:** There is no automatic unit-of-work / transaction scope in classic SEGW DPC_EXT. Every database write you do accumulates in the SAP LUW (Logical Unit of Work) — a single implicit transaction. `COMMIT WORK` finalises it all. `ROLLBACK WORK` undoes everything since the last commit. If your method does multiple INSERTs and the third one fails, call `ROLLBACK WORK` to undo the first two before raising the exception. Forgetting the rollback leaves partial data in the database.

> ⚠️ **C#/Python gotcha:** Never call `COMMIT WORK` in a read method (`GET_ENTITY`, `GET_ENTITYSET`). Some developers inadvertently commit an open LUW from a previous dialog step. In SEGW/OData calls this is typically not an issue (each HTTP call is a fresh dialog step), but it's a good habit.

### A reusable validation helper

Extract validation into a private method so CREATE and UPDATE can share it:

```abap
" In ZMY_PRODUCT_SRV_DPC_EXT — private method ZM_VALIDATE_PRODUCT

METHOD zm_validate_product.
  "--------------------------------------------------------------
  " Validates IS_PRODUCT. Raises /IWBEP/CX_MGW_BUSI_EXCEPTION
  " if validation fails.
  " IS_FOR_CREATE = abap_true  → ProductId must not be initial
  " IS_FOR_CREATE = abap_false → ProductId may be initial (UPDATE)
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
    " Validate currency against T006A (SAP currency table)
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

Call it at the top of both `CREATE_ENTITY` and `UPDATE_ENTITY`:

```abap
zm_validate_product(
    is_product    = ls_product
    is_for_create = abap_true ).
```

---

## ☕ The complete write-side picture

```mermaid
sequenceDiagram
    participant Client as Fiori / HTTP client
    participant FW as OData Framework
    participant DPC as DPC_EXT method
    participant DB as SAP Database

    Client->>FW: POST /ProductSet (JSON body)
    FW->>DPC: CREATE_ENTITY(io_data_provider, ...)
    DPC->>DPC: read_entry_data() → ls_product
    DPC->>DPC: validate
    alt validation fails
        DPC-->>FW: raise CX_MGW_BUSI_EXCEPTION
        FW-->>Client: HTTP 400 + OData error body
    end
    DPC->>DB: INSERT mara
    DPC->>DB: INSERT makt
    DPC->>DB: COMMIT WORK AND WAIT
    DPC-->>FW: fill er_entity
    FW-->>Client: HTTP 201 + created entity JSON

    style DPC fill:#4CAF50,color:white
```

---

## 🧠 Recap

- **`CREATE_ENTITY`**: read body with `io_data_provider->read_entry_data( IMPORTING es_data = ls_struct )`, validate, INSERT, `COMMIT WORK AND WAIT`, fill `er_entity`. The framework returns HTTP 201.
- **`UPDATE_ENTITY`**: read key from `it_key_tab`, check existence (404 if missing), read body, detect PUT vs PATCH via `io_tech_request_context->get_http_method( )`, UPDATE, commit, fill `er_entity`. Framework returns HTTP 200.
- **`DELETE_ENTITY`**: read key, check existence (404), apply business rules, DELETE, commit. No `er_entity` to fill — framework returns HTTP 204.
- **Error handling**: always raise `/IWBEP/CX_MGW_BUSI_EXCEPTION` with an appropriate HTTP status code. Never let plain `CX_ROOT` exceptions escape — they cause generic 500s with no detail.
- **COMMIT / ROLLBACK rule**: `COMMIT WORK AND WAIT` after all writes succeed; `ROLLBACK WORK` before raising an exception if any write partially succeeded.
- **Real projects use BAPIs**: wrap INSERT/UPDATE/DELETE in a BAPI call, check the RETURN table, then `BAPI_TRANSACTION_COMMIT` / `BAPI_TRANSACTION_ROLLBACK` rather than bare `COMMIT WORK`.

---

*[← Contents](../content.md) | [← Previous: Search Strings & Query Options](24-odata-search-and-query-options.md) | [Next: Associations — Principal & Dependent Entity Sets →](26-odata-associations.md)*
