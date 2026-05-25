# Chapter 32: Google Form → SAP Integration

*Turning a free Google Form into a real SAP data-entry pipeline — and making your CV stand out.*

---

## ☕ Why this chapter exists

Portfolio projects are what separate "I read about ABAP" from "I built something." A Google Form feeding data into SAP is exactly the kind of thing a hiring manager hasn't seen a hundred times. It shows you understand both worlds, can wire them together over HTTP, and don't need a six-figure middleware tool to do it.

Here's what we're building: a Google Form that captures, say, a **service request** (customer name, issue description, priority). The moment someone submits the form, a small Google Apps Script fires, packages the data as JSON, and POSTs it to an **inbound OData or REST service** you've already built on SAP. SAP stores it in a Z-table. Done. That's a real integration pattern used in small-to-medium SAP shops worldwide.

---

## 32.1 The Scenario

Imagine you're working for a company that wants field technicians to submit service requests from their phones — but the company's SAP ticket system isn't mobile-friendly (shocking, I know). The team decides: use Google Forms (free, works on any phone, no app install needed) and integrate it with SAP.

The requirements:
- Technician fills out a Google Form.
- On submit, data lands in SAP as a record in `ZTECHSVC_REQ` (your Z-table).
- SAP team can then see the records in a report or Fiori app.

This is a **push integration**: Google pushes to SAP. SAP doesn't poll. That keeps SAP load minimal and latency low.

---

## 32.2 The Integration Flow

Let's map the whole thing before writing a line of code.

```mermaid
sequenceDiagram
    participant T as Technician (Browser)
    participant GF as Google Form
    participant GAS as Google Apps Script<br/>(onSubmit trigger)
    participant SAP as SAP ICF Endpoint<br/>(REST/OData)
    participant DB as SAP Z-Table

    T->>GF: Submit form
    GF->>GAS: onFormSubmit trigger fires
    GAS->>GAS: Build JSON payload
    GAS->>SAP: HTTP POST /sap/opu/odata/sap/ZSR_SRV/ServiceRequestSet<br/>(Authorization: Basic ...)
    SAP->>SAP: Authenticate, parse body
    SAP->>DB: INSERT into ZTECHSVC_REQ
    SAP-->>GAS: 201 Created + response JSON
    GAS-->>GAS: Log success / alert on failure
```

The stack is refreshingly simple:

| Layer | Technology | Your analogy |
|---|---|---|
| Form UI | Google Forms | An HTML `<form>` hosted by Google |
| Glue logic | Google Apps Script | A tiny Node.js-style JavaScript runtime, also hosted by Google — free |
| Transport | HTTPS POST | Plain REST, same as you'd call any API |
| Inbound handler | SAP ICF / OData service | Your ASP.NET Web API `[HttpPost]` controller |
| Persistence | ABAP + Z-table | Your EF `DbContext.SaveChanges()` |

> 🧭 **On the job:** This pattern also works for Microsoft Forms (via Power Automate), Typeform (via webhooks), or any form tool that can call a URL on submit. Once you've built the SAP side, swapping the form tool is trivial.

---

## 32.3 The Google Side — Apps Script

### Create the form and spreadsheet

