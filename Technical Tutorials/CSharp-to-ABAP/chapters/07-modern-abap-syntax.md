# Chapter 7: Modern ABAP Syntax (7.4+) — The Good Stuff

*ABAP got a serious upgrade. It now looks closer to C# and Python than you'd believe — and this is what makes it fun.*

---

## ☕ The upgrade you didn't know happened

Here's a secret that most ABAP job postings don't mention: the ABAP you find on modern SAP systems (anything on ABAP 7.4+ — which covers most active SAP ERP and all S/4HANA installations) looks remarkably like C# or Python.

Inline declarations. Collection expressions. String templates. Functional table operations. It's not perfect — ABAP still has its quirks — but the gap between modern ABAP and modern C# is a lot smaller than the SAP reputation suggests. When I show C# developers modern ABAP for the first time, the usual reaction is: *"Oh. That's... actually fine."*

This chapter is the payoff. We take the verbose classic patterns from Chapter 6 and rewrite them the modern way. Then we put classic and modern side by side so you can read both (because production codebases are a mix).

> 🧭 **On the job:** Not every system runs 7.4+, and not every codebase uses modern syntax even when it could. Check your system's ABAP release with `SE38` → run `DEMO_ABAP_LANGUAGE_VERSION`. If you're on S/4HANA, you're almost certainly on 7.55+. If you're on an ancient ECC system, you might be on 7.02 and some of this chapter won't compile. Know your release.

---

## 7.1 Inline declarations — DATA(x) and FINAL(x)

### 1️⃣ The analogy

Classic ABAP required every variable to be declared at the top of the method, before any executable code — like old C code or Pascal. ABAP 7.4 introduced inline declarations: you declare a variable right where you first assign it, and the compiler infers the type.

This is exactly `var` in C# or implicit type inference in Python.

### 2️⃣ You already know this

```csharp
// C# — var infers type at declaration point
var customer = db.Customers.Find("CUST01");   // type = Customer?
var count    = customers.Count();              // type = int
var name     = customer?.Name ?? "Unknown";    // type = string
```

```python
# Python — always implicit
customer = get_customer("CUST01")   # type inferred
count    = len(customers)
name     = customer.name if customer else "Unknown"
```

### 3️⃣ The ABAP way

```abap
" Classic ABAP — declare everything at the top
DATA lv_customer_id TYPE kunnr.
DATA ls_customer    TYPE kna1.
DATA lt_orders      TYPE STANDARD TABLE OF zorders WITH EMPTY KEY.

lv_customer_id = 'CUST0001'.
SELECT SINGLE * FROM kna1 INTO ls_customer WHERE kunnr = lv_customer_id.
SELECT * FROM zorders INTO TABLE lt_orders WHERE kunnr = lv_customer_id.

" ---------------------------------------------------------------

" Modern ABAP 7.4+ — DATA() inline declaration
" Type is inferred from the right-hand side context

DATA(lv_customer_id) = 'CUST0001'.   " type = string literal context

SELECT SINGLE * FROM kna1
  INTO @DATA(ls_customer)             " declares ls_customer as TYPE kna1
  WHERE kunnr = @lv_customer_id.

SELECT * FROM zorders
  INTO TABLE @DATA(lt_orders)         " declares lt_orders as internal table
  WHERE kunnr = @lv_customer_id.

" In method calls:
DATA(lo_handler) = NEW zcl_order_handler( ).   " declared + instantiated inline
DATA(lv_result)  = lo_handler->calculate( lv_amount ).

" LOOP AT with inline declaration:
LOOP AT lt_orders INTO DATA(ls_order).
  DATA(lv_tax) = ls_order-amount * '0.19'.   " declared on first use
  WRITE: / ls_order-order_id, lv_tax.
ENDLOOP.
```

**`FINAL` — the const/readonly equivalent (ABAP 7.56+):**

```abap
" FINAL declares an immutable variable — like 'const' or 'readonly'
FINAL(lv_tax_rate) = '0.19'.
" lv_tax_rate = '0.21'.   ← compile error: cannot assign to FINAL

" Very useful for loop variables you never intend to modify:
LOOP AT lt_orders INTO FINAL(ls_order).
  " ls_order is read-only here — self-documenting + enforced
  WRITE: / ls_order-amount.
ENDLOOP.
```

