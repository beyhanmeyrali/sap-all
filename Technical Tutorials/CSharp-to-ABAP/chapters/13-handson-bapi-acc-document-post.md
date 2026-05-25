# Chapter 13: Hands-On — Posting via BAPI_ACC_DOCUMENT_POST

*The FI posting BAPI is the "Hello World" of SAP finance integration. Let's do it properly — header, line items, commit, error handling, and all.*

---

## ☕ Why this chapter exists

Every SAP project eventually touches FI (Financial Accounting). Whether you're migrating legacy invoices, building an expense-posting integration, or automating inter-company transfers, you will need to post a financial document. `BAPI_ACC_DOCUMENT_POST` is the standard, supported way to do it.

This chapter is entirely hands-on. We go from "what is a financial document?" to a complete, runnable ABAP report you can paste into SE38, adjust the company code and G/L accounts to match your system, and execute.

---

## 13.1 What "Posting a Financial Document" Means

### 1️⃣ The analogy

In accounting, every transaction is a **double-entry**: for every debit there must be an equal credit. SAP enforces this as a hard rule — the sum of all line-item amounts in a document must be **zero** (debits and credits net to zero in local currency).

A posted FI document has:
- A **header** — company code, posting date, document type, currency.
- **Line items** — each line hits a G/L account (general ledger account) with either a debit (`S` = Soll in German) or credit (`H` = Haben) amount.
- The total of all `S` amounts − all `H` amounts must equal zero.

Think of it like a database transaction with a constraint check: if you don't balance, the whole thing rolls back.

### 2️⃣ You already know this

```csharp
// C# mental model of a balanced ledger transaction
public class JournalEntry
{
    public string CompanyCode  { get; set; }
    public DateTime PostingDate { get; set; }
    public string Currency     { get; set; }
    public List<JournalLine> Lines { get; set; }

    public bool IsBalanced() =>
        Lines.Sum(l => l.IsDebit ? l.Amount : -l.Amount) == 0;
}

public record JournalLine(string GlAccount, decimal Amount, bool IsDebit);
```

```python
# Python
from dataclasses import dataclass, field
from decimal import Decimal

@dataclass
class JournalEntry:
    company_code: str
    posting_date: str
    currency: str
    lines: list = field(default_factory=list)

    def is_balanced(self) -> bool:
        return sum(
            l['amount'] if l['is_debit'] else -l['amount']
            for l in self.lines
        ) == Decimal('0')
```

### The BAPI structures

`BAPI_ACC_DOCUMENT_POST` uses four key parameters you need to understand before writing a single line:

| Parameter | Type | Purpose |
|-----------|------|---------|
| `DOCUMENTHEADER` | Structure `BAPIACHE09` | Document header (company code, dates, type) |
| `ACCOUNTGL` | Table `BAPIACGL09` | G/L account line items (one row per line) |
| `CURRENCYAMOUNT` | Table `BAPIACCR09` | Amount + currency for each line (linked by `ITEMNO_ACC`) |
| `RETURN` | Table `BAPIRET2` | Errors, warnings, and the resulting document number |

> ⚠️ **C#/Python gotcha:** The amounts and the G/L line items live in *separate* tables, linked by `ITEMNO_ACC`. This surprises most newcomers. Line 1 in `ACCOUNTGL` has `ITEMNO_ACC = 1`; the corresponding row in `CURRENCYAMOUNT` also has `ITEMNO_ACC = 1`. They are joined by that key. Think of it as a one-to-one foreign key between two tables in the same BAPI call.

---

## 13.2 Reading the BAPI Interface in SE37

Before writing code, go to **SE37** and look at the interface yourself.

1. Type `SE37` in the SAP command field, press Enter.
2. In the "Function Module" field, type `BAPI_ACC_DOCUMENT_POST`.
3. Press Enter (or the Display button).
4. You see five tabs: **Attributes**, **Import**, **Export**, **Changing**, **Tables**, **Exceptions**.

The important ones:

**Import tab — `DOCUMENTHEADER` structure (BAPIACHE09):**

