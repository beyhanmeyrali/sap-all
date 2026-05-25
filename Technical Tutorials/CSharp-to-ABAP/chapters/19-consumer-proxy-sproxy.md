# Chapter 19: RFC Web Services — The Consumer Proxy (SPROXY)

*ABAP calling external SOAP services — "Add Service Reference" but SAP-flavored.*

---

## ☕ The mental model first

So far we've talked about SAP *serving* data to clients. But the traffic goes both ways: your ABAP program often needs to *call* an external system — a payment gateway, an ERP from a supplier, a logistics partner's SOAP service, a customs authority API.

In .NET you'd right-click the project, choose "Add Service Reference", paste the WSDL URL, and Visual Studio generates a typed client class. In Python you'd use `zeep` to parse the WSDL and get a client object. The idea is the same in ABAP: **SPROXY** reads a WSDL, generates a local proxy class, and you call its methods as if you were calling a regular ABAP class. The runtime handles all the SOAP envelope construction, XML serialization, and HTTP plumbing behind the scenes.

> 🎯 **When you'll use this:** Any time ABAP needs to *call out* to an external web service — classic SOAP/WSDL-based integrations with banks, tax authorities, logistics providers, or older SAP systems exposed as web services. It's the outbound integration pattern.

---

## 19.1 The scenario: ABAP calling an external web service

### 1️⃣ The analogy

An external system publishes a SOAP web service with a WSDL. The WSDL is a contract — it says "I have an operation called `GetShippingStatus`, which takes a `TrackingNumber` and returns a `ShipmentStatus` response."

You want to call it from ABAP. You have two choices:
1. Build the HTTP request and XML envelope by hand — painful and error-prone.
2. Generate a **consumer proxy** from the WSDL — a local ABAP class that represents the remote service. You call the class; the proxy handles the plumbing.

SPROXY generates option 2.

### 2️⃣ You already know this

```csharp
// C# — "Add Service Reference" / WCF proxy (classic)
// Visual Studio reads the WSDL and generates ShippingServiceClient

var client = new ShippingServiceClient();
var request = new GetShippingStatusRequest
{
    TrackingNumber = "TRK123456",
    Carrier        = "DHL"
};

try
{
    GetShippingStatusResponse response = await client.GetShippingStatusAsync(request);
    Console.WriteLine($"Status: {response.Status}, ETA: {response.EstimatedDelivery}");
}
catch (FaultException<ServiceFaultDetail> ex)
{
    Console.WriteLine($"Service fault: {ex.Detail.Message}");
}
```

```csharp
// C# modern — typed HttpClient + Refit / NSwag generated client
// Same idea — you generate a typed client from a contract (WSDL or OpenAPI)
var client = new ShippingApiClient(httpClient);
var status = await client.GetShippingStatusAsync("TRK123456");
```

```python
# Python — zeep library (SOAP client, reads WSDL at runtime)
from zeep import Client

client = Client("https://shipping.partner.com/api?wsdl")
result = client.service.GetShippingStatus(
    TrackingNumber="TRK123456",
    Carrier="DHL"
)
print(f"Status: {result.Status}")
```

### 3️⃣ The ABAP way

In ABAP, SPROXY does what Visual Studio's "Add Service Reference" does: reads the WSDL, generates proxy classes. SOAMANAGER is where you configure the endpoint URL and authentication — like `appsettings.json` for your WCF client, or the HttpClient base address in .NET.

---

## 19.2 SPROXY — generating the consumer proxy

### What SPROXY does

**Transaction:** `SPROXY` (on the backend system)

1. You give SPROXY a WSDL URL or upload a WSDL file.
2. SPROXY parses the WSDL and generates:
   - An **interface** (the stub: method signatures matching the WSDL operations)
   - A **proxy class** (the implementation: handles HTTP, SOAP envelopes, XML serialization)
   - **DDIC types** for all request/response structures and complex types defined in the WSDL

