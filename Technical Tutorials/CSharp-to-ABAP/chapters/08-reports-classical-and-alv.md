# Chapter 8: Reports — Classical Lists and ALV Grids

*How ABAP turns a SELECT query into the output screen every SAP user expects — and why half your real-world tickets will be here.*

---

## ☕ What is an ABAP report, really?

Think about the simplest useful program you can imagine: the user fills in a few filter fields, hits a button, and gets a table of results. In .NET that might be an MVC action with a query form and a results view. In Python it could be a CLI script with `argparse` + a `pandas` `to_string()` call.

In SAP, that pattern has a name: **the ABAP report**. It's been the backbone of SAP output since the early 1990s, and — yes — you will maintain one on your first week. Understanding the event loop that drives it is the unlock.

---

## 8.1 The Report Event Model

### 1️⃣ The analogy

An ABAP report runs inside an invisible **event dispatcher**. The runtime fires named events in a fixed sequence; you hook into whichever ones you need. Skip an event and the runtime skips it silently — only your code runs.

Think of it like ASP.NET's request pipeline middleware, except it's completely synchronous, screen-oriented, and defined by keyword labels instead of `app.Use(...)`.

### 2️⃣ You already know this

```csharp
// C# console app lifecycle analogy
static void Main(string[] args)
{
    // INITIALIZATION — set defaults
    string cargoClass = args.FirstOrDefault() ?? "Y";

    // AT SELECTION-SCREEN — validate
    if (cargoClass.Length != 1) { Console.Error.WriteLine("Bad input"); return; }

    // START-OF-SELECTION — do the work
    var flights = FlightRepo.Query(cargoClass);

    // END-OF-SELECTION — finalize output
    foreach (var f in flights) Console.WriteLine(f);
}
```

```python
# Python equivalent
import argparse

def main():
    # INITIALIZATION
    parser = argparse.ArgumentParser()
    parser.add_argument("--class", default="Y")
    args = parser.parse_args()       # AT SELECTION-SCREEN (runtime handles)

    # START-OF-SELECTION
    flights = fetch_flights(args.__dict__["class"])

    # END-OF-SELECTION
    for f in flights:
        print(f)
```

### 3️⃣ The ABAP way

```abap
REPORT zmy_first_report.

" ── INITIALIZATION ──────────────────────────────────────────────
" Runs once before the selection screen is displayed.
" Use it to set default values for parameters.
INITIALIZATION.
  p_class = 'Y'.           " default the parameter below

" ── AT SELECTION-SCREEN ─────────────────────────────────────────
" Runs after the user clicks Execute but BEFORE your logic.
" Use it to validate input and MESSAGE to abort cleanly.
AT SELECTION-SCREEN.
  IF p_class NA 'YBC'.
    MESSAGE 'Cabin class must be Y, B or C' TYPE 'E'.
  ENDIF.

" ── START-OF-SELECTION ──────────────────────────────────────────
" Your main block. The runtime fires this after successful validation.
" Almost all reports live here.
START-OF-SELECTION.
  SELECT * FROM sflight
    INTO TABLE @DATA(lt_flights)
    WHERE class = @p_class.

  " ... display logic goes here (section 8.3 / 8.4)

" ── END-OF-SELECTION ────────────────────────────────────────────
" Fires after START-OF-SELECTION. Use for totals, summaries.
END-OF-SELECTION.
  WRITE: / 'Rows returned:', lines( lt_flights ).
```

> ⚠️ **C#/Python gotcha:** There is no `return` or `exit()` to jump between events. `MESSAGE TYPE 'E'` during `AT SELECTION-SCREEN` *aborts the run and redisplays the selection screen* — it does NOT throw an exception that propagates to `START-OF-SELECTION`. Think of it as a built-in validation short-circuit, not an exception.

**The full event sequence** (most reports only need the highlighted ones):