> 💡 Prefer `FINAL` over `DATA` in loops when you're only reading. It communicates intent and prevents accidental in-loop assignments.

---

## 7.2 VALUE #( ) and CORRESPONDING #( ) — collection expressions

### 1️⃣ The analogy

`VALUE #( )` is ABAP's collection/struct initializer — like C#'s object initializer `new T { ... }` or Python's dictionary literal `{"key": "val"}`. It constructs a value (a structure or a table) inline, without temporary variables.

`CORRESPONDING #( )` copies fields between two structures or tables that share field names — like AutoMapper in C# or `{**dict1, **dict2}` style merging in Python.

### 2️⃣ You already know this

```csharp
// C# object initializer
var order = new Order
{
    OrderId    = "ORD1",
    CustomerId = "CUST01",
    Amount     = 100m,
    Currency   = "EUR"
};

// C# collection initializer
var orders = new List<Order>
{
    new Order { OrderId = "ORD1", CustomerId = "C1", Amount = 100m },
    new Order { OrderId = "ORD2", CustomerId = "C1", Amount = 200m },
};

// AutoMapper / BeanUtils — copy matching fields
var dto = mapper.Map<OrderDto>(order);
```

```python
# Python struct/dict init
order = {"order_id": "ORD1", "customer_id": "CUST01", "amount": 100.0}

orders = [
    {"order_id": "ORD1", "customer_id": "C1", "amount": 100.0},
    {"order_id": "ORD2", "customer_id": "C1", "amount": 200.0},
]

# Copy matching fields between dicts (simple version)
dto = {k: order[k] for k in ("order_id", "amount")}
```

### 3️⃣ The ABAP way — VALUE #( )

```abap
" VALUE #( ) for a structure — constructor expression
DATA(ls_order) = VALUE zorders(
  order_id    = 'ORD1'
  kunnr       = 'CUST01'
  amount      = 100
  waers       = 'EUR'
).

" VALUE #( ) for an internal table — list of rows
DATA(lt_orders) = VALUE zorders_tab(
  ( order_id = 'ORD1'  kunnr = 'CUST01'  amount = 100  waers = 'EUR' )
  ( order_id = 'ORD2'  kunnr = 'CUST01'  amount = 200  waers = 'EUR' )
  ( order_id = 'ORD3'  kunnr = 'CUST02'  amount =  50  waers = 'USD' )
).

" VALUE #( ) — the # means "infer type from context"
" When the type is known (e.g., from DATA declaration), use #:
DATA lt_known TYPE STANDARD TABLE OF zorders WITH EMPTY KEY.
lt_known = VALUE #(
  ( order_id = 'ORD4'  kunnr = 'CUST03'  amount = 300 )
).

" Add rows to an existing table with VALUE #( BASE ... )
lt_known = VALUE #( BASE lt_known   " keep existing rows
  ( order_id = 'ORD5'  kunnr = 'CUST04'  amount = 400 )
).

" Initialize / clear a structure to initial values
CLEAR ls_order.           " old way (still valid)
ls_order = VALUE #( ).    " modern way — explicit empty construction
```

**CORRESPONDING #( ) — field-by-field copy:**

```abap
" Copy matching fields from one structure to another
" (Only fields with the same name AND compatible type are copied)

DATA ls_bapi_order TYPE bapiordhdr.   " BAPI header structure
DATA ls_my_order   TYPE zorders.      " my custom structure

ls_my_order-order_id = 'ORD1'.
ls_my_order-kunnr    = 'CUST01'.
ls_my_order-amount   = 500.

" CORRESPONDING copies fields where names match
ls_bapi_order = CORRESPONDING #( ls_my_order ).
" → only copies fields that exist in BOTH structures with the same name

" CORRESPONDING for tables:
DATA lt_bapi_orders TYPE STANDARD TABLE OF bapiordhdr WITH EMPTY KEY.
lt_bapi_orders = CORRESPONDING #( lt_orders ).

" CORRESPONDING with MAPPING — rename fields during copy
ls_bapi_order = CORRESPONDING #( ls_my_order
  MAPPING kunnr = kunnr_ext   " map ls_my_order-kunnr_ext → ls_bapi_order-kunnr
).

" CORRESPONDING with EXCEPT — exclude certain fields
ls_bapi_order = CORRESPONDING #( ls_my_order
  EXCEPT amount waers   " don't copy these fields
).
```

