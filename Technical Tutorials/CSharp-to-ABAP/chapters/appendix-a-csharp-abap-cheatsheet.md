# Appendix A: C# / Python ↔ ABAP Cheat Sheet

*The running translation dictionary — keep this open while you work.*

---

## 📌 How to use this appendix

Every table here is a direct mapping: left column is what you already write in C# or Python; right column is the SAP/ABAP equivalent. Where there is no clean equivalent, we say so honestly rather than inventing a false analogy.

---

## 1 — Concepts: the ecosystem map

| What you know (C# / Python) | What SAP calls it | SAP tool / transaction |
|---|---|---|
| Class library / NuGet package | **Package + Transport Request** | SE80, ADT (Eclipse) |
| Solution / project folder | **Package** (namespace container) | SE80, DEVC object |
| NuGet package manager | No direct equivalent — standard SAP objects arrive via **Support Packages / Notes** | SNOTE, transaction SPAM |
| Entity Framework model / SQLAlchemy | **DDIC** (Data Dictionary) — tables, structures, data elements, domains | SE11 |
| SQL Server / PostgreSQL | **SAP HANA** (and you write **Open SQL**, not HANA SQL, for portability) | SE11, HANA Studio / HDBSQL |
| ASP.NET Web API controller | **OData service** (classic: SEGW; modern: RAP CDS-based service) | SEGW, ADT |
| DTO / record / dataclass | **Structure + Work Area** | SE11 (structure type) |
| `List<T>` / pandas DataFrame | **Internal Table** | ABAP language construct |
| `IEnumerable<T>` / generator | **LOOP AT** (no lazy streaming equivalent; ABAP reads full result sets) | — |
| `interface` + dependency injection | **Interface** (`INTERFACE … ENDINTERFACE`) + **BAdI** for framework-level DI | SE18, SE19, SE24 |
| Abstract class / virtual method | ABAP abstract class / method (identical concept, different keyword order) | SE24, ADT |
| `try / catch / finally` | `TRY … CATCH cx_root … CLEANUP … ENDTRY` | — |
| Custom exception class | Class inheriting `cx_static_check` or `cx_dynamic_check` or `cx_no_check` | SE24 |
| `async / await` | ⚠️ **No direct equivalent** in classic ABAP. Background processing uses **background jobs** (SM36/SM37). Modern ABAP Cloud has `CL_ABAP_PARALLEL` for limited parallelism — but think "scheduled job", not async/await. | SM36, SM37 |
| `Task<T>` / `Promise<T>` | ⚠️ **No equivalent.** SAP is synchronous-first at the dialog layer. | — |
| `HttpClient` / `requests` | `CL_HTTP_CLIENT` (classic) or `CL_WEB_HTTP_CLIENT_MANAGER` (modern) | — |
| REST API endpoint | **OData service** (SEGW/RAP) or **RFC-enabled Function Module** | SEGW, SE37 |
| XML/JSON serialization | `CL_SXML_*`, `CL_ABAP_CONV_OUT_CE`, `/UI2/CL_JSON` — manual or class-based | — |
| Unit test (`xUnit`, `pytest`) | **ABAP Unit** (`CLASS … FOR TESTING`) built into ADT | ADT test runner |
| Logging / Serilog | **Application Log** (`BAL_*` function modules) or `CL_DEMO_OUTPUT` in dev | SLG1 |
| Regex | `CL_ABAP_REGEX` + `CL_ABAP_MATCHER` | — |
| Git / version control | **abapGit** (open source, maps Git ↔ ABAP objects) + **Transport Requests** (system-to-system promotion) | SE01, SE09, STMS |

---

## 2 — Syntax side-by-side

Three columns: **C#** | **Python** | **ABAP**. Read across a row for the translation.

### 2.1 Variable declaration

```csharp
// C# — strongly typed / var
int count = 0;
string name = "Acme";
var customer = new Customer();
```

```python
# Python — dynamic
count = 0
name = "Acme"
customer = Customer()
```

```abap
" ABAP — DATA statement (inline DATA() available since 7.40)
DATA lv_count TYPE i.
DATA lv_name  TYPE string.
DATA lo_cust  TYPE REF TO zcl_customer.

" Inline declaration (modern, preferred)
DATA(lv_total) = 0.
```

> ⚠️ **Gotcha:** ABAP prefixes are a convention, not enforced by the compiler: `lv_` = local variable, `lt_` = local table, `ls_` = local structure, `lo_` = local object reference, `gv_` / `gt_` / `gs_` = global. You'll see them everywhere.

---

### 2.2 String interpolation / formatting

```csharp
// C#
string msg = $"Customer {id} has {count} orders.";
```

```python
# Python
msg = f"Customer {id} has {count} orders."
```

```abap
" ABAP — string templates (|…| syntax, ABAP 7.40+)
DATA(lv_msg) = |Customer { lv_id } has { lv_count } orders.|.

" Classic concatenation (older code — you'll see this in legacy systems)
CONCATENATE 'Customer' lv_id 'has' lv_count 'orders.'
  INTO lv_msg SEPARATED BY space.
```

---

### 2.3 Collections / internal table initialisation

```csharp
// C#
var orders = new List<Order>
{
    new Order { Id = 1, Amount = 100m },
    new Order { Id = 2, Amount = 200m }
};
```

```python
# Python
orders = [
    {"id": 1, "amount": 100},
    {"id": 2, "amount": 200},
]
```

```abap
" ABAP — VALUE constructor (7.40+)
TYPES: BEGIN OF ty_order,
         id     TYPE i,
         amount TYPE p DECIMALS 2,
       END OF ty_order.
DATA lt_orders TYPE TABLE OF ty_order WITH EMPTY KEY.

lt_orders = VALUE #(
  ( id = 1  amount = '100.00' )
  ( id = 2  amount = '200.00' )
).
```

---

### 2.4 Loop + filter

```csharp
// C#
foreach (var o in orders.Where(o => o.Amount > 100))
    Console.WriteLine(o.Id);
```

```python
# Python
for o in [o for o in orders if o["amount"] > 100]:
    print(o["id"])
```

```abap
" ABAP — LOOP AT … WHERE (7.40+)
LOOP AT lt_orders INTO DATA(ls_order) WHERE amount > 100.
  WRITE: / ls_order-id.
ENDLOOP.

" Field-symbols variant (faster for large tables — modifies in place)
LOOP AT lt_orders ASSIGNING FIELD-SYMBOL(<s_order>) WHERE amount > 100.
  WRITE: / <s_order>-id.
ENDLOOP.
```

---

### 2.5 Null / initial value check

```csharp
// C#
if (name == null || name == string.Empty) { ... }
if (obj == null) { ... }
```

```python
# Python
if not name: ...
if obj is None: ...
```

```abap
" ABAP — IS INITIAL checks for the "zero value" of any type
IF lv_name IS INITIAL.         " empty string, or zero, or initial date
  ...
ENDIF.
IF lo_object IS INITIAL.       " null reference
  ...
ENDIF.
" Modern: IS NOT BOUND for object references
IF lo_object IS NOT BOUND.
  ...
ENDIF.
```

> 💡 ABAP variables are **always initialised** to their type's zero value (`0`, `''`, `'00000000'` for date, etc.) — there is no uninitialised / undefined state.

---

### 2.6 Ternary / conditional expression

```csharp
// C#
string label = active ? "Active" : "Inactive";
```

```python
# Python
label = "Active" if active else "Inactive"
```

```abap
" ABAP — COND (7.40+)
DATA(lv_label) = COND #( WHEN lv_active = abap_true
                          THEN 'Active'
                          ELSE 'Inactive' ).

" Classic IF/ELSE still common in older code
IF lv_active = abap_true.
  lv_label = 'Active'.
ELSE.
  lv_label = 'Inactive'.
ENDIF.
```

---

### 2.7 Switch / pattern matching

```csharp
// C#
string desc = status switch
{
    "A" => "Active",
    "I" => "Inactive",
    _   => "Unknown"
};
```

```python
# Python (3.10+)
match status:
    case "A": desc = "Active"
    case "I": desc = "Inactive"
    case _:   desc = "Unknown"
```

```abap
" ABAP — CASE
CASE lv_status.
  WHEN 'A'. lv_desc = 'Active'.
  WHEN 'I'. lv_desc = 'Inactive'.
  WHEN OTHERS. lv_desc = 'Unknown'.
ENDCASE.

" SWITCH expression (7.40+, inline)
DATA(lv_desc2) = SWITCH #( lv_status
  WHEN 'A' THEN 'Active'
  WHEN 'I' THEN 'Inactive'
  ELSE          'Unknown' ).
```

---

### 2.8 Exception handling

```csharp
// C#
try
{
    var result = Divide(a, b);
}
catch (DivideByZeroException ex)
{
    Console.WriteLine(ex.Message);
}
finally
{
    Cleanup();
}
```

```python
# Python
try:
    result = divide(a, b)
except ZeroDivisionError as ex:
    print(ex)
finally:
    cleanup()
```

```abap
" ABAP
TRY.
    DATA(lv_result) = divide( a = lv_a  b = lv_b ).
  CATCH cx_sy_zerodivide INTO DATA(lx_ex).
    WRITE: / lx_ex->get_text( ).
  CLEANUP.
    " runs when an unhandled exception propagates out of this TRY block
    cleanup( ).
ENDTRY.
```

> 📌 `cx_root` is the superclass of every ABAP exception — catching it is the ABAP equivalent of catching `Exception` in C#. Prefer catching the specific class.

---

### 2.9 Reading a single row from a DB / table

```csharp
// C# (EF Core)
var customer = dbContext.Customers.FirstOrDefault(c => c.Id == id);
if (customer == null) throw new NotFoundException();
```

```python
# Python (SQLAlchemy)
customer = session.query(Customer).filter_by(id=id).first()
if customer is None:
    raise NotFoundException()
```

```abap
" ABAP — SELECT SINGLE
SELECT SINGLE *
  FROM zcustomer
  WHERE id = @lv_id
  INTO @DATA(ls_customer).

IF sy-subrc <> 0.
  RAISE EXCEPTION TYPE zcx_not_found.
ENDIF.
```

> ⚠️ **`sy-subrc` is the ABAP return code.** `0` = success. Every database statement, READ TABLE, CALL FUNCTION, etc. sets it. Check it — always. It is the ABAP equivalent of checking for `null` after every call.

---

## 3 — Data type mapping

| C# type | Python type | ABAP type | Notes |
|---|---|---|---|
| `int` | `int` | `i` | 32-bit integer |
| `long` | `int` | `int8` | 64-bit integer (7.40+) |
| `decimal` | `Decimal` | `p LENGTH n DECIMALS d` | Packed decimal — **use for money** |
| `double` / `float` | `float` | `f` | Floating point — avoid for financials |
| `string` | `str` | `string` | Dynamic length (heap) |
| `char` | `str` (len 1) | `c LENGTH n` | Fixed-length char, right-padded with spaces |
| `bool` | `bool` | `abap_bool` (`c LENGTH 1`) | `abap_true` = `'X'`, `abap_false` = `' '` — there is **no native bool** |
| `DateTime` | `datetime` | `d` (date, YYYYMMDD) + `t` (time, HHMMSS) | Date and time are **separate types** |
| `DateTimeOffset` | — | `utclong` (ABAP 7.53+) | UTC timestamp with microseconds |
| `TimeSpan` | `timedelta` | No direct type — compute in seconds (`i`) | — |
| `byte[]` | `bytes` | `xstring` | Raw binary / hex string |
| `Guid` | `uuid.UUID` | `sysuuid_x16` / `sysuuid_c32` | `cl_system_uuid=>create_uuid_c32_static( )` |
| `enum` | `Enum` | **Domain** with fixed values (DDIC) or `CONSTANTS` block | No language-level enum keyword |
| `null` | `None` | IS INITIAL / IS NOT BOUND | ABAP variables are never truly null; they have initial values |
| `object` | `object` | `TYPE REF TO object` | Root reference type |

---

## 4 — Mental model gotchas

These are the things that trip up every C#/Python developer in their first weeks. Read them now; re-read them when something weird happens.

| # | Gotcha | What you expect | What ABAP does |
|---|---|---|---|
| 1 | **1-indexed tables** | `list[0]` is the first element | `READ TABLE itab INDEX 1` is the first row. There is no index 0. |
| 2 | **`sy-subrc` not exceptions** | Bad DB call raises an exception | Most ABAP statements set `sy-subrc` silently. You must check it manually. |
| 3 | **Case-insensitive** | `customerName ≠ CustomerName` | ABAP keywords and identifiers are fully case-insensitive. `DATA` = `data` = `Data`. |
| 4 | **`=` is both assign and compare** | `==` for equality, `=` for assignment | In ABAP, `IF a = b` is a comparison. `a = b.` (outside IF) is assignment. Context decides. |
| 5 | **Value semantics for structures** | Objects are references | ABAP **structures** are value types — assigning one to another copies all fields. Like C# `struct`. |
| 6 | **Client / MANDT** | One DB, one schema | SAP runs **multiple clients** (logical tenants) in the same physical database. Every client-dependent table has a `MANDT` key field. Open SQL adds the current client automatically — you rarely write it explicitly, but you must know it exists. |
| 7 | **`COMMIT WORK`** | ORM / transaction auto-commits or rolls back | In ABAP, database changes are **not committed** until you explicitly call `COMMIT WORK`. A background job crash will roll back uncommitted changes. Forgetting `COMMIT WORK` is a common bug. |
| 8 | **No garbage collector worries** | GC manages memory | ABAP has reference-counted memory management. Circular references can leak, but in practice this rarely causes problems. Internal tables are cleaned up when they go out of scope. |
| 9 | **Periods end statements** | `;` ends statements | In ABAP, every statement ends with a `.` (period). A missing period is a syntax error. |
| 10 | **Chaining with `,`** | Multiple statements on one line | ABAP has the **chain syntax**: `WRITE: / lv_a, lv_b, lv_c.` — the `:` starts the chain, `,` continues it. |
| 11 | **WRITE vs. output methods** | `Console.WriteLine` / `print()` | `WRITE: / text.` outputs to the classical list. In OOP code use `cl_demo_output` or message classes. |
| 12 | **`TABLES` parameter in Form routines** | Pass collections as normal parameters | Old-style subroutines (`FORM/ENDFORM`) pass internal tables through a `TABLES` parameter — a quirk you'll see in legacy code. In ABAP OOP, pass them like any other parameter. |
| 13 | **Transport-first development** | Commit to git, done | Every ABAP change is made inside a **Transport Request**. Objects must be transported through DEV → QAS → PRD to go live. Nothing "just appears" in production. |
| 14 | **`TYPE` vs `LIKE`** | One keyword for types | `TYPE` references a DDIC type or ABAP built-in. `LIKE` copies the type of an existing variable. Prefer `TYPE` in new code. |
| 15 | **Fixed-length char padding** | Strings trim/compare naturally | `c` fields are padded with spaces to their fixed length. Comparing `'AB'` with `'AB  '` works, but concatenating them gives you the spaces. Use `string` type for dynamic strings. |

---

## 5 — Quick reference: common ABAP patterns you'll write every day

```abap
" ── SELECT into internal table ────────────────────────────────────────
SELECT id, name, city
  FROM zcustomer
  WHERE region = @lv_region
  INTO TABLE @DATA(lt_customers).

" ── READ TABLE — find one row ─────────────────────────────────────────
READ TABLE lt_customers
  WITH KEY id = lv_id
  INTO DATA(ls_cust).
IF sy-subrc = 0.
  " found
ENDIF.

" ── MODIFY TABLE — update a row in memory ─────────────────────────────
ls_cust-city = 'Munich'.
MODIFY TABLE lt_customers FROM ls_cust.

" ── DELETE TABLE row ──────────────────────────────────────────────────
DELETE TABLE lt_customers WITH TABLE KEY id = lv_id.

" ── Sorted / hashed table for fast lookups ────────────────────────────
DATA lt_hash TYPE HASHED TABLE OF ty_customer
     WITH UNIQUE KEY id.

" ── String operations ─────────────────────────────────────────────────
DATA(lv_upper)  = to_upper( lv_name ).
DATA(lv_length) = strlen( lv_name ).
FIND 'error' IN lv_message IGNORING CASE.  " sy-subrc = 0 if found

" ── Date arithmetic ───────────────────────────────────────────────────
DATA(lv_today)    = sy-datum.             " system date, type d
DATA(lv_tomorrow) = lv_today + 1.        " date arithmetic just works

" ── Object creation ───────────────────────────────────────────────────
DATA(lo_service) = NEW zcl_order_service( ).
DATA(lv_result)  = lo_service->process( iv_id = lv_id ).
```

---

*[← Contents](../content.md) | [← Previous: Chapter 35 — RAP](35-rap-restful-application-programming.md) | [Next: Appendix B — Glossary →](appendix-b-glossary.md)*