| Event | When it fires | Analogy |
|---|---|---|
| `LOAD-OF-PROGRAM` | Program first loaded into memory | Static constructor |
| **`INITIALIZATION`** | Before selection screen renders | Form `Load` with defaults |
| `AT SELECTION-SCREEN OUTPUT` | Every time selection screen redraws | `Paint` / re-render |
| **`AT SELECTION-SCREEN`** | After Execute, before main logic | Form validation on submit |
| **`START-OF-SELECTION`** | Main block — your workhorse | `Main()` / request handler |
| **`END-OF-SELECTION`** | After main block | Response finalization |
| `TOP-OF-PAGE` | Before each new output page | Page header template |
| `END-OF-PAGE` | After each output page fills | Page footer template |

> 🧭 **On the job:** 95 % of reports you'll maintain only use `INITIALIZATION`, `AT SELECTION-SCREEN`, and `START-OF-SELECTION`. The page-break events (`TOP-OF-PAGE`, `END-OF-PAGE`) belong to classical lists (section 8.3) and are rarely written new.

---

## 8.2 Selection Screens — Your Input Form

### 1️⃣ The analogy

A **selection screen** is SAP's built-in input form. The ABAP runtime renders it as a dialog before running your code. You declare what fields you want; SAP handles the UI, F4 value-help, translation, and layout automatically.

It's closest to an MVC form model or a CLI argument parser — except the runtime generates the HTML (SAP GUI screen) from your declarations.

### 2️⃣ You already know this

```csharp
// C# — a simple query form model
public class FlightSearchForm
{
    [Required]
    public string AirlineCode { get; set; }

    public DateTime? DepartureDateFrom { get; set; }
    public DateTime? DepartureDateTo   { get; set; }

    [Range(1, 9999)]
    public int? MaxRows { get; set; } = 100;
}
```

```python
# Python CLI equivalent
parser = argparse.ArgumentParser()
parser.add_argument("--airline",  required=True)
parser.add_argument("--date-low", type=lambda s: datetime.strptime(s, "%Y-%m-%d"))
parser.add_argument("--date-high", type=lambda s: datetime.strptime(s, "%Y-%m-%d"))
parser.add_argument("--max-rows", type=int, default=100)
```

### 3️⃣ The ABAP way

```abap
REPORT zflight_search.

" ── PARAMETERS ──────────────────────────────────────────────────
" A single-value input field. Like a text box / CLI arg.
PARAMETERS:
  p_carrid TYPE sflight-carrid OBLIGATORY,  " required; F4 help from DDIC
  p_maxrow TYPE i DEFAULT 100.

" ── SELECT-OPTIONS ──────────────────────────────────────────────
" A FROM / TO range pair (low value, high value, sign, option).
" Renders as two input fields + a button to open a multi-value popup.
SELECT-OPTIONS:
  s_fldate FOR sflight-fldate,              " date range; no OBLIGATORY needed
  s_seats  FOR sflight-seatsocc.

" ── SELECTION-SCREEN BLOCK ──────────────────────────────────────
" Adds a visual box (frame) with a title around fields.
SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-001.
  PARAMETERS p_detail AS CHECKBOX DEFAULT ' '.
SELECTION-SCREEN END OF BLOCK b1.

START-OF-SELECTION.
  SELECT carrid connid fldate seatsocc seatsmax
    FROM sflight
    INTO TABLE @DATA(lt_fl)
    WHERE carrid    = @p_carrid
      AND fldate    IN @s_fldate       " IN = range table automatically
      AND seatsocc  IN @s_seats
    UP TO @p_maxrow ROWS.
```

**Key `SELECT-OPTIONS` mechanics:**

A `SELECT-OPTIONS` field (`s_fldate` above) is actually a **ranges table** — a table with columns `SIGN`, `OPTION`, `LOW`, `HIGH`. The `IN` operator in Open SQL handles the whole thing for you.

```abap
" You can also build ranges programmatically:
DATA lr_date TYPE RANGE OF sflight-fldate.
lr_date = VALUE #(
  ( sign = 'I' option = 'BT' low = '20240101' high = '20241231' )
  ( sign = 'E' option = 'EQ' low = '20240701' )  " exclude July 1
).
SELECT ... WHERE fldate IN @lr_date.
```

> ⚠️ **C#/Python gotcha:** `SELECT-OPTIONS` is NOT a simple `WHERE field BETWEEN low AND high`. It's a multi-row range table that supports includes (`sign = 'I'`), excludes (`sign = 'E'`), patterns (`option = 'CP'` for LIKE), and multiple ranges at once. This power is why it's everywhere in real reports.