| Field | Type | Description |
|-------|------|------------|
| `BUS_ACT` | `CHAR4` | Business transaction (`RFBU` for GL posting) |
| `USERNAME` | `USNAM` | Who is posting (usually `sy-uname`) |
| `COMP_CODE` | `BUKRS` | Company code (e.g., `'1000'`) |
| `DOC_DATE` | `DATS` | Document date |
| `PSTNG_DATE` | `DATS` | Posting date |
| `DOC_TYPE` | `BLART` | Document type (`SA` = G/L account document, `KR` = vendor invoice) |
| `CURRENCY` | `WAERS` | Currency key (`USD`, `EUR`) |
| `REF_DOC_NO` | `XBLNR` | Reference document number (the external reference — useful for tracing) |
| `HEADER_TXT` | `BKTXT` | Header text |

**Tables tab — `ACCOUNTGL` (BAPIACGL09), key fields:**

| Field | Description |
|-------|------------|
| `ITEMNO_ACC` | Item number (1, 2, 3…) — links to `CURRENCYAMOUNT` |
| `GL_ACCOUNT` | G/L account number (10 chars, zero-padded) |
| `COMP_CODE` | Company code (repeat the header value) |
| `PSTNG_DATE` | Posting date (repeat the header value) |
| `ITEM_TEXT` | Line item text |
| `TAX_CODE` | Tax code (leave blank for simple G/L postings) |

**Tables tab — `CURRENCYAMOUNT` (BAPIACCR09), key fields:**

| Field | Description |
|-------|------------|
| `ITEMNO_ACC` | Links back to `ACCOUNTGL` row |
| `CURRENCY` | Currency key |
| `AMT_DOCCUR` | Amount in document currency (positive = debit, negative = credit) |

> 🧭 **On the job:** Press **F8** in SE37 to test `BAPI_ACC_DOCUMENT_POST` interactively. Fill the structures by clicking the table buttons, enter valid company code, G/L accounts, and a balanced amount, and execute. Watch the RETURN table. This "test it in SE37 first" workflow saves hours of debug time on every new BAPI you encounter.

---

## 13.3 Filling the Header and Line-Item Tables in ABAP

Let's build the data structures before connecting everything into a full program.

```abap
" ── Local type declarations ──────────────────────────────────────────────────
DATA: ls_header     TYPE bapiache09,
      lt_accountgl  TYPE TABLE OF bapiacgl09,
      lt_currency   TYPE TABLE OF bapiaccr09,
      lt_return     TYPE TABLE OF bapiret2.

DATA: ls_gl_line    TYPE bapiacgl09,
      ls_curr_line  TYPE bapiaccr09.

" ── Fill the document header ──────────────────────────────────────────────────
ls_header-bus_act    = 'RFBU'.             " GL posting
ls_header-username   = sy-uname.           " current user
ls_header-comp_code  = '1000'.             " your company code
ls_header-doc_date   = sy-datum.           " today
ls_header-pstng_date = sy-datum.           " today
ls_header-doc_type   = 'SA'.               " SA = GL account document
ls_header-currency   = 'USD'.
ls_header-ref_doc_no = 'EXT-REF-001'.      " external reference
ls_header-header_txt = 'Test posting from ABAP'.

" ── Line item 1: DEBIT  account 400000 (expense), USD 1000 ───────────────────
CLEAR ls_gl_line.
ls_gl_line-itemno_acc  = '1'.
ls_gl_line-gl_account  = '0000400000'.     " expense GL account (10 digits, zero-padded)
ls_gl_line-comp_code   = '1000'.
ls_gl_line-pstng_date  = sy-datum.
ls_gl_line-item_text   = 'Expense line'.
APPEND ls_gl_line TO lt_accountgl.

CLEAR ls_curr_line.
ls_curr_line-itemno_acc  = '1'.
ls_curr_line-currency    = 'USD'.
ls_curr_line-amt_doccur  = '1000.00'.      " positive = debit
APPEND ls_curr_line TO lt_currency.

" ── Line item 2: CREDIT account 100000 (bank), USD -1000 ─────────────────────
CLEAR ls_gl_line.
ls_gl_line-itemno_acc  = '2'.
ls_gl_line-gl_account  = '0000100000'.     " bank GL account
ls_gl_line-comp_code   = '1000'.
ls_gl_line-pstng_date  = sy-datum.
ls_gl_line-item_text   = 'Bank clearing'.
APPEND ls_gl_line TO lt_accountgl.

CLEAR ls_curr_line.
ls_curr_line-itemno_acc  = '2'.
ls_curr_line-currency    = 'USD'.
ls_curr_line-amt_doccur  = '-1000.00'.     " negative = credit
APPEND ls_curr_line TO lt_currency.
```