3. You get a fully typed ABAP proxy class — no XML parsing in your business code.

### The generated artifacts

For a WSDL with one operation `ShipmentStatus_Query`, SPROXY might generate:

```
Package: ZSHIPPING_PROXY

Interface:       CO_ZSHIPPING_PROXY (the port type interface)
Proxy class:     CL_ZSHIPPING_PROXY (implements the interface)
Request type:    ZSHIPPING_STATUS_QUERY_REQUEST   (DDIC structure)
Response type:   ZSHIPPING_STATUS_QUERY_RESPONSE  (DDIC structure)
Fault type:      ZSHIPPING_FAULT_TYPE             (DDIC structure)
```

> 🛠️ **SPROXY step by step:**
> 1. Open `SPROXY` in SAP GUI.
> 2. **Create** → Choose "Service Consumer" → Paste WSDL URL or import WSDL file.
> 3. Enter a proxy name and target package.
> 4. SPROXY generates and activates all objects.
> 5. Inspect the generated interface and class — you'll see the operation methods and DDIC types.

> ⚠️ **C#/Python gotcha:** SPROXY works well with classic WS-* / document-literal SOAP. If the external service uses SOAP with non-standard WS-Security extensions or exotic envelope structures, the generated proxy may need tweaking. SPROXY doesn't handle all WSDL dialects perfectly — especially very old RPC-encoded WSDL styles. For those, you might fall back to `CL_HTTP_CLIENT` with manual XML (section 19.5).

---

## 19.3 Logical ports and SOAMANAGER

The generated proxy class doesn't hard-code the endpoint URL. That lives in a **logical port** configured in **SOAMANAGER** (transaction `SOAMANAGER`). Think of it as the `appsettings.json` that holds `"ShippingServiceUrl": "https://..."` — you configure it once per environment (DEV, QAS, PRD) and the code never changes.

### 1️⃣ The analogy

```csharp
// C# — HttpClient with named client + IOptions for endpoint config
// Program.cs:
builder.Services.AddHttpClient("ShippingAPI", client =>
{
    client.BaseAddress = new Uri(config["ShippingService:Url"]);
});
// appsettings.json:
// "ShippingService": { "Url": "https://shipping.partner.com/api" }

// The client code doesn't know the URL — it gets it from DI/config
var client = httpClientFactory.CreateClient("ShippingAPI");
```

SOAMANAGER is the ABAP equivalent: you set the endpoint URL, authentication, and timeouts once per logical port. The proxy code just references the port name.

### Configuring a logical port in SOAMANAGER

**Transaction:** `SOAMANAGER`

1. Open `SOAMANAGER` → **Web Service Administration → Manage Business System**.
2. Find your proxy class (e.g., `CL_ZSHIPPING_PROXY`).
3. Click **Create Logical Port**.
4. Enter:
   - **Port name:** e.g., `ZSHIPPING_DEFAULT`
   - **Description:** Shipping partner — default
   - **Is Default:** Yes (the proxy will use this port unless you specify another)
   - **WSDL URL or Endpoint URL:** `https://shipping.partner.com/soap/v2`
5. On the **Authentication** tab: choose Basic Auth or SSL client certificate and enter credentials.
6. Save and activate.

> 🧭 **On the job:** Logical ports are environment-specific configuration. DEV, QAS, and PRD will have different endpoints and credentials. SAP Basis manages these in production. Your job is to specify the correct port name in the ABAP code and document which logical ports the service needs in the transport request notes.

---

## 19.4 Calling the proxy method in ABAP

### 3️⃣ The ABAP way — a complete proxy call example