1. Go to [forms.google.com](https://forms.google.com), create a form with fields: **TechnicianName** (Short answer), **CustomerName** (Short answer), **Issue** (Paragraph), **Priority** (Multiple choice: Low / Medium / High).
2. Click the green Sheets icon (Responses → Link to Sheets). This creates a linked spreadsheet — Apps Script lives there.
3. In the spreadsheet: **Extensions → Apps Script**.

### The onSubmit trigger script

```javascript
// Google Apps Script — Code.gs
// Fires every time the linked form is submitted.

const SAP_ENDPOINT = "https://your-sap-host.example.com/sap/opu/odata/sap/ZSR_SRV/ServiceRequestSet";
const SAP_USER     = "APIUSER";          // dedicated RFC/API user in SAP
const SAP_PASS     = "YourPassword123";  // store in Script Properties in prod!

/**
 * Trigger: Form → onSubmit
 * e.namedValues gives us { "Field Label": ["value"], ... }
 */
function onFormSubmit(e) {
  const vals = e.namedValues;

  const payload = {
    TechnicianName: (vals["TechnicianName"] || [""])[0],
    CustomerName:   (vals["CustomerName"]   || [""])[0],
    Issue:          (vals["Issue"]          || [""])[0],
    Priority:       (vals["Priority"]       || ["LOW"])[0].toUpperCase(),
    SubmitTimestamp: new Date().toISOString()
  };

  const options = {
    method:      "post",
    contentType: "application/json",
    headers: {
      "Authorization": "Basic " + Utilities.base64Encode(SAP_USER + ":" + SAP_PASS),
      "Accept":        "application/json",
      "x-csrf-token":  fetchCsrfToken()   // SAP OData requires a CSRF token on mutating calls
    },
    payload:     JSON.stringify(payload),
    muteHttpExceptions: true  // so we can read the error body on failure
  };

  try {
    const response = UrlFetchApp.fetch(SAP_ENDPOINT, options);
    const code     = response.getResponseCode();

    if (code >= 200 && code < 300) {
      Logger.log("SAP record created: " + response.getContentText());
    } else {
      Logger.log("SAP error " + code + ": " + response.getContentText());
      // In prod: send an alert email to an admin
      MailApp.sendEmail("admin@example.com", "SAP Form Integration Error",
        "HTTP " + code + "\n" + response.getContentText());
    }
  } catch (err) {
    Logger.log("Exception calling SAP: " + err.toString());
  }
}

/**
 * OData services protect POST/PUT/DELETE with a CSRF token.
 * Fetch it with a HEAD/GET that includes x-csrf-token: Fetch.
 */
function fetchCsrfToken() {
  const metaUrl = "https://your-sap-host.example.com/sap/opu/odata/sap/ZSR_SRV/";
  const resp = UrlFetchApp.fetch(metaUrl, {
    method: "get",
    headers: {
      "Authorization": "Basic " + Utilities.base64Encode(SAP_USER + ":" + SAP_PASS),
      "x-csrf-token":  "Fetch"
    },
    muteHttpExceptions: true
  });
  return resp.getHeaders()["x-csrf-token"] || "";
}
```

### Wire the trigger

In Apps Script: **Triggers → Add Trigger → onFormSubmit → From form → On form submit**. Save. Test by submitting the form and checking the Execution log.

> ⚠️ **C#/Python gotcha:** `e.namedValues` values are always **arrays** even if the field appears once — `vals["Field"][0]` not `vals["Field"]`. Forgetting this gives you `"undefined"` strings in SAP.

> 💡 **Security tip:** Never hardcode passwords in Apps Script source. Use **File → Project Properties → Script Properties** (key-value store) and read them with `PropertiesService.getScriptProperties().getProperty("SAP_PASS")`. The script source is visible to anyone with editor access on the spreadsheet.

---

## 32.4 The SAP Side — Inbound Service

You have two realistic choices for the inbound endpoint:

| Option | When to use | Notes |
|---|---|---|
| **OData service (SEGW)** | When the data is an entity you'll also read via Fiori/UI5 | More setup, but you get `$metadata`, Fiori-ready, filter/expand for free |
| **ICF handler (IF_HTTP_EXTENSION)** | Quick REST endpoint, no OData overhead | Less infrastructure, perfect for pure push integrations |

We'll do **both** — OData first (because you know SEGW from Part VI), then the ICF approach.

### Option A: OData CREATE_ENTITY (SEGW)

You've built this in Chapter 25. The SAP side for this integration is a standard `CREATE_ENTITY` on your `ServiceRequest` entity type. The only thing different is you're receiving JSON from Google, not from a Fiori app.

```abap
" ZSR_DPC_EXT — CREATE_ENTITY for entity type 'ServiceRequest'
METHOD serviceRequestset_create_entity.

  DATA: ls_req  TYPE ztechsvc_req,  " Your Z-structure matching the entity
        ls_data TYPE zcl_zsr_mpc=>ts_servicerequest.

  " 1. Deserialize the JSON body into the entity structure
  io_data_provider->read_entry_data( IMPORTING es_data = ls_data ).

  " 2. Map to DB structure
  ls_req-guid          = cl_system_uuid=>create_uuid_c32_static( ).
  ls_req-tech_name     = ls_data-technicianname.
  ls_req-cust_name     = ls_data-customername.
  ls_req-issue         = ls_data-issue.
  ls_req-priority      = ls_data-priority.
  ls_req-submit_ts     = ls_data-submittimestamp.
  ls_req-created_by    = sy-uname.
  ls_req-created_at    = sy-datum.

  " 3. Insert — let ABAP raise an exception on duplicate
  INSERT ztechsvc_req FROM ls_req.
  IF sy-subrc <> 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid  = /iwbep/cx_mgw_busi_exception=>business_error
        message = 'Insert failed — duplicate key?'.
  ENDIF.

  " 4. Hand the created entity back to the OData framework
  er_entity = ls_data.

ENDMETHOD.
```

### Option B: ICF REST handler (IF_HTTP_EXTENSION)

Sometimes you just want a plain HTTP endpoint without the OData ceremony. Create a class `ZCL_HTTP_SR_HANDLER` that implements `IF_HTTP_EXTENSION`, register it under a node in SICF (e.g., `/sap/zgform/sr`), and publish it.

```abap
CLASS zcl_http_sr_handler DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    INTERFACES if_http_extension.
ENDCLASS.

CLASS zcl_http_sr_handler IMPLEMENTATION.

  METHOD if_http_extension~handle_request.

    DATA: lv_body   TYPE string,
          ls_req    TYPE ztechsvc_req,
          lo_json   TYPE REF TO cl_trex_json_deserializer.

    " ── 1. Only accept POST ──────────────────────────────────────────
    IF server->request->get_method( ) <> 'POST'.
      server->response->set_status( code = 405 reason = 'Method Not Allowed' ).
      RETURN.
    ENDIF.

    " ── 2. Read raw JSON body ────────────────────────────────────────
    lv_body = server->request->get_cdata( ).

    " ── 3. Parse JSON → ABAP structure ──────────────────────────────
    " We use CL_SXML_STRING_READER + transformation, or the simpler
    " /UI2/CL_JSON (available from SAP_BASIS 7.40+).
    /ui2/cl_json=>deserialize(
      EXPORTING json = lv_body
      CHANGING  data = ls_req ).

    " ── 4. Stamp system fields ──────────────────────────────────────
    ls_req-guid       = cl_system_uuid=>create_uuid_c32_static( ).
    ls_req-created_by = server->request->get_header_field( 'x-sap-user' ).
    ls_req-created_at = sy-datum.

    " ── 5. Persist ──────────────────────────────────────────────────
    INSERT ztechsvc_req FROM ls_req.

    IF sy-subrc = 0.
      server->response->set_status( code = 201 reason = 'Created' ).
      server->response->set_cdata(
        data = |{"guid":"{ ls_req-guid }","status":"created"}| ).
      server->response->set_header_field(
        name = 'Content-Type' value = 'application/json' ).
    ELSE.
      server->response->set_status( code = 409 reason = 'Conflict' ).
    ENDIF.

  ENDMETHOD.

ENDCLASS.
```

> ⚠️ **C#/Python gotcha:** SAP's ICF handler `handle_request` has **no return type** — you communicate success/failure entirely by setting the response status and body on the `server` object. Forgetting `set_status` leaves the client with a confusing 200 even when you've done nothing.

### The Z-table

```abap
" SE11 → Database Table → ZTECHSVC_REQ
" Key field: GUID (CHAR 32, UUID)
" Other fields:
"   TECH_NAME  CHAR 80   (Technician name)
"   CUST_NAME  CHAR 80   (Customer name)
"   ISSUE      STRING    (Issue description)
"   PRIORITY   CHAR 6    (LOW / MEDIUM / HIGH)
"   SUBMIT_TS  CHAR 27   (ISO timestamp from Google)
"   CREATED_BY UNAME     (sy-uname)
"   CREATED_AT DATS      (sy-datum)
"   STATUS     CHAR 10   (OPEN/INPROG/CLOSED — default OPEN)
```

> 🧭 **On the job:** In a real project, `ISSUE` as a STRING field creates issues for some older ABAP tools (ABAP strings are heap-allocated, not inline). Prefer `CHAR 1333` or a text table for long text. Talk to your senior before choosing STRING in a DDIC table.

---

## 32.5 Testing End to End + Security Notes

### Manual testing before wiring Google

Use your favorite REST client (Postman, Insomnia, or the VS Code REST extension):

```http
POST https://your-sap-host/sap/opu/odata/sap/ZSR_SRV/ServiceRequestSet
Authorization: Basic QVBJVVNFUJI6UGFzc3dvcmQxMjM=
Content-Type: application/json
x-csrf-token: <token-from-metadata-call>

{
  "TechnicianName": "Jane Smith",
  "CustomerName": "Acme Corp",
  "Issue": "Printer offline on 2nd floor",
  "Priority": "MEDIUM",
  "SubmitTimestamp": "2025-10-01T09:30:00Z"
}
```

Expected: `HTTP 201 Created` + the new entity in the response body.

### End-to-end test

1. Open the Google Form in a browser.
2. Fill in test data and submit.
3. Wait ~5 seconds for the Apps Script trigger to fire.
4. In SAP: `SE16N → ZTECHSVC_REQ` — your record should be there.
5. Check Apps Script **Execution log** (View → Executions) if it doesn't appear.

### Security checklist

| Concern | Recommendation |
|---|---|
| **Credentials in Apps Script** | Use Script Properties, never hardcode |
| **SAP API user** | Create a dedicated user (type `S` or `B`) with only the minimal authorizations for `SRV_OData`; no dialog logon |
| **Network exposure** | Prefer SAP Cloud Connector + BTP connectivity over exposing port 443 directly from your SAP system to the internet |
| **CSRF token** | OData V2 requires it on POST/PUT/DELETE — always fetch it with `x-csrf-token: Fetch` first |
| **Input validation** | Validate `Priority` is one of LOW/MEDIUM/HIGH before inserting; otherwise ABAP will insert whatever the form sends |
| **HTTPS only** | Never transmit Basic Auth over plain HTTP — it's just Base64, not encryption |

> 💡 For production-grade setups, replace Basic Auth with **OAuth 2.0 Client Credentials** via SAP BTP's API Management or a service key. But Basic Auth with a dedicated API user is perfectly acceptable for internal tooling.

---

## 🧠 Recap

- Google Apps Script's `onFormSubmit` trigger is your free, hosted webhook glue layer.
- The CSRF token dance (`x-csrf-token: Fetch` → use the returned value on POST) is unique to SAP OData V2 — always remember it.
- The SAP inbound side is either a SEGW `CREATE_ENTITY` (OData, Fiori-ready) or an ICF `IF_HTTP_EXTENSION` handler (lightweight REST).
- Use a dedicated API user, Script Properties for credentials, and prefer SAP Cloud Connector for internet exposure.
- This project — Google Forms → SAP — is a genuine CV differentiator. Add it to your GitHub with a README and a screenshot.

---

*[← Contents](../content.md) | [← Previous: Upload & Download Files in OData](31-odata-file-upload-download.md) | [Next: WhatsApp Integration from ABAP →](33-whatsapp-integration.md)*
