# Chapter 24: Search Strings & Query Options

*Make your collection endpoint actually useful: $filter, $search, $top/$skip, $orderby, and $inlinecount — all wired up inside GET_ENTITYSET.*

---

## ☕ Why this chapter matters

A GET_ENTITYSET that returns all 50,000 materials with no filter is fine for prototyping and worthless in production. Real Fiori apps send `$filter`, `$top`, `$skip`, `$orderby`, and `$inlinecount=allpages` together. Your job is to translate those OData query options into Open SQL clauses and page them correctly.

This chapter takes the skeleton from ch 23 and makes it production-ready. By the end, one method body handles all five query mechanisms.

---

## 24.1 $filter → WHERE clause

### 1️⃣ The analogy

`$filter` is the OData equivalent of a SQL WHERE clause. The client sends it as a URL parameter and you read it in ABAP as a pre-parsed *select-options* table (SIGN, OPTION, LOW, HIGH) — the same ABAP ranges concept you've already met in reports and selections screens.

### 2️⃣ You already know this

```csharp
// C# Web API — filter comes in as query string, you apply it to IQueryable
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

In OData the client expresses the same thing in the URL:

```http
GET /ProductSet?$filter=Price ge 1000 and Price le 5000 and substringof('Lathe',Name)
```

### 3️⃣ The ABAP way — reading IT_FILTER_SELECT_OPTIONS

```abap
" IT_FILTER_SELECT_OPTIONS is of type /IWBEP/T_MGW_SELECT_OPTION
" Each row is:
"   PROPERTY     = 'Price'     (OData property name)
"   SELECT_OPTIONS = RANGE OF <abap-type>
"       SIGN   = 'I' (include) or 'E' (exclude)
"       OPTION = 'EQ','NE','GT','GE','LT','LE','BT','CP' ...
"       LOW    = '1000.00'
"       HIGH   = '5000.00'  (only for BT — between)

" The classic approach: extract per-field ranges and use SELECT ... WHERE field IN ranges

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

" Now use them in the SELECT:
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
   AND a~stprs IN @lt_price_range     " ← filter applied!
   AND t~maktx IN @lt_name_range      " ← filter applied!
 ORDER BY a~matnr
 UP TO @lv_top ROWS
 OFFSET @lv_skip ROWS.
```

> 💡 If a range table is empty (the client didn't filter on that field), `field IN empty_range_table` matches **everything** — which is exactly what you want. No special `IF lt_price_range IS NOT INITIAL` guard needed.

> ⚠️ **C#/Python gotcha:** The range table rows use ABAP SIGN/OPTION notation, not SQL. `EQ` = `=`, `NE` = `<>`, `GT` = `>`, `GE` = `>=`, `LT` = `<`, `LE` = `<=`, `BT` = `BETWEEN`. `CP` = contains pattern (use with `*` wildcards). The OData framework has already translated `$filter=Price ge 1000` into SIGN=I, OPTION=GE, LOW='1000' for you.

### Using get_filter_select_options() for type-safe extraction

Alternatively, call the request context helper which returns the same data through the OO interface:

```abap
DATA(lt_filter_so) = io_tech_request_context->get_filter(
    )->get_filter_select_options( ).
```

This is identical in content to `it_filter_select_options` but fetched through the interface — useful if you want to do post-processing or logging before extracting ranges.

### The $filter operators OData supports

| OData operator | SQL equivalent | ABAP OPTION |
|---|---|---|
| `eq` | `=` | `EQ` |
| `ne` | `<>` | `NE` |
| `gt` | `>` | `GT` |
| `ge` | `>=` | `GE` |
| `lt` | `<` | `LT` |
| `le` | `<=` | `LE` |
| `and` | `AND` | Multiple range rows (SIGN=I) |
| `or` | `OR` | Multiple range rows (SIGN=I) |
| `substringof('x', field)` | `LIKE '%x%'` | OPTION=`CP`, LOW=`*x*` |
| `startswith(field,'x')` | `LIKE 'x%'` | OPTION=`CP`, LOW=`x*` |

---

## 24.2 Free-text search string ($search)

### 1️⃣ The analogy

`$search=lathe` is like a search box — the client passes a single text string and expects you to search relevant text fields with it. It's not SQL WHERE; it's more like a lightweight full-text search. You decide which fields to search.

### 2️⃣ You already know this

```python
# FastAPI — a q= search param you apply however you want
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

### 3️⃣ The ABAP way

```abap
" Read the free-text search expression
DATA(lv_search) = io_tech_request_context->get_search_expression( ).

" Apply it as a wildcard match across text fields
DATA lt_search_range TYPE RANGE OF maktx.

IF lv_search IS NOT INITIAL.
  " Wrap in wildcards for a CONTAINS-style match
  APPEND VALUE #(
    sign   = 'I'
    option = 'CP'
    low    = |*{ lv_search }*|    " '*lathe*' → LIKE '%lathe%'
  ) TO lt_search_range.
ENDIF.

" Add to the SELECT:
"   AND ( t~maktx IN @lt_search_range
"      OR a~matnr IN @lt_search_range_matnr )
```

