# Chapter 6: ABAP Syntax Basics for C#/Python Devs

*New spellings for things you already do — and the one mindset shift that matters most.*

---

## ☕ The honest orientation

ABAP is verbose. It's wordy in a way that reads almost like English, which was intentional in the 1980s when it was designed for business developers who weren't programmers. Coming from C# or Python, your first reaction to ABAP will probably be: *"Why does this need so many words?"*

Lean into it. Once you accept the verbosity, ABAP is actually quite readable. A `LOOP AT customers INTO work_area.` is harder to misread than `foreach (var c in customers)`. And once you know the syntax, the patterns repeat endlessly — SAP's codebase is remarkably consistent because everyone's writing the same language in the same way.

This chapter covers the everyday syntax you'll use in 90% of your real tickets. We go fast because you already know how to program — you just need the new spellings.

> ⚠️ **One massive mindset shift upfront:** In C#/Python, errors throw exceptions. In ABAP, most operations set a return code in the system variable `sy-subrc`. **If you don't check `sy-subrc`, you silently proceed on failure.** This is the single biggest trap for developers coming from exception-based languages. We'll beat this drum throughout the chapter.

---

## 6.1 Variables & Types — DATA, TYPES, and the built-in types

### 1️⃣ The analogy

Variable declaration in ABAP is similar to C# — you declare before you use (in classic ABAP; Chapter 7 covers inline `DATA(x)`). The types map directly to what you know, just with different names.

### 2️⃣ You already know this

```csharp
// C# declarations
int    count    = 0;
string name     = "Acme";
decimal amount  = 12.50m;
bool   isActive = true;
DateTime orderDate = DateTime.Today;
```

```python
# Python — dynamic, but type hints show the intent
count:     int   = 0
name:      str   = "Acme"
amount:    float = 12.50
is_active: bool  = True
from datetime import date
order_date: date = date.today()
```

### 3️⃣ The ABAP way

```abap
" Basic variable declarations (at the top of the method/program)
DATA lv_count    TYPE i.           " integer (4 bytes, ~2 billion range)
DATA lv_name     TYPE string.      " dynamic-length string
DATA lv_amount   TYPE p LENGTH 10 DECIMALS 2.  " packed decimal
DATA lv_flag     TYPE xfeld.       " CHAR(1) 'X'=true, ' '=false
DATA lv_date     TYPE d.           " date: YYYYMMDD as 8-char numeric text
DATA lv_time     TYPE t.           " time: HHMMSS as 6-char numeric text

" Initial values — ABAP zero-initializes by default:
" i → 0,  string → '',  p → 0.00,  d → '00000000',  c → '   '
" No null references for value types (unlike C#)

" Constants
CONSTANTS c_max_items TYPE i VALUE 100.
CONSTANTS c_true      TYPE xfeld VALUE 'X'.
```

**The essential ABAP built-in types:**

| ABAP type | Meaning | C# equivalent | Python equivalent |
|-----------|---------|---------------|-------------------|
| `i` | Integer (4 bytes) | `int` | `int` |
| `int8` | 8-byte integer (7.4+) | `long` | `int` |
| `p` | Packed decimal (BCD) | `decimal` | `Decimal` |
| `f` | Floating point (8 bytes) | `double` | `float` |
| `c` | Fixed-length character | `char[]` / `string` | `str` |
| `string` | Variable-length string | `string` | `str` |
| `n` | Numeric text (digits only) | custom `struct` | `str` (digits) |
| `d` | Date (YYYYMMDD) | `DateOnly` | `datetime.date` |
| `t` | Time (HHMMSS) | `TimeOnly` | `datetime.time` |
| `x` | Hex/raw bytes | `byte[]` | `bytes` |
| `xstring` | Variable-length bytes | `byte[]` | `bytes` |

> ⚠️ **C#/Python gotcha:** ABAP `d` (date) is stored as `YYYYMMDD` — an 8-character numeric text, *not* a `DateTime` object. `sy-datum` (today's date) is of type `d`. Arithmetic on dates works: `lv_date = sy-datum + 30.` adds 30 days. But passing dates to external systems always requires explicit formatting.

**Type declarations with TYPES:**