> ⚠️ **C#/Python gotcha:** `CORRESPONDING` is tempting but dangerous with large structures — it silently skips fields it can't match. If a field name changes in either structure, the copy silently stops for that field. Use it thoughtfully; document what it's mapping.

---

## 7.3 String templates — |Hello { lv_name }|

### 1️⃣ The analogy

Classic ABAP string concatenation was painful — `CONCATENATE str1 str2 INTO result`. ABAP 7.4 introduced string templates: inline expressions wrapped in `|...|` with `{ variable }` placeholders. It's C#'s `$"..."` interpolation or Python's `f"..."` f-strings.

### 2️⃣ You already know this

```csharp
// C# string interpolation
string name = "Acme GmbH";
decimal amount = 12500.50m;
string msg = $"Invoice for {name}: EUR {amount:N2}";
// → "Invoice for Acme GmbH: EUR 12,500.50"
```

```python
# Python f-string
name   = "Acme GmbH"
amount = 12500.50
msg    = f"Invoice for {name}: EUR {amount:,.2f}"
# → "Invoice for Acme GmbH: EUR 12,500.50"
```

### 3️⃣ The ABAP way

```abap
DATA lv_name   TYPE string VALUE 'Acme GmbH'.
DATA lv_amount TYPE p DECIMALS 2 VALUE '12500.50'.

" Simple string template
DATA(lv_msg) = |Invoice for { lv_name }: EUR { lv_amount }|.

" Formatting options inside { }
DATA(lv_date_str) = |Today is { sy-datum DATE = USER }|.
" DATE = USER → formats according to user's date format settings
" DATE = ISO  → YYYY-MM-DD
" DATE = RAW  → YYYYMMDD (internal ABAP format)

DATA(lv_num_str) = |Amount: { lv_amount NUMBER = USER }|.
" NUMBER = USER → formats with thousands separator per user locale

" Chaining / multi-line strings
DATA(lv_body) = |Dear { lv_name },| &&
                |Your order total is { lv_amount }.| &&
                |Thank you.|.
" && is string concatenation (replaces old CONCATENATE)

" Conditional inside template (ABAP 7.56+)
DATA(lv_status_text) = |Status: { COND #( WHEN lv_active = 'X' THEN 'Active' ELSE 'Inactive' ) }|.

" Classic ABAP concatenation — you'll see this in old code
DATA lv_old_msg TYPE string.
CONCATENATE 'Invoice for ' lv_name ': EUR ' lv_amount
  INTO lv_old_msg SEPARATED BY space.   " clunky, gone from modern code
```

**String operations (modern):**

```abap
" Length
DATA(lv_len) = strlen( lv_name ).           " like .Length

" Contains / find substring
IF lv_name CS 'GmbH'.                       " CS = Contains String
  " ...
ENDIF.

" Replace
DATA(lv_clean) = replace( val = lv_name  sub = ' GmbH'  with = '' ).

" To upper/lower
DATA(lv_upper) = to_upper( lv_name ).
DATA(lv_lower) = to_lower( lv_name ).

" Trim
DATA(lv_trimmed) = condense( lv_name ).     " removes leading/trailing spaces
```

---

## 7.4 Modern SELECT — inline INTO, single, aggregations, @ host variables

### 1️⃣ The analogy

ABAP 7.4+ brought Open SQL much closer to standard SQL syntax: inline `INTO DATA(...)`, `@` host variable prefix, subqueries, common table expressions, and CAST. This is the SQL you want to write on modern SAP systems.

### 2️⃣ You already know this

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
# SQLAlchemy 2.0 style
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

### 3️⃣ The ABAP way

