# Chapter 23: Reading Data — GET_ENTITY & GET_ENTITYSET

*Build a complete, working read-only OData service for the Product entity — from SEGW model to live JSON response.*

---

## ☕ This is the chapter where it all clicks

Chapters 21 and 22 were set-up: vocabulary, scaffolding, method signatures. This chapter is the first time you write real ABAP, fire a real HTTP request, and get real JSON back. We build the full read path for the `Product` entity end-to-end.

By the end you will have:
- `PRODUCTSET_GET_ENTITYSET` returning a collection with realistic SELECTs across `MARA` + `MAKT`
- `PRODUCTSET_GET_ENTITY` returning one product by key
- A working test URL in `/IWFND/GW_CLIENT`

---

## 23.1 GET_ENTITYSET — returning the collection

### 1️⃣ The analogy

`GET_ENTITYSET` is the `GET /api/products` endpoint of your Web API. Client calls it, you return a list. The difference is that your "controller" is a redefined ABAP method, and you return data by filling an internal table rather than returning `IEnumerable<T>`.

### 2️⃣ You already know this

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

### 3️⃣ The ABAP way — full GET_ENTITYSET implementation

Open `SE24` → class `ZMY_PRODUCT_SRV_DPC_EXT` → Methods tab → find `PRODUCTSET_GET_ENTITYSET` → right-click **Redefine**.

```abap
METHOD productset_get_entityset.
  "--------------------------------------------------------------------
  " GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet
  " Returns a list of products (joined from MARA + MAKT).
  "--------------------------------------------------------------------

  " ── Types ──────────────────────────────────────────────────────────
  TYPES: BEGIN OF lty_result,
           product_id TYPE matnr,
           name       TYPE maktx,
           price      TYPE p DECIMALS 2,
           currency   TYPE waers,
           created_on TYPE dats,
         END OF lty_result.

  " ── Local data ─────────────────────────────────────────────────────
  DATA lt_result   TYPE TABLE OF lty_result.
  DATA lv_top      TYPE i.
  DATA lv_skip     TYPE i.
  DATA lv_langiso  TYPE laiso.

  " ── 1. Determine the language for MAKT ─────────────────────────────
  lv_langiso = cl_abap_syst=>get_language( ).         " logon language

  " ── 2. Read paging params ──────────────────────────────────────────
  lv_top  = COND #( WHEN is_paging-top  > 0 THEN is_paging-top  ELSE 100 ).
  lv_skip = is_paging-skip.

  " ── 3. SELECT — MARA (header) joined with MAKT (descriptions) ──────
  "    We will layer filters and paging on top of this in ch 24.
  SELECT a~matnr  AS product_id,
         t~maktx  AS name,
         a~stprs  AS price,          " std price from MARA (demo field)
         a~waers  AS currency,
         a~ersda  AS created_on
    INTO CORRESPONDING FIELDS OF TABLE @lt_result
    FROM mara AS a
    LEFT OUTER JOIN makt AS t
      ON  t~matnr = a~matnr
      AND t~spras = @lv_langiso
   WHERE a~lvorm = space                  " not flagged for deletion
   ORDER BY a~matnr
   UP TO @lv_top ROWS
   OFFSET @lv_skip ROWS.

  " ── 4. Map into the MPC-generated entity structure ─────────────────
  "    The generated type is zcl_zmy_product_srv_mpc=>ts_product
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

  " ── 5. Inline count (total rows before paging) ─────────────────────
  "    Only bother if the client asked with $inlinecount=allpages
  IF io_tech_request_context->get_inlinecount( ) = abap_true.
    SELECT COUNT(*) INTO @DATA(lv_total)
      FROM mara
     WHERE lvorm = space.
    es_response_context-inlinecount = lv_total.
  ENDIF.

ENDMETHOD.
```

> 💡 **`OFFSET ... ROWS`** is ABAP 7.50+ Open SQL syntax. On older systems use a post-SELECT loop to skip rows manually, or use `@lv_skip` with a helper variable. Always check your system release (`SE38 → ABAP editor → menu System → Status`).