```abap
" TYPES defines a local type alias — like 'type' in C# or TypeAlias in Python
TYPES ty_amount TYPE p LENGTH 10 DECIMALS 2.
TYPES ty_name   TYPE c LENGTH 40.

DATA lv_price   TYPE ty_amount.
DATA lv_company TYPE ty_name.
```

**Referencing DDIC types directly (the preferred approach):**

```abap
" Instead of guessing the type, reference the DDIC:
DATA lv_customer TYPE kunnr.    " KUNNR data element → CHAR 10
DATA lv_country  TYPE land1.    " LAND1 → CHAR 3
DATA lv_netval   TYPE netwr.    " NETWR → CURR 15,2

" Or reference a table field directly:
DATA lv_cust_name TYPE kna1-name1.  " same type as KNA1-NAME1
```

This last pattern is extremely common and powerful. Your variable automatically gets the right length, type, and DDIC metadata — no guessing, no maintenance if SAP changes the field.

---

## 6.2 Control flow — IF, CASE, DO, WHILE, LOOP

### 1️⃣ The analogy

Control flow in ABAP reads like English sentences more than C/C#/Python. There are no curly braces. Blocks end with `END<keyword>`. Once you see the pattern, it's consistent everywhere.

### 2️⃣ You already know this

```csharp
// C# control flow
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

### 3️⃣ The ABAP way

```abap
" IF / ELSEIF / ELSE / ENDIF
IF lv_amount > 0.
  CALL METHOD process_positive.
ELSEIF lv_amount = 0.
  CALL METHOD process_zero.
ELSE.
  CALL METHOD process_negative.
ENDIF.

" Logical operators: AND  OR  NOT  (not && || !)
IF lv_country = 'DE' AND lv_currency = 'EUR'.
  ...
ENDIF.

" String comparison: = is equality (case-sensitive for strings)
IF lv_name CS 'GmbH'.     " CS = contains string (like Contains())
  ...
ENDIF.
```

**Comparison operators:**

| ABAP | Meaning | C# equivalent |
|------|---------|---------------|
| `=` or `EQ` | equal | `==` |
| `<>` or `NE` | not equal | `!=` |
| `<` or `LT` | less than | `<` |
| `>` or `GT` | greater than | `>` |
| `<=` or `LE` | less or equal | `<=` |
| `>=` or `GE` | greater or equal | `>=` |
| `CO` | contains only | — |
| `CS` | contains string | `.Contains()` |
| `CP` | conforms to pattern (wildcards `*` and `+`) | `.Matches()` (regex-ish) |

```abap
" CASE / WHEN / ENDCASE — like switch
CASE lv_status.
  WHEN 'A'.
    process_active( ).
  WHEN 'I'.
    process_inactive( ).
  WHEN OTHERS.           " OTHERS = default
    process_unknown( ).
ENDCASE.

" DO loop — fixed iterations (like for i=1 to n)
DO 10 TIMES.
  lv_counter = lv_counter + 1.
  IF lv_counter = 5.
    CONTINUE.            " like C# continue
  ENDIF.
  IF lv_counter > 8.
    EXIT.                " like C# break
  ENDIF.
ENDDO.

" WHILE loop
WHILE lv_index <= lv_max.
  process_item( lv_index ).
  lv_index = lv_index + 1.
ENDWHILE.
```

> 💡 There is no `for(int i; ...)` in classic ABAP. Index loops use `DO n TIMES.` or `LOOP AT itab ... FROM x TO y.` (covered below). For clarity, `sy-index` holds the current iteration count inside a `DO` loop automatically.

---

## 6.3 Structures & work areas — the DTO equivalent

### 1️⃣ The analogy

A **structure** is a named collection of fields — SAP's equivalent of a C# record/DTO or a Python dataclass. A **work area** is just a variable that holds one instance of a structure — it's the in-memory "current row" you read table data into.

### 2️⃣ You already know this

```csharp
// C# — record type (DTO)
public record Customer(string Id, string Name, string Country, decimal Balance);

// Usage: single instance
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

### 3️⃣ The ABAP way