```abap
"! Calls an external shipping web service via the generated SPROXY consumer proxy.
"! Demonstrates request construction, response handling, and fault catching.
METHOD check_shipment_status.

  " ── 1. Instantiate the proxy class ────────────────────────────────
  DATA lo_proxy   TYPE REF TO cl_zshipping_proxy.
  DATA ls_request TYPE zshipping_status_query_request.
  DATA ls_response TYPE zshipping_status_query_response.

  TRY.
      " Constructor takes an optional logical port name.
      " If omitted, the default port (from SOAMANAGER) is used.
      CREATE OBJECT lo_proxy
        EXPORTING
          logical_port_name = 'ZSHIPPING_DEFAULT'.

    CATCH cx_ai_system_fault INTO DATA(lx_create).
      " Could not connect to endpoint — config problem in SOAMANAGER
      RAISE EXCEPTION TYPE zcx_shipping_error
        EXPORTING
          message = lx_create->get_text( ).
  ENDTRY.

  " ── 2. Fill the request structure (generated DDIC type) ────────────
  ls_request-tracking_number = iv_tracking_number.
  ls_request-carrier_code    = iv_carrier.

  " ── 3. Call the proxy method ───────────────────────────────────────
  TRY.
      lo_proxy->shipment_status_query(
        EXPORTING
          input  = ls_request
        IMPORTING
          output = ls_response
      ).

      " ── 4. Read the response ───────────────────────────────────────
      es_status-status      = ls_response-status_code.
      es_status-description = ls_response-status_text.
      es_status-eta         = ls_response-estimated_delivery_date.

    CATCH cx_ai_application_fault INTO DATA(lx_app_fault).
      " ── 5a. Application-level fault — the service returned a SOAP Fault
      "        with a typed fault message (defined in the WSDL)
      DATA ls_fault TYPE zshipping_fault_type.
      lx_app_fault->get_fault_detail( IMPORTING fault_detail = ls_fault ).
      RAISE EXCEPTION TYPE zcx_shipping_error
        EXPORTING
          message     = ls_fault-error_text
          error_code  = ls_fault-error_code.

    CATCH cx_ai_system_fault INTO DATA(lx_sys_fault).
      " ── 5b. System-level fault — HTTP error, timeout, SSL failure, etc.
      RAISE EXCEPTION TYPE zcx_shipping_error
        EXPORTING
          message = lx_sys_fault->get_text( ).
  ENDTRY.

ENDMETHOD.
```

### The two exception classes

| Exception | When | C# equivalent |
|-----------|------|---------------|
| `CX_AI_SYSTEM_FAULT` | HTTP failure, timeout, SSL error, connection refused, endpoint not reachable | `HttpRequestException`, `TaskCanceledException` (timeout) |
| `CX_AI_APPLICATION_FAULT` | Service returned a SOAP Fault element (the remote side processed the request but returned a business error) | `FaultException<TDetail>` (WCF) |

> ⚠️ **C#/Python gotcha:** The exception split is important. `CX_AI_SYSTEM_FAULT` means "we never got a proper response from the service." `CX_AI_APPLICATION_FAULT` means "the service answered, but with an error message." Handle them separately because the recovery strategies differ: a system fault might warrant a retry; an application fault is usually a data/business logic issue.

---

## 19.5 Calling REST/JSON directly without a proxy

SPROXY is for SOAP/WSDL. When the external service is REST/JSON (no WSDL), you have two options in modern ABAP:

### Option A: `CL_HTTP_CLIENT` (classic)

```abap
" Classic HTTP client — works everywhere (older systems too)
DATA lo_http    TYPE REF TO if_http_client.
DATA lo_request TYPE REF TO if_http_request.

" Create HTTP client
cl_http_client=>create_by_url(
  EXPORTING
    url                = 'https://api.partner.com/v1/shipment/TRK123456'
  IMPORTING
    client             = lo_http
  EXCEPTIONS
    argument_not_found = 1
    OTHERS             = 2
).

IF sy-subrc <> 0.
  " handle error
ENDIF.

" Set headers
lo_http->request->set_method( if_http_request=>co_request_method_get ).
lo_http->request->set_header_field(
  name  = 'Authorization'
  value = |Bearer { lv_token }|
).
lo_http->request->set_header_field(
  name  = 'Accept'
  value = 'application/json'
).

" Send
lo_http->send( ).
lo_http->receive( ).

" Read response
DATA lv_json   TYPE string.
DATA lv_status TYPE i.
lv_json   = lo_http->response->get_cdata( ).
lv_status = lo_http->response->get_status_code( ).

lo_http->close( ).

" Parse JSON — using class from SAP's JSON library
DATA lo_json TYPE REF TO cl_trex_json_deserializer.
" ... or use /ui2/cl_json for a simpler interface
```