> 🧭 **On the job:** When a user says "I need to filter by multiple plant codes and exclude two of them," the answer is a `SELECT-OPTIONS` on `WERKS`. Don't reach for custom multi-value input logic — the runtime already does it.

---

## 8.3 Classical WRITE Lists — The Output You'll Still Encounter

### 1️⃣ The analogy

Before grids and HTML, SAP reports printed to a virtual page — like a dot-matrix printout that happened to appear on screen. The `WRITE` statement is the `Console.Write` of ABAP output. The result is a scrollable "list" with fixed columns, page breaks, and a toolbar.

### 2️⃣ You already know this

```csharp
// C# — formatted console output
Console.WriteLine($"{"Airline",-8} {"Flight",6} {"Date",-12} {"Seats",8}");
Console.WriteLine(new string('-', 40));
foreach (var f in flights)
    Console.WriteLine($"{f.Carrid,-8} {f.Connid,6} {f.Fldate,-12} {f.Seatsocc,8}");
```

```python
# Python — formatted print
print(f"{'Airline':<8} {'Flight':>6} {'Date':<12} {'Seats':>8}")
print("-" * 40)
for f in flights:
    print(f"{f['carrid']:<8} {f['connid']:>6} {f['fldate']:<12} {f['seatsocc']:>8}")
```

### 3️⃣ The ABAP way

```abap
START-OF-SELECTION.
  SELECT carrid connid fldate seatsocc seatsmax
    FROM sflight
    INTO TABLE @DATA(lt_fl)
    WHERE carrid = @p_carrid.

  " ── Header line (fires at TOP-OF-PAGE) ─────────────────────────
  " Defined separately, see below.

  LOOP AT lt_fl INTO DATA(ls_fl).
    WRITE: /  ls_fl-carrid,       " / = new line
              ls_fl-connid,
              ls_fl-fldate,
          (8) ls_fl-seatsocc,     " (8) = column width
              ls_fl-seatsmax.
    " Highlight sold-out flights
    IF ls_fl-seatsocc >= ls_fl-seatsmax.
      FORMAT COLOR COL_NEGATIVE INTENSIFIED ON.
      WRITE: / '*** FULL ***'.
      FORMAT COLOR OFF INTENSIFIED OFF.
    ENDIF.
  ENDLOOP.

TOP-OF-PAGE.
  WRITE: / 'Airline', 10 'Conn', 18 'Date', 26 'Occupied', 36 'Max'.
  ULINE.   " horizontal rule
```

**Common WRITE modifiers:**

| Syntax | Effect | C# analogy |
|---|---|---|
| `WRITE: /` | New line before output | `Console.WriteLine` |
| `WRITE: (8) field` | Fixed column width | `{field,8}` |
| `WRITE: 20 field` | Output at column 20 | Tab stop |
| `FORMAT COLOR COL_POSITIVE` | Green background | Console color |
| `ULINE` | Horizontal line | `new string('-', n)` |

> ⚠️ **C#/Python gotcha:** Classical lists are **read-only output**. You cannot make a cell editable or add a button inside a `WRITE` list. For interactive output — sorting, filtering, editing — you need ALV (next section). Classical lists still exist because they're fast, printable, and every SAP user knows how to use them.

> 🧭 **On the job:** You will find classical `WRITE` reports in every legacy system. You probably won't *write* new ones, but you'll be asked to "add a column" or "change the color for negative stock" — so knowing the syntax pays dividends immediately.

---

## 8.4 ALV — The Grid Every SAP User Expects

### 1️⃣ The analogy

ALV (ABAP List Viewer) is SAP's built-in `DataGridView` / `DataFrame` display component. Every SAP user knows how to sort, filter, total, export to Excel, and configure column layout — because ALV gives them all of that for free. Your job is just to provide the internal table; ALV does the rest.

There are two ALV APIs:

| API | Age | Verdict |
|---|---|---|
| `REUSE_ALV_GRID_DISPLAY` (function module) | Classic, pre-OO | You'll *read* this; don't write new |
| `CL_SALV_TABLE` (OO factory) | Modern (7.0+) | Write this. Clean, testable. |