```abap
DATA lv_customer_id TYPE kunnr  VALUE 'CUST0001'.
DATA lv_min_amount  TYPE wrbtr  VALUE 500.

" Full modern SELECT — @ host vars, inline INTO, ORDER BY
SELECT order_id, kunnr, amount, waers
  FROM zorders
  INTO TABLE @DATA(lt_orders)
  WHERE kunnr       = @lv_customer_id
    AND amount      > @lv_min_amount
  ORDER BY amount DESCENDING.

" SELECT SINGLE — primary key lookup, inline
SELECT SINGLE name1, land1, ort01
  FROM kna1
  INTO @DATA(ls_addr)
  WHERE kunnr = @lv_customer_id.

IF sy-subrc <> 0.
  RAISE EXCEPTION TYPE zcx_not_found.
ENDIF.

" Aggregation
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

" JOIN — cleaner in modern syntax
SELECT o~order_id,
       o~amount,
       o~waers,
       c~name1   AS customer_name,
       c~land1   AS country
  FROM zorders   AS o
  INNER JOIN kna1 AS c  ON  o~kunnr  = c~kunnr
                        AND o~mandt  = c~mandt    " explicit MANDT join
  INTO TABLE @DATA(lt_enriched)
  WHERE o~kunnr = @lv_customer_id.

" CAST — convert types in SQL (ABAP 7.5+)
SELECT order_id,
       CAST( amount AS DEC( 15,2 ) ) AS amount_dec
  FROM zorders
  INTO TABLE @DATA(lt_cast)
  WHERE kunnr = @lv_customer_id.

" UP TO N ROWS — pagination / first-N
SELECT *
  FROM zorders
  INTO TABLE @DATA(lt_latest)
  ORDER BY erdat DESCENDING
  UP TO 10 ROWS.
```

> 💡 **`SELECT SINGLE` vs `SELECT ... UP TO 1 ROWS`:** Use `SELECT SINGLE` only when you know the WHERE clause returns exactly one row (primary key). For "give me the top row of many," use `UP TO 1 ROWS` with `ORDER BY` — it's semantically clear and the optimizer handles it well.

**Sub-queries and Common Table Expressions (ABAP 7.55+):**

```abap
" Subquery in WHERE
SELECT order_id, amount
  FROM zorders
  INTO TABLE @DATA(lt_vip_orders)
  WHERE kunnr IN ( SELECT kunnr FROM zkna1_vip WHERE vip_flag = 'X' ).

" With CTE (Common Table Expression) — ABAP 7.55+
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

## 7.5 Table expressions, FILTER, REDUCE, and FOR — ABAP's answer to LINQ

### 1️⃣ The analogy

ABAP 7.4 introduced functional operators for internal tables that map almost 1:1 to LINQ in C# and list comprehensions in Python. This is the part that feels most like modern programming.

### 2️⃣ You already know this

```csharp
// LINQ equivalents
var allOrders = new List<Order> { ... };

// Single-item lookup
var order = allOrders.Single(o => o.OrderId == "ORD1");

// Filter to new list
var eurOrders = allOrders.Where(o => o.Currency == "EUR").ToList();

// Transform / project
var amounts   = allOrders.Select(o => o.Amount).ToList();

// Aggregate
var total = allOrders.Sum(o => o.Amount);
var max   = allOrders.Max(o => o.Amount);

// Collect into new list with transformation
var summaries = allOrders
    .Select(o => new OrderSummary(o.OrderId, o.Amount * 1.1m))
    .ToList();
```

```python
orders = [...]   # list of dicts or dataclass instances

# Filter
eur_orders = [o for o in orders if o["currency"] == "EUR"]

# Transform
amounts = [o["amount"] for o in orders]

# Aggregate
total = sum(o["amount"] for o in orders)

# Filter + transform
summaries = [
    {"id": o["order_id"], "adjusted": o["amount"] * 1.1}
    for o in orders
]
```

### 3️⃣ The ABAP way

**Table expressions — itab[ key = value ]:**

```abap
" Direct row access by key — replaces READ TABLE for single lookups
" Works like dictionary[] access — raises cx_sy_itab_line_not_found if missing

" Access a row from a standard table (key lookup)
DATA(ls_order) = lt_orders[ order_id = 'ORD1' ].   " exception if not found

" Safe access with VALUE operator (returns initial value if not found, no exception)
DATA(ls_safe) = VALUE #( lt_orders[ order_id = 'ORD1' ] OPTIONAL ).

" Read a single field from a table row
DATA(lv_amount) = lt_orders[ order_id = 'ORD1' ]-amount.

