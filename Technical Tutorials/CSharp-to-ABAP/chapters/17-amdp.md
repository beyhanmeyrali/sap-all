# Chapter 17: AMDP (ABAP Managed Database Procedures)

*When a CDS view is not enough — bringing SQLScript loops and temp tables into ABAP's world.*

---

## ☕ The mental model first

CDS views are great for JOINs, filters, and projections — the stuff SQL does naturally. But SQL is declarative: you describe *what* you want, not *how* to compute it. Sometimes what you need is genuinely procedural: iterate over a result set, maintain running totals, build a temp table, branch on conditions, and return a shaped result. That's not a view anymore — that's a stored procedure.

In the classic ABAP world, you'd do this by pulling data into ABAP, looping over it, and computing in memory. On HANA that's inefficient. The database has far more horsepower for this kind of processing than your ABAP application server ever will.

**AMDP — ABAP Managed Database Procedures** is SAP's answer: write a HANA SQLScript procedure, but call and manage it from an ABAP class. The best of both worlds — ABAP's type safety and transport system, HANA's processing power.

> 💡 **Quick reality check:** AMDP is a specialist tool. In a typical shop, 90% of data needs are covered by Open SQL + CDS. You reach for AMDP when a CDS view gets too complex, performance profiling shows that ABAP-side processing is the bottleneck, or you need genuine procedural computation (complex running aggregates, decision-tree-style logic, iterative data transformations).

---

## 17.1 When CDS isn't enough

### 1️⃣ The analogy

Think about the ORM-vs-stored-procedure debate you've had in .NET or Python land:

- **ORM / LINQ:** great for CRUD, relationships, simple filters. Clean, testable, DB-agnostic.
- **Raw SQL view:** fine for a fixed projection you reuse everywhere.
- **Stored procedure:** when you have complex procedural logic — loops, cursors, temp tables, multi-step calculations — and you want it all to happen *inside* the database engine, not round-tripped to the app server.

ABAP's parallel:

| Scenario | Tool |
|----------|------|
| Simple SELECT, JOIN, filter | Open SQL |
| Reusable projection, annotated for Fiori/OData | CDS View Entity |
| Procedural logic, loops, temp tables — must stay in DB | AMDP |
| Logic is business-rule-heavy and DB-performance isn't the issue | ABAP class method |

### Common AMDP trigger scenarios

- **Running balance calculation** — you need to iterate over ordered FI documents and maintain a cumulative balance; a window function *might* do it, but the business rule is complex enough that a procedure loop is cleaner.
- **Multi-step inventory allocation** — allocate stock across orders with priority rules that don't map neatly to a single SQL expression.
- **Complex hierarchical BOM explosion** — bill-of-materials where the recursion depth and pruning rules are non-trivial.
- **Custom analytical aggregation** — something the HANA analytic engine doesn't directly expose.

> ⚠️ **C#/Python gotcha:** You *cannot* mix ABAP statements inside an AMDP method body — the method is pure SQLScript. No `SELECT INTO @lt_result`, no `LOOP AT`, no field symbols. The method body is handed verbatim to HANA. Think of it as an embedded language, like writing a JavaScript template inside a C# file with Razor: two separate worlds in one file.

---

## 17.2 AMDP class anatomy

An AMDP is just a regular ABAP class with a special interface and a special method keyword.

### The three pieces

1. **Implement `IF_AMDP_MARKER_HDB`** — this is a marker interface (no methods to implement). It tells the ABAP runtime "this class contains database procedures."
2. **Declare the method** normally in the class definition.
3. **Implement the method** with the `BY DATABASE PROCEDURE FOR HDB LANGUAGE SQLSCRIPT` addition.

### 2️⃣ You already know this

```csharp
// C# — calling a stored procedure via Dapper or ADO.NET
// The procedure lives in the DB; C# just calls it
public async Task<IEnumerable<SalesAggregate>> GetSalesAggregatesAsync(
    string companyCode, int fiscalYear)
{
    using var conn = new SqlConnection(_connectionString);
    return await conn.QueryAsync<SalesAggregate>(
        "usp_GetSalesAggregates",
        new { CompanyCode = companyCode, FiscalYear = fiscalYear },
        commandType: CommandType.StoredProcedure);
}
```

```python
# Python — calling a stored procedure via psycopg2
def get_sales_aggregates(conn, company_code: str, fiscal_year: int):
    cursor = conn.cursor()
    cursor.callproc("get_sales_aggregates", [company_code, fiscal_year])
    return cursor.fetchall()
```

In both cases: ABAP/Python is the caller; the DB is where the work happens. AMDP is the same idea, except the procedure definition lives *inside* an ABAP class — versioned, transported, and type-safe — not separately in the database.

### 3️⃣ The ABAP way — a complete AMDP class