We'll use `CL_SALV_TABLE` exclusively for new code.

### 2️⃣ You already know this

```csharp
// C# — DataGridView on a WinForms form
var grid = new DataGridView { DataSource = flightList };
grid.Columns["Carrid"].HeaderText = "Airline";
grid.Columns["Seatsocc"].DefaultCellStyle.Alignment = DataGridViewContentAlignment.MiddleRight;
form.Controls.Add(grid);
form.ShowDialog();
```

```python
# Python — pandas tabulate equivalent
import pandas as pd
from tabulate import tabulate

df = pd.DataFrame(flights)
df.rename(columns={"carrid": "Airline", "seatsocc": "Occupied"}, inplace=True)
print(tabulate(df, headers="keys", tablefmt="grid", showindex=False))
```

### 3️⃣ The ABAP way — CL_SALV_TABLE basics

```abap
" Minimal CL_SALV_TABLE usage pattern
TRY.
    DATA lo_alv TYPE REF TO cl_salv_table.

    cl_salv_table=>factory(
      IMPORTING
        r_salv_table = lo_alv
      CHANGING
        t_table      = lt_flights    " your internal table
    ).

    " Optional: configure columns
    DATA(lo_cols) = lo_alv->get_columns( ).
    lo_cols->set_optimize( abap_true ).   " auto-width

    " Display — takes over the screen
    lo_alv->display( ).

  CATCH cx_salv_msg INTO DATA(lx).
    MESSAGE lx->get_text( ) TYPE 'E'.
ENDTRY.
```

That's it. Three steps: fill your table, call `factory`, call `display`. The user gets full sort/filter/export for free.

---

## 8.5 Full Working ALV Report — Paste Into SE38 and Run

Here is a complete, standalone ALV report you can copy into `SE38`, create it in any `Z` package, and execute immediately on a system that has the `SFLIGHT` demo tables (standard on every ABAP trial/training system).

### The C#/Python analogue first

```csharp
// C# — "fetch flights and print a grid"
public class FlightRow
{
    public string Airline    { get; init; }
    public string Connection { get; init; }
    public DateTime Date     { get; init; }
    public decimal Price     { get; init; }
    public string Currency   { get; init; }
    public int SeatsOccupied { get; init; }
    public int SeatsMax      { get; init; }
    public double LoadFactor { get; init; }
}

var rows = db.Flights
    .Where(f => f.CarrId == carrierInput && f.FlDate >= dateFrom)
    .OrderBy(f => f.FlDate)
    .Select(f => new FlightRow
    {
        Airline       = f.CarrId,
        Connection    = f.ConnId,
        Date          = f.FlDate,
        Price         = f.Price,
        Currency      = f.Currency,
        SeatsOccupied = f.SeatsOcc,
        SeatsMax      = f.SeatsMax,
        LoadFactor    = f.SeatsMax > 0 ? (double)f.SeatsOcc / f.SeatsMax : 0
    })
    .ToList();

DataGrid.DataSource = rows;  // framework renders the grid
```

```python
# Python equivalent
import pandas as pd

df = pd.read_sql(
    "SELECT carrid, connid, fldate, price, currency, seatsocc, seatsmax "
    "FROM sflight WHERE carrid = %s AND fldate >= %s ORDER BY fldate",
    con, params=[carrier, date_from]
)
df["load_pct"] = (df["seatsocc"] / df["seatsmax"] * 100).round(1)
print(df.to_string(index=False))
```

### The full ABAP report