" Modify a field inline (for standard tables without key definition)
lt_orders[ order_id = 'ORD1' ]-amount = 999.
```

**FILTER — the WHERE for internal tables:**

```abap
" FILTER creates a new table containing only the matching rows
" Like LINQ Where() or Python list comprehension with condition

" Filter to only EUR orders
DATA(lt_eur) = FILTER #( lt_orders WHERE waers = 'EUR' ).

" Filter with complex conditions (requires sorted/hashed table on the key)
DATA(lt_big) = FILTER #( lt_orders USING KEY primary_key WHERE amount > 500 ).

" Compare to old way:
DATA lt_eur_old TYPE STANDARD TABLE OF zorders WITH EMPTY KEY.
LOOP AT lt_orders INTO DATA(ls_o) WHERE waers = 'EUR'.
  APPEND ls_o TO lt_eur_old.
ENDLOOP.
" → FILTER replaces this entire loop
```

**REDUCE — aggregate to a scalar:**

```abap
" REDUCE computes a single value from a table
" Like LINQ Aggregate() / Sum() or Python functools.reduce() / sum()

" Sum all amounts
DATA(lv_total) = REDUCE wrbtr(
  INIT sum = VALUE wrbtr( )
  FOR  row IN lt_orders
  NEXT sum = sum + row-amount
).

" Count rows matching a condition
DATA(lv_eur_count) = REDUCE i(
  INIT cnt = 0
  FOR  row IN lt_orders WHERE ( waers = 'EUR' )
  NEXT cnt = cnt + 1
).

" Build a concatenated string (like String.Join)
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

**FOR — table comprehension (like Python list comprehension or LINQ Select):**

```abap
" FOR creates a new table by iterating over another
" Like LINQ Select() or Python [expr for item in list]

" Project: create a table of just IDs and amounts (no currency)
TYPES: BEGIN OF ty_summary,
         order_id TYPE zorders-order_id,
         amount   TYPE zorders-amount,
       END OF ty_summary.

DATA(lt_summary) = VALUE ty_summary_tab(
  FOR row IN lt_orders
  ( order_id = row-order_id  amount = row-amount )
).

" FOR with WHERE condition (filter + project combined)
DATA(lt_eur_summary) = VALUE ty_summary_tab(
  FOR row IN lt_orders WHERE ( waers = 'EUR' )
  ( order_id = row-order_id  amount = row-amount )
).

" FOR with transformation
DATA(lt_with_tax) = VALUE ty_summary_tab(
  FOR row IN lt_orders
  ( order_id = row-order_id  amount = row-amount * '1.19' )
).
```

**COND #( ) — inline conditional (ternary):**

```abap
" COND #( WHEN ... THEN ... WHEN ... THEN ... ELSE ... )
" Like C# ?:  or  Python (a if cond else b)

DATA(lv_label) = COND string(
  WHEN lv_amount > 10000 THEN 'Large'
  WHEN lv_amount > 1000  THEN 'Medium'
  ELSE                        'Small'
).

" Compare to classic:
DATA lv_old_label TYPE string.
IF lv_amount > 10000.
  lv_old_label = 'Large'.
ELSEIF lv_amount > 1000.
  lv_old_label = 'Medium'.
ELSE.
  lv_old_label = 'Small'.
ENDIF.
```

**SWITCH #( ) — cleaner CASE/WHEN for expressions:**

```abap
" SWITCH #( ) is CASE as an expression
DATA(lv_currency_symbol) = SWITCH string( lv_currency
  WHEN 'EUR' THEN '€'
  WHEN 'USD' THEN '$'
  WHEN 'GBP' THEN '£'
  ELSE            lv_currency
).
```

---

## 7.6 The big before/after — classic to modern rewrite

Let's take a realistic, ugly classic ABAP snippet and rewrite it. This is the transformation you'll actually do on the job when modernizing a codebase.

### Classic ABAP (circa 2005 style)