> ⚠️ **C#/Python gotcha:** `ct_data` is typed as a generic table (`ANY TABLE`) — the framework maps it dynamically to the entity type. You can fill it either by mapping to `zcl_..._mpc=>ts_product` (safest — compiler checks your field names) or by using `CORRESPONDING #( )`. Never use `ct_data` directly as if it's the entity table — cast it or fill a typed local table first and then `ct_data = CORRESPONDING #( lt_typed )`.

---

## 23.2 GET_ENTITY — returning one record by key

### 1️⃣ The analogy

`GET_ENTITY` is `GET /api/products/{id}`. One key, one record back. If not found, throw a 404.

### 2️⃣ You already know this

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

### 3️⃣ The ABAP way — full GET_ENTITY implementation

```abap
METHOD productset_get_entity.
  "--------------------------------------------------------------------
  " GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')
  " Returns a single product by its key.
  "--------------------------------------------------------------------

  " ── 1. Read the key from IT_KEY_TAB ────────────────────────────────
  "    IT_KEY_TAB is of type /IWBEP/T_MGW_TECH_PAIRS
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

  " ── 2. SELECT from MARA + MAKT ─────────────────────────────────────
  DATA lv_langiso TYPE laiso.
  lv_langiso = cl_abap_syst=>get_language( ).

  DATA ls_mara TYPE mara.
  DATA ls_makt TYPE makt.

  SELECT SINGLE * FROM mara INTO @ls_mara WHERE matnr = @lv_product_id.
  IF sy-subrc <> 0.
    " Standard OData "not found" exception — framework returns HTTP 404
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

  " ── 3. Fill the output entity ──────────────────────────────────────
  er_entity-product_id  = ls_mara-matnr.
  er_entity-name        = lv_name.
  er_entity-price       = ls_mara-stprs.
  er_entity-currency    = ls_mara-waers.
  er_entity-created_on  = ls_mara-ersda.

ENDMETHOD.
```

> 🧭 **On the job:** In real projects `er_entity` is declared as `TYPE REF TO data` (a generic data reference). The framework derefs it into the correct MPC structure. In the redefined method body you'll see it typed to `zcl_..._mpc=>ts_product` — just fill it like a normal structure.

---

## 23.3 Reading the key — it_key_tab vs get_keys()

There are two equivalent ways to extract the key value. Know both — you'll see both on the job.

### Pattern A: Direct table read (simplest, most common)

```abap
" Reads directly from the import parameter table
DATA(lv_id) = it_key_tab[ name = 'ProductId' ]-value.

" Safe variant with OPTIONAL (no exception if key not found)
DATA(lv_id_safe) = VALUE #( OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).
```

### Pattern B: Using get_keys() on io_tech_request_context

```abap
" The OO way — useful if you want richer metadata about the key
io_tech_request_context->get_keys(
  IMPORTING
    et_key_tab = DATA(lt_keys) ).

DATA(ls_key) = VALUE /iwbep/s_mgw_tech_pair(
    OPTIONAL
    ( lt_keys[ name = 'ProductId' ] ) ).

DATA(lv_product_id) = ls_key-value.
```

### Composite keys (multiple key fields)

For `SalesOrderItem` which has two keys (`SalesOrderId` and `ItemNumber`):

```abap
DATA(lv_order_id) = it_key_tab[ name = 'SalesOrderId' ]-value.
DATA(lv_item_no)  = it_key_tab[ name = 'ItemNumber'   ]-value.
```

> ⚠️ **C#/Python gotcha:** The `name` field in `IT_KEY_TAB` is the **OData property name** (camelCase, as defined in SEGW — e.g. `'ProductId'`), not the ABAP field name (`product_id`). They often differ in casing. If you get an empty value, check the exact property name in the SEGW model.

---

## 23.4 A complete read-only service end-to-end

Let's put it all together. Here's everything you need to have a fully working read service for `Product`.

### What you've done in SEGW (recap from ch 21)

1. Created project `ZMY_PRODUCT_SRV`
2. Defined Entity Type `Product` with properties `ProductId` (key), `Name`, `Price`, `Currency`, `CreatedOn`
3. Generated — got classes `ZMY_PRODUCT_SRV_DPC_EXT`, `ZMY_PRODUCT_SRV_MPC_EXT`
4. Registered in `/IWFND/MAINT_SERVICE`