```abap
*&---------------------------------------------------------------------*
*& Report  ZDEMO_FLIGHT_ALV
*& Author  : Career-Switcher Exercises
*& Purpose : Full ALV report — SFLIGHT data, CL_SALV_TABLE, column config
*&---------------------------------------------------------------------*
REPORT zdemo_flight_alv.

"─────────────────────────────────────────────────────────────────────
" 1. TYPE DEFINITION — the shape of our output rows
"─────────────────────────────────────────────────────────────────────
TYPES: BEGIN OF ty_flight,
         carrid    TYPE sflight-carrid,
         connid    TYPE sflight-connid,
         fldate    TYPE sflight-fldate,
         price     TYPE sflight-price,
         currency  TYPE sflight-currency,
         seatsocc  TYPE sflight-seatsocc,
         seatsmax  TYPE sflight-seatsmax,
         load_pct  TYPE p LENGTH 5 DECIMALS 1,   " calculated: % full
       END OF ty_flight.

DATA gt_flights TYPE STANDARD TABLE OF ty_flight WITH EMPTY KEY.

"─────────────────────────────────────────────────────────────────────
" 2. SELECTION SCREEN
"─────────────────────────────────────────────────────────────────────
SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-001.
  PARAMETERS:
    p_carrid TYPE sflight-carrid DEFAULT 'LH',
    p_maxrow TYPE i              DEFAULT 200.
  SELECT-OPTIONS:
    s_fldate FOR sflight-fldate.
SELECTION-SCREEN END OF BLOCK b1.

"─────────────────────────────────────────────────────────────────────
" 3. INITIALIZATION — set a default date range (last 2 years)
"─────────────────────────────────────────────────────────────────────
INITIALIZATION.
  s_fldate-low  = cl_abap_context_info=>get_system_date( ) - 730.
  s_fldate-high = cl_abap_context_info=>get_system_date( ).
  APPEND s_fldate.

"─────────────────────────────────────────────────────────────────────
" 4. INPUT VALIDATION
"─────────────────────────────────────────────────────────────────────
AT SELECTION-SCREEN.
  IF p_carrid IS INITIAL.
    MESSAGE 'Enter an airline code (e.g. LH, AA, UA)' TYPE 'E'.
  ENDIF.
  IF p_maxrow <= 0 OR p_maxrow > 9999.
    MESSAGE 'Max rows must be between 1 and 9999' TYPE 'E'.
  ENDIF.

"─────────────────────────────────────────────────────────────────────
" 5. MAIN LOGIC
"─────────────────────────────────────────────────────────────────────
START-OF-SELECTION.

  " 5a. Fetch data from SFLIGHT
  SELECT carrid, connid, fldate, price, currency, seatsocc, seatsmax
    FROM sflight
    INTO CORRESPONDING FIELDS OF TABLE @gt_flights
    WHERE carrid = @p_carrid
      AND fldate IN @s_fldate
    ORDER BY fldate ASCENDING
    UP TO @p_maxrow ROWS.

  IF sy-subrc <> 0.
    MESSAGE |No flights found for airline { p_carrid }| TYPE 'I'.
    RETURN.
  ENDIF.

  " 5b. Calculate the load-factor column (not stored in DB)
  LOOP AT gt_flights ASSIGNING FIELD-SYMBOL(<ls_fl>).
    IF <ls_fl>-seatsmax > 0.
      <ls_fl>-load_pct = ( <ls_fl>-seatsocc / <ls_fl>-seatsmax ) * 100.
    ENDIF.
  ENDLOOP.

  " 5c. Hand off to display method
  lcl_display=>show( gt_flights ).

"─────────────────────────────────────────────────────────────────────
" 6. LOCAL CLASS — keeps display logic separate (good habit)
"─────────────────────────────────────────────────────────────────────
CLASS lcl_display DEFINITION.
  PUBLIC SECTION.
    CLASS-METHODS show
      IMPORTING it_data TYPE STANDARD TABLE.
ENDCLASS.

CLASS lcl_display IMPLEMENTATION.
  METHOD show.
    DATA lo_alv  TYPE REF TO cl_salv_table.
    DATA lo_cols TYPE REF TO cl_salv_columns_table.
    DATA lo_col  TYPE REF TO cl_salv_column_table.
    DATA lo_funs TYPE REF TO cl_salv_functions_list.
    DATA lo_disp TYPE REF TO cl_salv_display_settings.
    DATA lo_sort TYPE REF TO cl_salv_sorts.

    TRY.
        " ── Create ALV instance ──────────────────────────────────
        cl_salv_table=>factory(
          IMPORTING r_salv_table = lo_alv
          CHANGING  t_table      = it_data
        ).

        " ── Toolbar: all standard buttons (sort, filter, Excel) ──
        lo_funs = lo_alv->get_functions( ).
        lo_funs->set_all( abap_true ).

        " ── Display settings ─────────────────────────────────────
        lo_disp = lo_alv->get_display_settings( ).
        lo_disp->set_list_header( 'Flight Occupancy Report' ).
        lo_disp->set_striped_pattern( abap_true ).  " zebra rows

        " ── Column configuration ─────────────────────────────────
        lo_cols = lo_alv->get_columns( ).
        lo_cols->set_optimize( abap_true ).

        " Rename headers (DDIC names → human-readable)
        lo_col ?= lo_cols->get_column( 'CARRID' ).
        lo_col->set_long_text( 'Airline' ).

        lo_col ?= lo_cols->get_column( 'CONNID' ).
        lo_col->set_long_text( 'Connection' ).

        lo_col ?= lo_cols->get_column( 'FLDATE' ).
        lo_col->set_long_text( 'Flight Date' ).

        lo_col ?= lo_cols->get_column( 'PRICE' ).
        lo_col->set_long_text( 'Ticket Price' ).

        lo_col ?= lo_cols->get_column( 'SEATSOCC' ).
        lo_col->set_long_text( 'Seats Occupied' ).

        lo_col ?= lo_cols->get_column( 'SEATSMAX' ).
        lo_col->set_long_text( 'Seats Total' ).

        lo_col ?= lo_cols->get_column( 'LOAD_PCT' ).
        lo_col->set_long_text( 'Load %' ).

        " ── Default sort: newest flights first ───────────────────
        lo_sort = lo_alv->get_sorts( ).
        lo_sort->add_sort(
          columnname = 'FLDATE'
          order      = if_salv_c_sort_order=>descending
        ).

        " ── GO! ──────────────────────────────────────────────────
        lo_alv->display( ).

      CATCH cx_salv_msg cx_salv_not_found cx_salv_data_error INTO DATA(lx).
        MESSAGE lx->get_text( ) TYPE 'E'.
    ENDTRY.
  ENDMETHOD.
ENDCLASS.
```