> 💡 **The zero-sum rule in practice:** Line 1 is +1000.00 (debit), Line 2 is −1000.00 (credit). Sum = 0. If your amounts don't sum to zero, the BAPI will put error message `F5703` in the RETURN table and no document will be posted. No exceptions are thrown — check RETURN every time.

---

## 13.4 BAPI_TRANSACTION_COMMIT and Checking the RETURN Table

### Why commit is separate

BAPIs run inside SAP's **Logical Unit of Work (LUW)** — the database transaction. When you call a write BAPI, it locks records and stages changes in SAP's update task. Nothing is committed to the database until you explicitly call `BAPI_TRANSACTION_COMMIT`. This is deliberate: you can call multiple BAPIs in sequence (post header, post items, trigger follow-on document) and commit them all at once. If anything goes wrong, you call `BAPI_TRANSACTION_ROLLBACK` and nothing is written.

```abap
" ── Call BAPI_TRANSACTION_COMMIT ─────────────────────────────────────────────
CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
  EXPORTING
    wait = 'X'.   " 'X' = synchronous — wait for update task to finish
                  " Space = asynchronous (returns before update completes)

" ── NEVER use COMMIT WORK directly after a BAPI write call ───────────────────
" COMMIT WORK.   " Wrong — bypasses BAPI's update task mechanism
```

> ⚠️ **C#/Python gotcha:** Do NOT use `COMMIT WORK` after a BAPI. SAP's update task (the background process that actually writes to the database) is only triggered correctly by `BAPI_TRANSACTION_COMMIT`. Using `COMMIT WORK` directly can cause partial writes and data inconsistency. This is one of the most common mistakes ABAP beginners make.

### Checking the RETURN table

The `RETURN` table has type `BAPIRET2`. Each row is a message with a `TYPE` field:

| TYPE | Meaning | Action |
|------|---------|--------|
| `S` | Success | All good |
| `I` | Information | Informational — read it |
| `W` | Warning | Check — may still succeed |
| `E` | Error | Failed — do NOT commit, call ROLLBACK |
| `A` | Abort | Hard failure — call ROLLBACK |

```abap
" ── Check RETURN table and decide commit or rollback ─────────────────────────
DATA: lv_has_error TYPE abap_bool VALUE abap_false,
      lv_doc_num   TYPE bkpf-belnr.

LOOP AT lt_return INTO DATA(ls_return).
  WRITE: / ls_return-type, ls_return-id, ls_return-number, ls_return-message.

  IF ls_return-type = 'E' OR ls_return-type = 'A'.
    lv_has_error = abap_true.
  ENDIF.

  " The document number is in the return message parameter for this BAPI
  IF ls_return-type = 'S' AND ls_return-id = 'RW' AND ls_return-number = '605'.
    " Message RW 605: "Document &1 was posted in company code &2 for fiscal year &3"
    lv_doc_num = ls_return-message_v1.
  ENDIF.
ENDLOOP.

IF lv_has_error = abap_true.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
  WRITE: / 'Posting failed — rolled back.'.
ELSE.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING wait = 'X'.
  WRITE: / |Document posted: { lv_doc_num }|.
ENDIF.
```

---

## 13.5 Complete Runnable ABAP Report

Here is a full, self-contained report you can create in **SE38** (or ADT). Adjust `lc_bukrs` (company code), `lc_gl_debit`, and `lc_gl_credit` to valid accounts in your system. The rest will work as-is on any SAP system with FI configured.