### Option B: `IF_WEB_HTTP_CLIENT` (modern, ABAP 7.54+)

```abap
" Modern HTTP client — cleaner API, available on S/4HANA 2020+
DATA lo_dest   TYPE REF TO if_http_destination.
DATA lo_client TYPE REF TO if_web_http_client.

lo_dest = cl_http_destination_provider=>create_by_url(
              'https://api.partner.com' ).

lo_client = cl_web_http_client_manager=>create_by_http_destination(
                lo_dest ).

" Build request
DATA lo_req TYPE REF TO if_web_http_request.
lo_req = lo_client->get_http_request( ).
lo_req->set_header_field( i_name  = 'Authorization'
                          i_value = |Bearer { lv_token }| ).

" Execute GET
DATA lo_resp TYPE REF TO if_web_http_response.
lo_resp = lo_client->execute( if_web_http_client=>get ).

" Read JSON response
DATA lv_body TYPE string.
lv_body = lo_resp->get_text( ).

" Parse with /ui2/cl_json or sxml deserializer
DATA ls_result TYPE zty_shipment_status.
/ui2/cl_json=>deserialize(
  EXPORTING json = lv_body
  CHANGING  data = ls_result
).
```

> 💡 **When to use what:**
> - External service has a WSDL → **SPROXY** (type-safe, generated, easier to maintain).
> - External service is REST/JSON, modern system → **IF_WEB_HTTP_CLIENT**.
> - External service is REST/JSON, older system → **CL_HTTP_CLIENT**.
> - External service is another SAP system → **RFC / BAPI** (chapter 12) or **SPROXY** if exposed as web service.

---

## 🧠 Recap

| Concept | C#/Python equivalent | ABAP |
|---------|---------------------|------|
| "Add Service Reference" / WSDL import | WCF Add Service Reference / NSwag | `SPROXY` |
| Generated typed client class | `ShippingServiceClient` (WCF) | `CL_ZSHIPPING_PROXY` (generated) |
| Endpoint URL & auth config | `appsettings.json` + HttpClient config | Logical port in `SOAMANAGER` |
| Call a web service operation | `await client.GetShippingStatusAsync(req)` | `lo_proxy->shipment_status_query(...)` |
| HTTP/transport error | `HttpRequestException` | `CX_AI_SYSTEM_FAULT` |
| Service-level business error | `FaultException<TDetail>` | `CX_AI_APPLICATION_FAULT` |
| REST/JSON without WSDL | `HttpClient` / `requests` | `IF_WEB_HTTP_CLIENT` / `CL_HTTP_CLIENT` |

**Five things to remember:**
1. SPROXY generates a typed ABAP proxy class from a WSDL — same concept as "Add Service Reference."
2. SOAMANAGER holds endpoint URLs and credentials per environment — never hard-code them.
3. Always catch both `CX_AI_SYSTEM_FAULT` (transport error) and `CX_AI_APPLICATION_FAULT` (SOAP Fault) separately.
4. SPROXY is for SOAP/WSDL; for REST/JSON use `IF_WEB_HTTP_CLIENT` (modern) or `CL_HTTP_CLIENT` (classic).
5. The proxy class is generated — don't modify it by hand. If the WSDL changes, regenerate in SPROXY.

---

*[← Contents](../content.md) | [← Previous: OData & REST — The Big Picture](18-odata-and-rest-intro.md) | [Next: ALE & IDocs →](20-ale-idocs.md)*