### The two methods in `ZMY_PRODUCT_SRV_DPC_EXT` (full, clean version)

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
" GET /ProductSet  — return the collection
"======================================================================
  METHOD productset_get_entityset.

    CONSTANTS c_max_rows TYPE i VALUE 500.    " hard safety cap

    " Determine logon language for MAKT
    DATA(lv_langiso) = cl_abap_syst=>get_language( ).

    " Paging
    DATA(lv_top)  = COND i( WHEN is_paging-top  > 0
                             THEN CONV i( is_paging-top )
                             ELSE c_max_rows ).
    DATA(lv_skip) = CONV i( is_paging-skip ).

    " Result type matched to generated MPC structure
    DATA lt_products TYPE TABLE OF zcl_zmy_product_srv_mpc=>ts_product.

    " Main SELECT — MARA header + MAKT description
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

    " Inline count (total before paging)
    IF io_tech_request_context->get_inlinecount( ) = abap_true.
      SELECT COUNT(*) INTO @DATA(lv_total)
        FROM mara WHERE lvorm = space.
      es_response_context-inlinecount = lv_total.
    ENDIF.

  ENDMETHOD.

"======================================================================
" GET /ProductSet('P-100')  — return one entity by key
"======================================================================
  METHOD productset_get_entity.

    " Extract key
    DATA(lv_product_id) = VALUE matnr(
        OPTIONAL ( it_key_tab[ name = 'ProductId' ]-value ) ).

    IF lv_product_id IS INITIAL.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid           = /iwbep/cx_mgw_busi_exception=>business_error
          http_status_code = /iwbep/cx_mgw_busi_exception=>gcs_http_status_codes-bad_request
          message          = 'ProductId is required'.
    ENDIF.

    " Language
    DATA(lv_langiso) = cl_abap_syst=>get_language( ).

    " Read header
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

    " Read description
    SELECT SINGLE maktx INTO @DATA(lv_name)
      FROM makt
     WHERE matnr = @lv_product_id
       AND spras = @lv_langiso.

    " Fill output
    er_entity-product_id  = ls_mara-matnr.
    er_entity-name        = lv_name.
    er_entity-price       = ls_mara-stprs.
    er_entity-currency    = ls_mara-waers.
    er_entity-created_on  = ls_mara-ersda.

  ENDMETHOD.

ENDCLASS.
```

### Testing in /IWFND/GW_CLIENT

Transaction `/IWFND/GW_CLIENT` is your Postman. Here's how to use it:

1. Run `/IWFND/GW_CLIENT`
2. In the **Request URI** field, enter the relative path:
   - Collection: `/sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet?$format=json`
   - Single entity: `/sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')?$format=json`
3. Click **Execute** (F8)
4. Watch the response in the lower half

### What you should see — collection response

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

### What you should see — single entity response

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

### What a 404 looks like

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

> 🧭 **On the job:** When a Fiori app shows a generic "An error has occurred" message, open `/IWFND/ERROR_LOG` (or `/IWFND/APPS_LOG`). These are your server-side OData error logs — they show the full exception stack trace even when the HTTP response is stripped. This is the first place you check in every OData debugging session.

---

## 🧠 Recap

- **`GET_ENTITYSET`** = the collection endpoint. SELECT into a typed local table, copy to `ct_data`, optionally set `es_response_context-inlinecount`.
- **`GET_ENTITY`** = the single-record endpoint. Read the key from `it_key_tab`, SELECT single, fill `er_entity`, raise a 404 exception if not found.
- **Key extraction:** `it_key_tab[ name = 'PropertyName' ]-value` (direct table expression) — the most common pattern. The `name` uses the OData property name (as in SEGW), not the ABAP field name.
- **Not-found handling:** Raise `/IWBEP/CX_MGW_BUSI_EXCEPTION` with `http_status_code = gcs_http_status_codes-not_found`. The framework converts it to HTTP 404 with a proper OData error body.
- **Test tool:** `/IWFND/GW_CLIENT` is Postman inside the SAP system. Use it for every round of testing.
- **Debug tool:** `/IWFND/ERROR_LOG` shows full server-side exception traces for OData calls.

---

*[← Contents](../content.md) | [← Previous: Service Methods & Import Parameters](22-odata-methods-and-parameters.md) | [Next: Search Strings & Query Options →](24-odata-search-and-query-options.md)*