```abap
*&---------------------------------------------------------------------*
*& Report  ZTEST_BAPI_FI_POST
*& Purpose: Demonstrate BAPI_ACC_DOCUMENT_POST — post a balanced
*&          2-line G/L document and print the resulting document number.
*&
*& HOW TO USE:
*&   1. Open SE38, create program ZTEST_BAPI_FI_POST (type 1, no pool)
*&   2. Paste this code
*&   3. Adjust the constants section to match your system
*&   4. F8 to execute
*&---------------------------------------------------------------------*
REPORT ztest_bapi_fi_post.

*----------------------------------------------------------------------*
* CONSTANTS — Adjust these to your system
*----------------------------------------------------------------------*
CONSTANTS:
  lc_bukrs      TYPE bukrs  VALUE '1000',   " Company code
  lc_gl_debit   TYPE hkont  VALUE '0000400000',  " Expense GL account (debit)
  lc_gl_credit  TYPE hkont  VALUE '0000100000',  " Bank/clearing GL (credit)
  lc_currency   TYPE waers  VALUE 'USD',
  lc_amount     TYPE wrbtr  VALUE '500.00',
  lc_doc_type   TYPE blart  VALUE 'SA'.     " SA = GL account document

*----------------------------------------------------------------------*
* DATA DECLARATIONS
*----------------------------------------------------------------------*
DATA: ls_header     TYPE bapiache09,
      lt_accountgl  TYPE TABLE OF bapiacgl09,
      lt_currency   TYPE TABLE OF bapiaccr09,
      lt_return     TYPE TABLE OF bapiret2.

DATA: ls_gl         TYPE bapiacgl09,
      ls_curr       TYPE bapiaccr09,
      ls_ret        TYPE bapiret2.

DATA: lv_has_error  TYPE abap_bool,
      lv_doc_number TYPE belnr_d.

*----------------------------------------------------------------------*
* FILL DOCUMENT HEADER
*----------------------------------------------------------------------*
ls_header-bus_act    = 'RFBU'.
ls_header-username   = sy-uname.
ls_header-comp_code  = lc_bukrs.
ls_header-doc_date   = sy-datum.
ls_header-pstng_date = sy-datum.
ls_header-doc_type   = lc_doc_type.
ls_header-currency   = lc_currency.
ls_header-ref_doc_no = |ZTEST-{ sy-datum }|.
ls_header-header_txt = 'BAPI test posting from ZTEST_BAPI_FI_POST'.

*----------------------------------------------------------------------*
* LINE ITEM 1 — DEBIT (positive amount)
*----------------------------------------------------------------------*
CLEAR ls_gl.
ls_gl-itemno_acc = '1'.
ls_gl-gl_account = lc_gl_debit.
ls_gl-comp_code  = lc_bukrs.
ls_gl-pstng_date = sy-datum.
ls_gl-item_text  = 'Test expense debit line'.
APPEND ls_gl TO lt_accountgl.

CLEAR ls_curr.
ls_curr-itemno_acc = '1'.
ls_curr-currency   = lc_currency.
ls_curr-amt_doccur = lc_amount.           " positive = debit
APPEND ls_curr TO lt_currency.

*----------------------------------------------------------------------*
* LINE ITEM 2 — CREDIT (negative amount, balances line 1)
*----------------------------------------------------------------------*
CLEAR ls_gl.
ls_gl-itemno_acc = '2'.
ls_gl-gl_account = lc_gl_credit.
ls_gl-comp_code  = lc_bukrs.
ls_gl-pstng_date = sy-datum.
ls_gl-item_text  = 'Test bank clearing credit line'.
APPEND ls_gl TO lt_accountgl.

CLEAR ls_curr.
ls_curr-itemno_acc = '2'.
ls_curr-currency   = lc_currency.
ls_curr-amt_doccur = lc_amount * -1.      " negative = credit
APPEND ls_curr TO lt_currency.

*----------------------------------------------------------------------*
* CALL THE BAPI
*----------------------------------------------------------------------*
WRITE: / '--- Calling BAPI_ACC_DOCUMENT_POST ---'.
WRITE: / |  Company: { lc_bukrs }  Date: { sy-datum }  Amount: { lc_amount } { lc_currency }|.

CALL FUNCTION 'BAPI_ACC_DOCUMENT_POST'
  EXPORTING
    documentheader = ls_header
  TABLES
    accountgl      = lt_accountgl
    currencyamount = lt_currency
    return         = lt_return.

*----------------------------------------------------------------------*
* EVALUATE RETURN TABLE
*----------------------------------------------------------------------*
WRITE: / '--- RETURN messages ---'.
lv_has_error = abap_false.

LOOP AT lt_return INTO ls_ret.
  " Print each message with formatting
  WRITE: / |  [{ ls_ret-type }] { ls_ret-id } { ls_ret-number }: { ls_ret-message }|.

  CASE ls_ret-type.
    WHEN 'E' OR 'A'.
      lv_has_error = abap_true.
    WHEN 'S'.
      " Check for the document-posted success message
      " BAPI_ACC_DOCUMENT_POST puts the document number in MESSAGE_V1
      " for message RW 605
      IF ls_ret-id = 'RW' AND ls_ret-number = '605'.
        lv_doc_number = ls_ret-message_v1.
      ENDIF.
  ENDCASE.
ENDLOOP.

*----------------------------------------------------------------------*
* COMMIT OR ROLLBACK
*----------------------------------------------------------------------*
IF lv_has_error = abap_true.
  WRITE: / '--- Errors found — calling ROLLBACK ---'.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
  WRITE: / 'Document NOT posted. Check the error messages above.'.
ELSE.
  WRITE: / '--- No errors — calling COMMIT (WAIT = X) ---'.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING
      wait = 'X'.

  IF lv_doc_number IS NOT INITIAL.
    WRITE: / |SUCCESS: FI document { lv_doc_number } posted in company { lc_bukrs }.|.
    WRITE: / |  -> Open FB03 and enter document { lv_doc_number } / { lc_bukrs } to view it.|.
  ELSE.
    " Commit succeeded but we didn't parse the doc number from the messages.
    " Check RETURN table above for message RW605 manually.
    WRITE: / 'COMMIT called — check FB03 for the new document.'.
  ENDIF.
ENDIF.

WRITE: / '--- Done ---'.
```