> ⚠️ **C#/Python gotcha:** SAP's OData framework doesn't automatically wire `$search` to any field — you implement the search logic yourself. The framework just passes the raw string via `get_search_expression( )`. If you don't call that method, the `$search` parameter is silently ignored. Make sure to document which fields your service searches.

---

## 24.3 Paging & sorting — $top, $skip, $orderby, $inlinecount

### 1️⃣ The analogy

This is exactly like pagination in any REST API — you've done it in C# or Python a dozen times. `$top` = page size, `$skip` = offset, `$orderby` = sort column and direction, `$inlinecount=allpages` = "also tell me the total count so I can render page numbers".

### 2️⃣ You already know this

```csharp
// ASP.NET with OData (Microsoft.AspNetCore.OData) — familiar pattern:
[HttpGet]
[EnableQuery(MaxTop = 500)]
public IQueryable<Product> Get() => _db.Products;
// The framework applies $top/$skip/$orderby automatically.

// Manual equivalent:
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

### 3️⃣ The ABAP way

#### $top and $skip (IS_PAGING)

```abap
" IS_PAGING is of type /IWBEP/S_MGW_PAGING
"   IS_PAGING-top   = integer value of $top  (0 if not supplied)
"   IS_PAGING-skip  = integer value of $skip (0 if not supplied)

DATA(lv_top)  = COND i( WHEN is_paging-top  > 0
                         THEN CONV i( is_paging-top )
                         ELSE 100 ).                  " default page size
DATA(lv_skip) = CONV i( is_paging-skip ).

" Applied in SELECT:
" UP TO @lv_top ROWS OFFSET @lv_skip ROWS
```

#### $orderby (IT_ORDER)

```abap
" IT_ORDER is of type /IWBEP/T_MGW_SORTING_ORDER
" Each row:
"   PROPERTY  = 'Price'
"   ORDER     = /IWBEP/IF_MGW_APPL_SRV_RUNTIME=>GC_SORT_ORDER-ascending
"               or GC_SORT_ORDER-descending

" Build a dynamic ORDER BY string from IT_ORDER
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
  lv_order_by = 'a~matnr ASCENDING'.   " default sort
ENDIF.
```

> ⚠️ **C#/Python gotcha:** ABAP Open SQL does **not** support a plain string variable in an `ORDER BY` clause in older releases. For truly dynamic sorting you have two options: (1) use the `cl_abap_dyn_prog=>check_column_name( )` / `adbc` dynamic SQL approach, or (2) SELECT everything and then sort the internal table with `SORT it_result BY (lv_fieldname)`. Option 2 is safer and common in practice — the cost is you sort after the `UP TO N ROWS`, so paging + sorting must be done carefully (sort first if possible, or accept that dynamic ORDER BY requires a full scan then manual slice).

#### $inlinecount=allpages

```abap
" After the main SELECT, run a COUNT(*) and assign it:
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

The client then receives a `__count` field in the JSON:

```json
{
  "d": {
    "__count": "342",
    "results": [ ... ]
  }
}
```

Fiori SmartTable uses this to render "342 items (showing 1-25)".

---

## 24.4 Full GET_ENTITYSET with all query options

Here is the complete, production-shaped `PRODUCTSET_GET_ENTITYSET` that handles all five query mechanisms together:

```abap
METHOD productset_get_entityset.
  "======================================================================
  " GET /ProductSet
  " Supports: $filter (Price, Name), $search, $top, $skip,
  "           $orderby (ProductId, Price, Name), $inlinecount=allpages
  "======================================================================

  CONSTANTS c_default_page TYPE i VALUE 100.
  CONSTANTS c_max_page      TYPE i VALUE 500.

  " ── Language ─────────────────────────────────────────────────────────
  DATA(lv_langiso) = cl_abap_syst=>get_language( ).

  " ── Paging ───────────────────────────────────────────────────────────
  DATA(lv_top) = COND i(
      WHEN is_paging-top > 0 AND is_paging-top <= c_max_page
      THEN CONV i( is_paging-top )
      WHEN is_paging-top > c_max_page
      THEN c_max_page
      ELSE c_default_page ).
  DATA(lv_skip) = CONV i( is_paging-skip ).

  " ── Filter — extract ranges per filterable property ──────────────────
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

  " ── Free-text search ─────────────────────────────────────────────────
  DATA lt_search_name_range TYPE RANGE OF maktx.
  DATA(lv_search) = io_tech_request_context->get_search_expression( ).

  IF lv_search IS NOT INITIAL.
    " Search in Name field (can extend to other text fields)
    APPEND VALUE #(
        sign   = 'I'
        option = 'CP'
        low    = |*{ lv_search }*|
    ) TO lt_search_name_range.
  ELSE.
    " If no search, pass-through (empty range = match all)
    " Nothing to do — lt_search_name_range stays empty → matches all rows in SELECT
  ENDIF.

  " ── Main SELECT ──────────────────────────────────────────────────────
  DATA lt_products TYPE TABLE OF zcl_zmy_product_srv_mpc=>ts_product
       WITH EMPTY KEY.

  " Note: search AND name filter run as OR-like logic here
  " (if both supplied, we apply both as AND — adjust to your business rule)
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

  " ── Sorting (post-SELECT — safe approach) ────────────────────────────
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

    SORT lt_products BY (lv_sort_field) ASCENDING.  " default ascending

    IF lv_sort_descending = abap_true.
      SORT lt_products BY (lv_sort_field) DESCENDING.
    ENDIF.
  ELSE.
    " Default: sort by ProductId ascending
    SORT lt_products BY product_id ASCENDING.
  ENDIF.

  " ── Paging slice (since sorting is post-SELECT) ───────────────────────
  DATA(lv_total_before_page) = lines( lt_products ).

  " Slice: skip the first lv_skip rows, then take lv_top rows
  DATA lt_paged LIKE lt_products.
  DATA(lv_start) = lv_skip + 1.
  DATA(lv_end)   = lv_skip + lv_top.

  LOOP AT lt_products INTO DATA(ls_prod)
      FROM lv_start TO lv_end.
    APPEND ls_prod TO lt_paged.
  ENDLOOP.

  " ── Return data ───────────────────────────────────────────────────────
  ct_data = CORRESPONDING #( lt_paged ).

  " ── Inline count (total BEFORE paging slice, AFTER filter) ───────────
  IF io_tech_request_context->get_inlinecount( ) = abap_true.
    es_response_context-inlinecount = lv_total_before_page.
  ENDIF.

ENDMETHOD.
```

### The trade-off: SELECT-then-sort vs. ORDER BY in SQL

The implementation above sorts in ABAP after the SELECT. This is the safe, portable approach — no dynamic SQL risks. The downside: for very large tables you pull more rows than needed before slicing.

In practice on production systems:

```
small-medium tables (< 50k rows)  → sort in ABAP, slice in ABAP — fine
large tables (> 100k rows)         → push ORDER BY into SELECT with dynamic SQL
                                      using cl_abap_dyn_prog for column validation
```

For the SEGW/SEGW tutorial scope, the ABAP sort approach is what you'll see on most projects.

> 🧭 **On the job:** Fiori SmartTable will always send `$top=30&$skip=0&$inlinecount=allpages` on first load, then `$top=30&$skip=30` on scroll-down. If you implement paging incorrectly (forgetting to return `inlinecount`, or applying `UP TO N ROWS` before sorting), users will see duplicate rows on page 2 or wrong totals in the table header. Always test with page size 5 to spot these bugs fast in `/IWFND/GW_CLIENT`.

---

### Sample URLs to test the full method

```http
" All products, default paging:
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet?$format=json

" Filter by price range:
GET /ProductSet?$filter=Price ge 1000 and Price le 5000&$format=json

" Free-text search:
GET /ProductSet?$search=lathe&$format=json

" Paging with sort and count:
GET /ProductSet?$top=10&$skip=0&$orderby=Price desc&$inlinecount=allpages&$format=json

" Combined — filter + search + paging:
GET /ProductSet?$filter=Price ge 500&$search=mill&$top=5&$skip=0
    &$orderby=Name asc&$inlinecount=allpages&$format=json
```

---

## 🧠 Recap

- **`IT_FILTER_SELECT_OPTIONS`** gives you per-field ABAP ranges tables ready to drop into `SELECT WHERE field IN range`. Empty range = match everything — no guard needed.
- **`$search`** comes in via `io_tech_request_context->get_search_expression( )` as a plain string. You decide which fields to wildcard-match against it.
- **`IS_PAGING`** has `-top` and `-skip` — use them in `UP TO n ROWS OFFSET m ROWS` (ABAP 7.50+) or post-SELECT loop slicing for older systems.
- **`IT_ORDER`** gives you property + direction. Safest approach: sort the in-memory internal table with `SORT lt BY (dynamic_fieldname) ASCENDING/DESCENDING`.
- **`$inlinecount=allpages`** → check with `io_tech_request_context->get_inlinecount( )`, then set `es_response_context-inlinecount` to the total row count *after* filter *before* paging.
- The full method body stacks all five mechanisms together — each is additive and independent.

---

*[← Contents](../content.md) | [← Previous: Reading Data — GET_ENTITY & GET_ENTITYSET](23-odata-read-crud-get-entity.md) | [Next: Create, Update & Delete →](25-odata-create-update-delete.md)*