```abap
"! AMDP class: calculates sales totals per customer for a given period.
"! Runs entirely in HANA SQLScript — not in ABAP memory.
CLASS zcl_sales_aggregator DEFINITION
  PUBLIC FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    " ① The marker interface — no methods, just a flag to the runtime
    INTERFACES if_amdp_marker_hdb.

    " ② Declare the AMDP method like any other ABAP method
    "   Note: parameters are typed with DDIC types
    TYPES:
      BEGIN OF ty_sales_total,
        kunnr   TYPE kunnr,      " customer number
        name1   TYPE name1,      " customer name
        netwr   TYPE netwr_ak,   " net value
        waerk   TYPE waerk,      " currency
      END OF ty_sales_total,
      tt_sales_total TYPE STANDARD TABLE OF ty_sales_total WITH EMPTY KEY.

    CLASS-METHODS get_sales_by_customer
      IMPORTING
        iv_bukrs      TYPE bukrs          " company code
        iv_date_from  TYPE dats           " start date
        iv_date_to    TYPE dats           " end date
      EXPORTING
        et_result     TYPE tt_sales_total
      RAISING
        cx_amdp_error.

ENDCLASS.


CLASS zcl_sales_aggregator IMPLEMENTATION.

  METHOD get_sales_by_customer
    BY DATABASE PROCEDURE              "③ The magic keyword
    FOR HDB                            "   For HANA (HDB = HANA DB)
    LANGUAGE SQLSCRIPT                 "   Language = SQLScript
    OPTIONS READ-ONLY                  "   This proc only reads data
    USING                              "④ Declare all DB objects used
      vbak                             "   Sales order header
      vbap                             "   Sales order items
      kna1.                            "   Customer master

    " ─────────────────────────────────────────────────────────────────
    " Everything from here down is pure SQLScript — NOT ABAP!
    " You have: temp tables (:lt_xxx), variables, loops, IF/ELSEIF,
    " standard SQL functions, and HANA-specific functions.
    " ─────────────────────────────────────────────────────────────────

    DECLARE lv_date_from NVARCHAR(8);
    DECLARE lv_date_to   NVARCHAR(8);

    " SQLScript doesn't use ABAP types directly — cast them
    lv_date_from := :iv_date_from;
    lv_date_to   := :iv_date_to;

    " Step 1: aggregate order values per customer
    lt_order_agg = SELECT
        h.kunnr,
        SUM( i.netwr ) AS total_netwr,
        h.waerk
      FROM vbak AS h
      INNER JOIN vbap AS i ON i.vbeln = h.vbeln
      WHERE h.bukrs   = :iv_bukrs
        AND h.erdat  >= :lv_date_from
        AND h.erdat  <= :lv_date_to
        AND h.abgru  = ''              -- not rejected
      GROUP BY h.kunnr, h.waerk;

    " Step 2: enrich with customer name
    et_result = SELECT
        a.kunnr,
        c.name1,
        a.total_netwr AS netwr,
        a.waerk
      FROM :lt_order_agg AS a
      INNER JOIN kna1 AS c ON c.kunnr = a.kunnr
      ORDER BY a.total_netwr DESC;

  ENDMETHOD.

ENDCLASS.
```

Key anatomy points:

| Element | What it means |
|---------|---------------|
| `IF_AMDP_MARKER_HDB` | Tells ABAP: "this class has DB procedures." Required. |
| `BY DATABASE PROCEDURE FOR HDB LANGUAGE SQLSCRIPT` | Everything below is SQLScript, sent to HANA |
| `OPTIONS READ-ONLY` | Optimisation hint — omit for write procedures |
| `USING vbak vbap kna1` | Explicit dependency declaration — ABAP tracks which tables the procedure uses |
| `:lt_order_agg` | SQLScript tabular variable (temp table) — the `:` prefix is SQLScript, not ABAP |
| `et_result` | The EXPORTING parameter — SQLScript assigns to it directly |

---

## 17.3 Calling AMDP from ABAP, debugging, and performance

### Calling it — nothing special

```abap
" Call an AMDP method exactly like any static class method
DATA lt_result TYPE zcl_sales_aggregator=>tt_sales_total.

TRY.
    zcl_sales_aggregator=>get_sales_by_customer(
      IMPORTING
        ev_result    = lt_result        " wait — EXPORTING in callee = IMPORTING in caller
      EXPORTING
        iv_bukrs     = '1000'
        iv_date_from = '20240101'
        iv_date_to   = '20241231'
    ).
  CATCH cx_amdp_error INTO DATA(lx_amdp).
    " AMDP-level error: procedure failed on HANA
    MESSAGE lx_amdp->get_text( ) TYPE 'E'.
ENDTRY.

LOOP AT lt_result INTO DATA(ls_line).
  WRITE: / ls_line-kunnr, ls_line-name1, ls_line-netwr, ls_line-waerk.
ENDLOOP.
```

> ⚠️ **C#/Python gotcha:** In ABAP the EXPORTING/IMPORTING direction is from the *callee's* point of view. So the method declares `EXPORTING et_result`; you receive it as `IMPORTING et_result` at the call site. This is backwards from what C# devs expect ("exporting" sounds like you'd be sending it *in*). Just remember: EXPORTING = OUT parameter = you get it back.

### Debugging AMDP

Regular ABAP debugger doesn't step into SQLScript. Your debugging options:

1. **ADT HANA Runtime Tools** (in Eclipse) — set a breakpoint in the SQLScript body with `BREAK` statements.
2. **HANA Studio / SQL Console** — copy the generated procedure from the HANA catalog (`SYS.PROCEDURES`) and run it directly.
3. **Add intermediate result SELECTs** — temporarily add a SELECT from `:lt_order_agg` and return it as a separate output table to inspect mid-procedure state.
4. **SQL EXPLAIN PLAN** — run the generated procedure through HANA's plan visualizer to find performance hotspots.

> 🧭 **On the job:** When an AMDP is slow, the first thing you check is whether the `USING` clause is missing a table (causing a full reference scan), whether the WHERE clause fields are in the HANA column store index, and whether you're doing unnecessary cross-joins in the SQLScript temp tables. It's the same thought process as analyzing a slow stored procedure in SQL Server.

### Performance considerations

- AMDP runs on the HANA DB engine — network round-trip to ABAP app server is eliminated.
- The SQLScript procedure is compiled and cached by HANA on first call.
- Parameters are passed as literals into the generated SQL; they are not bind variables by default, so the plan cache can fragment on large cardinality inputs — be aware of this.
- `OPTIONS READ-ONLY` allows HANA to apply additional read optimizations.
- Avoid returning huge result sets to ABAP; the entire `et_result` table crosses the DB↔AppServer boundary.

---

## 17.4 Choosing the tool: Open SQL vs CDS vs AMDP

This is the most practical question you'll face daily, and it maps perfectly to the debate every developer already knows.

### 1️⃣ The analogy

| DB World debate | ABAP/SAP equivalent |
|-----------------|---------------------|
| ORM (EF, SQLAlchemy) | Open SQL |
| SQL View / EF raw SQL | CDS View Entity |
| Stored Procedure | AMDP |

### The decision table

| Criterion | Open SQL | CDS View | AMDP |
|-----------|----------|----------|------|
| Simple SELECT / filter / sort | ✅ Best | ✅ Good | ❌ Overkill |
| Multi-table join (reusable) | 🟡 Possible | ✅ Best | ❌ Overkill |
| Annotations for Fiori/OData | ❌ N/A | ✅ Only option | ❌ N/A |
| Complex aggregation with window functions | 🟡 Possible | ✅ Good | ✅ Best |
| Procedural logic (loops, branches, temp tables) | ❌ N/A | ❌ Not supported | ✅ Only option |
| Must run on non-HANA DB | ✅ Yes | ✅ Yes | ❌ HANA only |
| Testable in pure ABAP unit tests | ✅ Easy (test double) | ✅ Easy | 🟡 Hard (needs real HANA) |
| Transport / versioning in ABAP | ✅ Yes | ✅ Yes | ✅ Yes |
| Performance on HANA for bulk data | 🟡 Good | ✅ Very good | ✅ Excellent |

### 3️⃣ Decision flow in code

```abap
" Rule of thumb in practice:
"
" 1. Can I write it as a SELECT ... FROM <cds_view> with a WHERE clause?
"    → Open SQL. Done.
"
" 2. Is this logic reused in multiple places, needs UI annotations,
"    or is it the data model for an OData/Fiori app?
"    → CDS View Entity. Define it once.
"
" 3. Does the logic require:
"    - Iterating over rows with running state?
"    - Complex multi-step intermediate temp tables?
"    - HANA-specific analytic functions that CDS doesn't surface?
"    - Demonstrated performance problem that ABAP-side processing causes?
"    → AMDP. And only then.
```

> 💡 **The clean-core angle:** AMDP is *not* clean core (it's HANA-specific, not ABAP Cloud Platform compatible in BTP environments). If you're building on BTP ABAP Environment (Steampunk), AMDP may not be available. For on-premise S/4HANA, it's fully supported. Know your target environment.

---

## 🧠 Recap

| Concept | C#/Python equivalent | ABAP/AMDP |
|---------|---------------------|-----------|
| Stored procedure in DB | `SqlCommand` / `cursor.callproc()` | AMDP method body (SQLScript) |
| Procedure declaration in code | Dapper wrapper class | ABAP class + `IF_AMDP_MARKER_HDB` |
| Temp table in procedure | `#temp` / CTE | SQLScript tabular variable `:lt_xxx` |
| Calling a procedure | `ExecuteStoredProcedure()` | Static class method call |
| Procedure-level exception | `SqlException` | `CX_AMDP_ERROR` |
| "When to use stored proc" debate | ORM vs raw SQL vs stored proc | Open SQL vs CDS vs AMDP |

**Four things to remember:**
1. AMDP = SQLScript stored procedure managed inside an ABAP class. HANA only.
2. The method body is pure SQLScript — no ABAP statements allowed inside it.
3. `USING` clause: declare every DB table/view the SQLScript references.
4. Default tool should be Open SQL or CDS; reach for AMDP only when procedural DB logic is genuinely needed and performance justifies the added complexity.

---

*[← Contents](../content.md) | [← Previous: CDS Views](16-cds-views.md) | [Next: OData & REST — The Big Picture →](18-odata-and-rest-intro.md)*