> 🧭 **On the job:** After a successful posting, open **FB03** (Display FI Document) and enter the document number, company code, and fiscal year to verify. You'll see the two line items, the debit and credit. This is the manual verification step your FI consultant will ask you to do when you're testing an integration.

> ⚠️ **C#/Python gotcha:** The document number returned by `BAPI_ACC_DOCUMENT_POST` is *not* directly in an `EXPORTING` parameter — it comes back inside the `RETURN` table as `MESSAGE_V1` of success message `RW 605`. This pattern of "the result is embedded in a return message" surprises most developers the first time. Always open the BAPI in SE37 and test manually so you know where the output lives.

---

## Common Errors and What They Mean

| RETURN message | What went wrong | Fix |
|---------------|-----------------|-----|
| `F5703 Debit/credit amounts do not balance` | Sum of AMT_DOCCUR ≠ 0 | Verify positive + negative amounts net to zero |
| `GLT2201 Account XXXXXXXX requires...` | G/L account configuration issue | Check the account in FS00 with your FI consultant |
| `F5808 Company code XXXX does not exist` | Wrong company code | Check T001 table for valid bukrs |
| `F5 002 Date XXXXXXXX not in open period` | Posting date is in a closed fiscal period | Use a date in the current open period |
| `RW 605 Document & posted in company code...` | **Success** — this is what you want | Parse MESSAGE_V1 for the document number |

---

## 🧠 Recap

- A financial document must **balance to zero** — sum of all line amounts = 0.
- `BAPI_ACC_DOCUMENT_POST` takes a header structure (`BAPIACHE09`), a G/L line table (`ACCOUNTGL`), and a currency-amount table (`CURRENCYAMOUNT`) linked by `ITEMNO_ACC`.
- Test it interactively in **SE37** before writing code — this saves enormous debugging time.
- After calling the BAPI: check the `RETURN` table for `TYPE = 'E'` / `'A'` errors. If clean, call `BAPI_TRANSACTION_COMMIT` with `WAIT = 'X'`. If errors, call `BAPI_TRANSACTION_ROLLBACK`.
- The posted document number lives in `RETURN-MESSAGE_V1` of the `RW 605` success message.
- Verify the result in **FB03** (Display FI Document).

---

*[← Contents](../content.md) | [← Previous: Function Modules, RFC & BAPI](12-function-modules-rfc-bapi.md) | [Next: Enhancing Standard SAP →](14-abap-enhancements.md)*