```abap
" Classic: verbose, temp variables everywhere, no inline anything
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

### Modern ABAP (7.4+ rewrite)

```abap
" Modern: inline declarations, VALUE #(), REDUCE, table expressions
METHOD get_customer_totals.

  " Single SELECT, inline type
  SELECT order_id, kunnr, amount, waers
    FROM zorders
    INTO TABLE @DATA(lt_orders)
    WHERE kunnr = @iv_customer
      AND waers = 'EUR'.

  CHECK sy-subrc = 0.   " CHECK exits the method if condition is false

  " REDUCE replaces the accumulator loop
  DATA(lv_total) = REDUCE wrbtr(
    INIT sum = VALUE wrbtr( )
    FOR  row IN lt_orders
    NEXT sum = sum + row-amount
  ).

  " Table expression + VALUE #( ) replace the READ/MODIFY/APPEND logic
  IF lt_result[ kunnr = iv_customer ] IS INITIAL.           " not found
    INSERT VALUE #( kunnr = iv_customer  total = lv_total  waers = 'EUR' )
      INTO TABLE et_result.
  ELSE.
    et_result[ kunnr = iv_customer ]-total = lv_total.      " direct field set
  ENDIF.

ENDMETHOD.
```

The modern version is shorter, more expressive, and communicates intent directly. No `lv_found` temp variable. No `CLEAR`. No `APPEND/MODIFY` dance. Just: select → reduce → upsert.

**C# equivalent for comparison:**

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

The modern ABAP version is structurally very close to both the C# and Python versions. The concepts map directly — you just need to remember the ABAP spellings.

---

## 7.7 Quick reference — classic vs modern side by side

| What you want | Classic ABAP | Modern ABAP (7.4+) | C# parallel |
|---|---|---|---|
| Declare variable | `DATA lv_x TYPE i.` | `DATA(lv_x) = 0.` | `var x = 0;` |
| Declare immutable | (no equivalent) | `FINAL(lv_x) = 0.` | `const int x = 0;` |
| Struct literal | `ls-field = val. ...` | `VALUE ty( field = val )` | `new T { Field = val }` |
| Table literal | `APPEND ls TO lt.` | `VALUE #( (f=v) (f=v) )` | `new List<T> { ... }` |
| String interpolation | `CONCATENATE a b INTO s.` | `\|text { var } more\|` | `$"text {var} more"` |
| Ternary | `IF/ELSE/ENDIF` | `COND #( WHEN ... )` | `cond ? a : b` |
| CASE expression | `CASE/WHEN/ENDCASE` | `SWITCH #( ... )` | `switch(x) { case ... }` |
| Loop iterate+read | `LOOP + READ TABLE` | `itab[ key = val ]` | `list.Single(x=>...)` |
| Filter table | `LOOP + APPEND` | `FILTER #(...)` | `.Where(...)` |
| Aggregate | `LOOP + accumulate` | `REDUCE #(...)` | `.Sum()/.Aggregate()` |
| Project table | `LOOP + APPEND` | `VALUE #( FOR row IN ... )` | `.Select(...)` |
| Copy matching fields | manual field by field | `CORRESPONDING #(...)` | AutoMapper |

---

## 🧠 Recap

- **`DATA(x)`** declares variables inline — no more top-of-method declarations for everything. `FINAL(x)` makes them immutable.
- **`VALUE #( )`** builds structs and tables inline — the ABAP object/collection initializer.
- **`CORRESPONDING #( )`** copies fields between compatible structures by name — use with care.
- **String templates** `|Hello { name }|` replace `CONCATENATE` — they support format options like `DATE = USER` and `NUMBER = USER`.
- **Modern SELECT** uses `@` host variables, inline `INTO @DATA(...)`, aggregations, JOINs, and CTEs — close to standard SQL.
- **Table expressions** `itab[ key = val ]` replace most `READ TABLE` calls.
- **FILTER / REDUCE / FOR** map directly to LINQ / Python comprehensions: filter a table, aggregate to a scalar, project to a new table.
- **`COND #( )` and `SWITCH #( )`** are inline conditional expressions — the ternary operators of ABAP.
- Modern ABAP looks much closer to C# and Python than the legacy syntax. On S/4HANA (7.4+), you should be writing modern ABAP — your code will be cleaner and your colleagues will thank you.

---

*[← Contents](../content.md) | [← Previous: ABAP Syntax Basics](06-abap-syntax-basics.md) | [Next: Reports: Classical & ALV →](08-reports-classical-and-alv.md)*