```abap
" Option 1: Define a local structure type inline
TYPES: BEGIN OF ty_customer,
         id      TYPE kunnr,       " CHAR 10
         name    TYPE name1_gp,    " CHAR 40
         country TYPE land1,       " CHAR 3
         balance TYPE wrbtr,       " CURR 13,2
       END OF ty_customer.

" Declare a work area (a single-row variable of that type)
DATA ls_customer TYPE ty_customer.   " 'ls_' = local structure convention

" Assign fields — uses '-' not '.'
ls_customer-id      = 'CUST01'.
ls_customer-name    = 'Acme GmbH'.
ls_customer-country = 'DE'.
ls_customer-balance = 5000.

" Read fields
WRITE: / ls_customer-id, ls_customer-name.

" Option 2: Type a work area directly off a DDIC table (very common)
DATA ls_kna1 TYPE kna1.   " work area with exact same shape as KNA1 table

" After a SELECT SINGLE:
SELECT SINGLE * FROM kna1 INTO @ls_kna1
  WHERE kunnr = 'CUST01'.

IF sy-subrc = 0.
  WRITE: / ls_kna1-name1, ls_kna1-land1.
ENDIF.
```

**ABAP naming conventions** (you'll see these in every codebase):

| Prefix | Meaning | Example |
|--------|---------|---------|
| `lv_` | local variable (scalar) | `lv_count`, `lv_name` |
| `ls_` | local structure (work area) | `ls_customer`, `ls_order` |
| `lt_` | local table (internal table) | `lt_customers`, `lt_orders` |
| `lc_` | local constant | `lc_max`, `lc_true` |
| `lo_` | local object reference | `lo_handler`, `lo_report` |
| `gv_`, `gs_`, `gt_` | global equivalents | (class/program globals) |
| `iv_`, `is_`, `it_` | import parameter | (method parameters) |
| `ev_`, `es_`, `et_` | export parameter | (method parameters) |
| `rv_`, `rs_`, `rt_` | return parameter | (method return value) |

These aren't enforced by the language — they're a strong community convention. Following them makes your code immediately readable to any experienced ABAPer.

---

## 6.4 Internal tables — the List\<T\> / DataFrame of ABAP

### 1️⃣ The analogy

An **internal table** is an in-memory list of rows that all share the same structure. Think `List<T>` in C# or a pandas DataFrame in Python. It's the single most-used data structure in all of ABAP.

You read database rows into an internal table, process them in a `LOOP`, modify rows, and optionally write back.

### 2️⃣ You already know this

```csharp
public record Order(string Id, string CustomerId, decimal Amount);

// Declare
var orders = new List<Order>();

// Add rows
orders.Add(new Order("ORD1", "CUST01", 100m));
orders.Add(new Order("ORD2", "CUST01", 200m));

// Loop
foreach (var o in orders)
    Console.WriteLine($"{o.Id}: {o.Amount}");

// Find one
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

### 3️⃣ The ABAP way — the three table types

ABAP has three flavors of internal table. Choose the right one for significant performance impact:

| Type | ABAP keyword | .NET/Python analogy | Use when |
|------|-------------|---------------------|----------|
| Standard | `STANDARD TABLE` | `List<T>` | General use; append-heavy; read by index |
| Sorted | `SORTED TABLE` | `SortedList<K,V>` | Always sorted; fast binary-search reads |
| Hashed | `HASHED TABLE` | `Dictionary<K,V>` | Key-only access; O(1) lookup |

```abap
" Declare an internal table (three ways — choose based on access pattern)

" 1. Standard table (append & loop — most common)
DATA lt_orders TYPE STANDARD TABLE OF ty_order WITH EMPTY KEY.

" 2. Sorted table (keep sorted, fast WHERE-key reads)
DATA lt_orders_s TYPE SORTED TABLE OF ty_order
                 WITH UNIQUE KEY order_id.

" 3. Hashed table (pure key lookup, like a dictionary)
DATA lt_orders_h TYPE HASHED TABLE OF ty_order
                 WITH UNIQUE KEY order_id.
```

**Adding rows:**

```abap
" APPEND adds a row to a standard table (like List.Add)
DATA ls_order TYPE ty_order.
ls_order-order_id    = 'ORD1'.
ls_order-customer_id = 'CUST01'.
ls_order-amount      = 100.
APPEND ls_order TO lt_orders.

" Shortcut (inline struct literal — covered properly in Ch 7)
APPEND VALUE #( order_id = 'ORD2'  customer_id = 'CUST01'  amount = 200 )
  TO lt_orders.

" INSERT adds at a specific position (standard table)
INSERT ls_order INTO lt_orders INDEX 1.  " prepend
```

**Looping:**

```abap
" LOOP AT ... INTO work_area — the main iteration construct
LOOP AT lt_orders INTO DATA(ls_order).
  WRITE: / ls_order-order_id, ls_order-amount.
ENDLOOP.

" LOOP AT ... INTO ... WHERE — filtered loop (no extra IF needed)
LOOP AT lt_orders INTO DATA(ls_order)
  WHERE customer_id = 'CUST01'.
  process_order( ls_order ).
ENDLOOP.

" LOOP AT ... ASSIGNING FIELD-SYMBOL — modify in place (like ref in C#)
LOOP AT lt_orders ASSIGNING FIELD-SYMBOL(<order>).
  <order>-amount = <order>-amount * 1.1.   " 10% markup, modifies the table
ENDLOOP.
```

> ⚠️ **C#/Python gotcha:** `LOOP AT ... INTO ls_row` makes a *copy* of the row. Modifying `ls_row` inside the loop does NOT change the table. To modify the table in-place, use `ASSIGNING FIELD-SYMBOL(<row>)` — the field symbol is like a C# `ref`. This is the most common ABAP bug for newcomers.

**Reading a single row:**

```abap
" READ TABLE — the single-row lookup
READ TABLE lt_orders INTO DATA(ls_found)
  WITH KEY order_id = 'ORD1'.

IF sy-subrc = 0.              " 0 = found, 4 = not found
  WRITE: / ls_found-amount.
ELSE.
  WRITE: / 'Order not found'.
ENDIF.

" For sorted/hashed tables with a defined key, use WITH TABLE KEY for speed:
READ TABLE lt_orders_h INTO DATA(ls_h)
  WITH TABLE KEY order_id = 'ORD1'.
```

**Other useful table operations:**

```abap
" SORT — sorts a standard table
SORT lt_orders BY customer_id ASCENDING amount DESCENDING.

" DELETE — remove rows
DELETE lt_orders WHERE amount < 10.
DELETE TABLE lt_orders_h WITH TABLE KEY order_id = 'ORD1'.

" DESCRIBE TABLE — like .Count
DESCRIBE TABLE lt_orders LINES DATA(lv_count).
" Modern equivalent: lv_count = lines( lt_orders ).

" CLEAR — empty a table
CLEAR lt_orders.   " removes all rows (same as lt_orders = VALUE #( ). )
```

---

## 6.5 Open SQL — SELECT into table (with sy-subrc)

### 1️⃣ The analogy

ABAP's **Open SQL** is an abstraction layer over the database. You write SQL-like syntax; ABAP translates it for the underlying DB (HANA, Oracle, DB2). The key differences from raw SQL: no `FROM dual`, no multi-table joins in older syntax (ABAP handles that in code), and the MANDT filter is automatic.

### 2️⃣ You already know this

```csharp
// C# — EF Core / LINQ
using var db = new AppDbContext();

// Multiple rows
var orders = db.Orders
    .Where(o => o.Country == "DE")
    .OrderBy(o => o.CreatedAt)
    .ToList();

// Single row
var order = db.Orders.SingleOrDefault(o => o.OrderId == "ORD1");
if (order == null) { /* handle not found */ }

// Aggregation
var total = db.Orders.Sum(o => o.Amount);
```

```python
# Python — SQLAlchemy
from sqlalchemy import select, func

# Multiple rows
stmt = select(Order).where(Order.country == "DE").order_by(Order.created_at)
orders = session.execute(stmt).scalars().all()

# Single row
order = session.get(Order, "ORD1")  # or .first() with filter
if order is None:
    pass  # handle not found
```

### 3️⃣ The ABAP way

```abap
" SELECT multiple rows INTO TABLE
SELECT order_id, customer_id, amount, currency
  FROM zorders
  INTO TABLE @DATA(lt_orders)
  WHERE currency = 'EUR'
  ORDER BY order_id.

IF sy-subrc = 0.
  " Rows found — process lt_orders
ELSE.
  " No rows matched — sy-subrc = 4
ENDIF.

" SELECT SINGLE — one row (use for primary key lookups)
SELECT SINGLE *
  FROM kna1
  INTO @DATA(ls_customer)
  WHERE kunnr = @lv_customer_id.

IF sy-subrc <> 0.
  " Customer not found
  MESSAGE 'Customer not found' TYPE 'E'.
ENDIF.

" SELECT with JOIN (ABAP 7.4+)
SELECT o~order_id, o~amount, c~name1 AS customer_name
  FROM zorders AS o
  INNER JOIN kna1 AS c ON o~kunnr = c~kunnr
  INTO TABLE @DATA(lt_result)
  WHERE o~currency = 'EUR'.

" Aggregation
SELECT currency, SUM( amount ) AS total_amount
  FROM zorders
  INTO TABLE @DATA(lt_totals)
  GROUP BY currency.

" The @ prefix on host variables is ABAP 7.4+ syntax (required in strict mode)
DATA lv_min_amount TYPE zorders-amount VALUE 1000.
SELECT * FROM zorders INTO TABLE @DATA(lt_big_orders)
  WHERE amount >= @lv_min_amount.
```

> ⚠️ **C#/Python gotcha:** `SELECT SINGLE` without `ORDER BY` is non-deterministic if multiple rows match. Use it only for primary-key or unique-key lookups. For "give me the first row of many," use `SELECT ... UP TO 1 ROWS` with `ORDER BY`.

> ⚠️ **C#/Python gotcha:** **Always check `sy-subrc` after SELECT.** A `SELECT INTO TABLE` that returns zero rows sets `sy-subrc = 4` but does NOT throw an exception. If you proceed without checking, you'll process an empty table silently. A `SELECT SINGLE` that finds nothing also sets `sy-subrc = 4`. This is the sysubrc drum — we're beating it again.

**Common `sy-subrc` values:**

| sy-subrc | Meaning |
|----------|---------|
| `0` | Success / found |
| `4` | Not found / no match |
| `8` | General error (depends on statement) |
| `12` | More specific error |

**UPDATE, INSERT, DELETE via Open SQL:**

```abap
" INSERT a row
INSERT zorders FROM @ls_order.           " sy-subrc = 0 ok, 4 = duplicate key

" UPDATE a row
UPDATE zorders SET amount = @lv_new_amount
  WHERE order_id = @lv_id.             " sy-subrc = 0 ok, 4 = not found

" DELETE rows
DELETE FROM zorders WHERE currency = 'XXX'.

" MODIFY — INSERT or UPDATE (upsert)
MODIFY zorders FROM @ls_order.
```

> 💡 **Performance rule #1:** Never do a SELECT inside a LOOP (the "N+1 query" problem). Fetch all data you need upfront into internal tables, then process in memory. SAP systems handle millions of records; a database round-trip per row is catastrophic.

```abap
" WRONG — N+1 problem
LOOP AT lt_order_ids INTO DATA(lv_id).
  SELECT SINGLE * FROM kna1 INTO @DATA(ls_cust)
    WHERE kunnr = @lv_id.             " DB call per row!
ENDLOOP.

" RIGHT — single SELECT with FOR ALL ENTRIES
IF lt_order_ids IS NOT INITIAL.       " mandatory check before FOR ALL ENTRIES
  SELECT *
    FROM kna1
    INTO TABLE @DATA(lt_customers)
    FOR ALL ENTRIES IN @lt_order_ids
    WHERE kunnr = @lt_order_ids-kunnr.
ENDIF.
```

> ⚠️ **FOR ALL ENTRIES gotcha:** If `lt_order_ids` is empty, `FOR ALL ENTRIES` returns *all rows* from the table — equivalent to no WHERE clause. Always guard it with `IF table IS NOT INITIAL`.

---

## 6.6 Subroutines, methods, MESSAGE, and sy-subrc error handling

### 1️⃣ The analogy

ABAP code can be organized as **FORM subroutines** (legacy, like old-school C functions) or **methods** (modern, object-oriented). New code should always use methods. You'll encounter FORMs in legacy code and need to read them, but write methods.

### 2️⃣ You already know this

```csharp
// C# method with error handling via exceptions
public decimal CalculateDiscount(string customerId, decimal amount)
{
    if (string.IsNullOrEmpty(customerId))
        throw new ArgumentException("Customer ID required");

    var rate = GetDiscountRate(customerId);  // may throw
    return amount * rate;
}

// Caller
try
{
    var discounted = CalculateDiscount("CUST01", 1000m);
}
catch (ArgumentException ex)
{
    Console.Error.WriteLine(ex.Message);
}
```

### 3️⃣ The ABAP way

**FORM subroutines (legacy — read-only, don't write new ones):**

```abap
" Legacy FORM — you'll see this in old programs
FORM calculate_discount
  USING    iv_customer TYPE kunnr
           iv_amount   TYPE wrbtr
  CHANGING cv_discount TYPE wrbtr.

  DATA lv_rate TYPE p DECIMALS 2.
  " ... logic ...
  cv_discount = iv_amount * lv_rate.
ENDFORM.

" Called with:
PERFORM calculate_discount
  USING    lv_customer_id
           lv_amount
  CHANGING lv_discount.
```

**Methods (modern — always prefer these):**

```abap
" In a class (covered fully in Chapter 11):
METHOD calculate_discount.
  " Parameters defined in class definition section:
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

" Called with:
TRY.
  DATA(lv_discount) = lo_pricer->calculate_discount(
    iv_customer = lv_customer_id
    iv_amount   = lv_amount ).
CATCH zcx_discount_error INTO DATA(lx_error).
  MESSAGE lx_error->get_text( ) TYPE 'E'.
ENDTRY.
```

**MESSAGE statements — SAP's user notification system:**

```abap
" MESSAGE is the SAP way to show feedback — in reports, dialogs, etc.
" Syntax: MESSAGE '<text>' TYPE '<type>'.
" Or:     MESSAGE ID '<msgclass>' TYPE '<type>' NUMBER '<num>' WITH '<var>'.

" Types:
" 'S' = Success (green)   → shown in status bar, continues
" 'I' = Information       → popup, user must click OK
" 'W' = Warning           → popup in dialogs, continues
" 'E' = Error             → re-shows screen with error, stops flow
" 'A' = Abort             → cancels transaction immediately
" 'X' = Termination       → program dump (use only for fatal dev errors)

MESSAGE 'Order saved successfully.' TYPE 'S'.
MESSAGE 'Customer not found.' TYPE 'E'.
MESSAGE 'Processing complete: & orders imported.' TYPE 'I'
  WITH lv_count.  " & is replaced by the WITH parameter
```

**sy-subrc — the return code pattern:**

```abap
" The fundamental ABAP error-handling pattern:
" Most statements set sy-subrc. You MUST check it.

" READ TABLE
READ TABLE lt_orders INTO DATA(ls_order)
  WITH KEY order_id = lv_id.
IF sy-subrc <> 0.
  MESSAGE 'Order not found.' TYPE 'E'.
  RETURN.   " like C# return; — exits current method/form
ENDIF.

" OPEN SQL
SELECT SINGLE * FROM kna1 INTO @DATA(ls_customer)
  WHERE kunnr = @lv_id.
IF sy-subrc <> 0.
  " handle not found
ENDIF.

" Function module call (covered in Chapter 12)
CALL FUNCTION 'BAPI_CUSTOMER_GETDETAIL'
  EXPORTING
    customerno = lv_id
  IMPORTING
    customeraddress = ls_address
  EXCEPTIONS
    customer_not_found = 1
    OTHERS             = 2.

CASE sy-subrc.
  WHEN 0.  " success
  WHEN 1.  MESSAGE 'Customer not found' TYPE 'E'.
  WHEN 2.  MESSAGE 'Unexpected error' TYPE 'E'.
ENDCASE.
```

> ⚠️ **C#/Python gotcha:** ABAP *does* have `TRY...CATCH` (for class-based exceptions, ABAP 6.10+), and modern code uses it for object-oriented error handling. But the vast majority of older ABAP — including most standard SAP function modules and BAPIs — uses `sy-subrc` or the `EXCEPTIONS` block on `CALL FUNCTION`. You have to know both. Every senior ABAP developer will immediately ask "did you check sy-subrc?" when debugging your code.

**Other important system variables (`sy-*`):**

| Variable | Meaning | C# parallel |
|----------|---------|-------------|
| `sy-subrc` | Return code of last statement | (exception presence) |
| `sy-datum` | Today's date (type d) | `DateOnly.Today` |
| `sy-uzeit` | Current time (type t) | `TimeOnly.Now` |
| `sy-uname` | Current user's login name | `Environment.UserName` |
| `sy-mandt` | Current client | (tenant ID) |
| `sy-langu` | Logon language | `CultureInfo.CurrentCulture` |
| `sy-tabix` | Current loop index (inside LOOP AT) | loop variable `i` |
| `sy-dbcnt` | Rows affected by last DB statement | `ExecuteNonQuery()` return |
| `sy-msgv1..4` | Message variables from last MESSAGE | (message args) |

---

## 6.7 Full example — putting it all together

Here's a realistic mini-program that demonstrates everything in this chapter: reads orders for a customer, calculates totals, and outputs results.

```abap
REPORT z_order_summary.

" --- Types & data ---
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

" --- Read customer master ---
SELECT SINGLE *
  FROM kna1
  INTO @ls_customer
  WHERE kunnr = @lv_customer.

IF sy-subrc <> 0.
  MESSAGE 'Customer not found.' TYPE 'E'.
  RETURN.
ENDIF.

WRITE: / 'Customer:', ls_customer-name1.

" --- Read orders ---
SELECT *
  FROM zorders
  INTO TABLE @lt_orders
  WHERE kunnr = @lv_customer
  ORDER BY waers.

IF sy-subrc <> 0.
  WRITE: / 'No orders found for this customer.'.
  RETURN.
ENDIF.

" --- Aggregate by currency ---
LOOP AT lt_orders INTO DATA(ls_order).
  READ TABLE lt_summary ASSIGNING FIELD-SYMBOL(<sum>)
    WITH KEY currency = ls_order-waers.

  IF sy-subrc = 0.
    " Currency already in summary — add to it
    <sum>-total = <sum>-total + ls_order-amount.
    <sum>-count = <sum>-count + 1.
  ELSE.
    " New currency — append
    ls_summary-currency = ls_order-waers.
    ls_summary-total    = ls_order-amount.
    ls_summary-count    = 1.
    APPEND ls_summary TO lt_summary.
  ENDIF.
ENDLOOP.

" --- Output ---
WRITE: / '--- Order Summary ---'.
LOOP AT lt_summary INTO DATA(ls_sum_row).
  WRITE: / ls_sum_row-currency, ls_sum_row-total, ls_sum_row-count.
ENDLOOP.
```

This single program uses: `DATA`, `TYPES`, `SELECT SINGLE`, `sy-subrc` checking, `SELECT INTO TABLE`, `LOOP AT ... INTO`, `READ TABLE ... ASSIGNING FIELD-SYMBOL`, `APPEND`, and `WRITE`. These are your daily-driver tools.

---

## 🧠 Recap

- **Declare with DATA / TYPES**; reference DDIC types whenever possible (`TYPE kunnr`, `TYPE kna1-name1`).
- ABAP is 1-indexed. String comparison ignores trailing spaces for `CHAR` types. No `null` — types have zero/initial values by default.
- **`sy-subrc`** is the return code for almost everything. Check it after every SELECT, READ TABLE, CALL FUNCTION, or MODIFY. Not checking it is the #1 ABAP bug.
- `LOOP AT ... INTO` makes a copy (read only). `LOOP AT ... ASSIGNING FIELD-SYMBOL(<row>)` gives a reference for in-place modification.
- Use **`FOR ALL ENTRIES IN`** instead of SELECT inside LOOP. Always guard it with `IS NOT INITIAL`.
- `MESSAGE` with type `'S'`/`'I'`/`'W'`/`'E'`/`'A'` controls the user feedback pattern throughout classic ABAP.
- FORMs are legacy; **methods** (CLASS/ENDCLASS) are the modern approach.
- `TRY/CATCH` exists for class-based exceptions; `sy-subrc` + `EXCEPTIONS` block exists for function modules. Know both.

---

*[← Contents](../content.md) | [← Previous: The Data Dictionary (DDIC)](05-data-dictionary-ddic.md) | [Next: Modern ABAP Syntax (7.4+) →](07-modern-abap-syntax.md)*