> 💡 **Why is `lcl_display` a local class?** Keeping presentation logic in a class (even a local one) means you can test the display method independently and swap it out. The alternative — stuffing everything into `START-OF-SELECTION` — is exactly the kind of unmaintainable code you'll inherit and be asked to clean up.

> 🧭 **On the job:** In a real project the `lcl_display` pattern evolves into a full global class like `ZCL_FLIGHT_REPORT` that implements `IF_SALV_EVENTS_ACTIONS_TABLE` for click/double-click events. Interviewers love asking "how do you handle double-click on an ALV row?" — the answer is implementing that interface and registering a handler on `lo_alv->get_event( )`.

**Running the report:**

1. Open `SE38`, enter program name `ZDEMO_FLIGHT_ALV`, click **Create**.
2. Paste the code, set the program type to **Executable Program**, assign a `$TMP` package.
3. Press `F8` (Execute). The selection screen appears; press **Execute** again.
4. You get a full ALV grid with sort, filter, and Excel export — zero extra UI code.

---

## 🧠 Recap

| Concept | What it is | Your mental model |
|---|---|---|
| `INITIALIZATION` | Pre-screen setup | Form `Load` / arg defaults |
| `AT SELECTION-SCREEN` | Post-submit validation | Model validation / argparse error |
| `START-OF-SELECTION` | Main logic | `Main()` / request handler |
| `PARAMETERS` | Single input field | CLI arg / text input |
| `SELECT-OPTIONS` | Range input (multi-value) | QueryString range + multi-select |
| `WRITE` | Classical text output | `Console.WriteLine` with column layout |
| `CL_SALV_TABLE` | Modern ALV grid | DataGridView / pandas `to_string` |

The report event model is sequential and simple once you've seen it once. The real skill is knowing *which event to put your code in* — and defaulting to `START-OF-SELECTION` for almost everything.

---

*[← Contents](../content.md) | [← Previous: Modern ABAP Syntax](07-modern-abap-syntax.md) | [Next: Module Pool Programming →](09-module-pool-programming.md)*
