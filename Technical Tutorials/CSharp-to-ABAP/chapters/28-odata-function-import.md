# Chapter 28: Function Imports

*When CRUD isn't enough — how to expose a business operation like "Approve" as a proper OData action.*

---

## 28.1 The problem: CRUD doesn't cover everything ☕

REST purists will tell you that everything is a resource and everything you do to resources is GET/POST/PUT/DELETE. That's beautiful in theory. In an ERP system built on 40 years of SAP business logic, reality is messier.

Consider an "Approve Sales Order" operation. What resource are you modifying? The order header? Sort of — but the *approval* is a *business event* with side effects: it might trigger a workflow, post an accounting document, update a status field, and send an email. Shoving all that into a `PATCH /SalesOrderHeaderSet('1001')` with `{ "Status": "B" }` is a lie — you're hiding the business intent and bypassing the SAP standard logic.

The OData answer to "I need an RPC-style action" is the **Function Import**.

### The three-pass mental model

| Analogy | What it is |
|---|---|
| A button that says "Approve" on a form | Not a data field — a *command* |
| A POST to `/orders/1001/approve` in a REST API | A named operation with a body and a return value |
| A Function Import in OData | The same thing — a named action, defined in metadata, callable by URL |

> 💡 OData v4 renamed these "Actions" (state-changing) and "Functions" (read-only). In OData v2 / SEGW, they're all called **Function Imports** regardless. On the job you'll hear both terms — know they're the same concept at different OData versions.

---

## 28.2 You already know this

### C# — Web API action method

```csharp
// C# — ASP.NET Core Web API
// This is NOT a standard REST resource operation — it's a command
[HttpPost("{orderId}/approve")]
public async Task<IActionResult> ApproveSalesOrder(
    string orderId,
    [FromBody] ApproveRequest request)
{
    var result = await _orderService.Approve(orderId, request.ApprovedBy, request.Note);
    if (!result.Success)
        return BadRequest(result.ErrorMessage);

    return Ok(new ApproveResult
    {
        OrderId    = orderId,
        NewStatus  = result.NewStatus,
        ApprovedAt = result.ApprovedAt
    });
}

public record ApproveRequest(string ApprovedBy, string Note);
public record ApproveResult(string OrderId, string NewStatus, DateTime ApprovedAt);
```

The caller:
```http
POST /api/orders/0000001001/approve
Content-Type: application/json

{ "ApprovedBy": "JDOE", "Note": "Approved after credit check." }
```

### Python — FastAPI POST endpoint

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

class ApproveRequest(BaseModel):
    approved_by: str
    note: str

class ApproveResult(BaseModel):
    order_id:    str
    new_status:  str
    approved_at: str

@router.post("/orders/{order_id}/approve", response_model=ApproveResult)
async def approve_sales_order(order_id: str, body: ApproveRequest):
    result = await order_service.approve(order_id, body.approved_by, body.note)
    return ApproveResult(
        order_id    = order_id,
        new_status  = result.new_status,
        approved_at = result.approved_at.isoformat()
    )
```

Now let's map that to the SAP/OData version.

---

## 28.3 Defining a Function Import in SEGW 🛠️

### Step 1 — Create the Function Import

In SEGW, right-click **Function Imports → Create** (or via the toolbar).

| Field | Value |
|---|---|
| Function Import name | `ApproveSalesOrder` |
| HTTP Method | `POST` ← state-changing operations MUST be POST |
| Return type | Entity type → `SalesOrderHeader` |
| Return cardinality | `0..1` (returns the updated header or nothing) |

> ⚠️ **C#/Python gotcha:** OData v2 Function Imports that change state **must** use HTTP POST. If you set a function import to GET and call it with a side effect, clients that cache GET responses (like some HTTP proxies) will break your system. GET = read-only. POST = action. This rule is non-negotiable.

### Step 2 — Add Parameters

In the **Parameters** tab of the Function Import editor:

| Parameter name | Type | Mode | Nullable |
|---|---|---|---|
| `OrderId` | `Edm.String` | In | No |
| `ApprovedBy` | `Edm.String` | In | No |
| `Note` | `Edm.String` | In | Yes |

### Step 3 — Generate

Hit Generate. SEGW updates `ZSALESORDER_SRV_MPC` and creates/updates the stub `ZSALESORDER_SRV_DPC_EXT`. Look for the method stub:

```abap
METHOD execute_action.
  " TODO: implement
ENDMETHOD.
```

The SEGW-generated metadata now includes:

```xml
<FunctionImport
  Name="ApproveSalesOrder"
  ReturnType="ZSALESORDER_SRV.SalesOrderHeader"
  m:HttpMethod="POST">
  <Parameter Name="OrderId"    Type="Edm.String" Mode="In" />
  <Parameter Name="ApprovedBy" Type="Edm.String" Mode="In" />
  <Parameter Name="Note"       Type="Edm.String" Mode="In" Nullable="true" />
</FunctionImport>
```

---

## 28.4 Redefining EXECUTE_ACTION in DPC_EXT 🔁

All function imports in a single service share **one** method: `execute_action`. You branch on `iv_action_name` to route to the right handler — like a dispatcher.

```abap
CLASS zsalesorder_srv_dpc_ext DEFINITION
  INHERITING FROM zsalesorder_srv_dpc
  FINAL
  CREATE PUBLIC.

PUBLIC SECTION.
  METHODS execute_action REDEFINITION.

PRIVATE SECTION.
  METHODS approve_sales_order
    IMPORTING
      it_parameter TYPE /iwbep/t_mgw_name_value_pair
      io_tech_request_context TYPE REF TO /iwbep/if_mgw_req_context
    CHANGING
      co_entity TYPE REF TO data
    RAISING
      /iwbep/cx_mgw_busi_exception
      /iwbep/cx_mgw_tech_exception.

ENDCLASS.

CLASS zsalesorder_srv_dpc_ext IMPLEMENTATION.

  "=========================================================================
  " EXECUTE_ACTION — router for all Function Imports in this service
  "=========================================================================
  METHOD execute_action.

    CASE iv_action_name.
      WHEN 'ApproveSalesOrder'.
        approve_sales_order(
          EXPORTING
            it_parameter            = it_parameter
            io_tech_request_context = io_tech_request_context
          CHANGING
            co_entity               = co_entity ).

      " Add more function imports here as WHEN branches
      WHEN 'RecalculatePrices'.
        " ... future implementation ...

      WHEN OTHERS.
        " Unknown action — let the framework raise a 404-style error
        SUPER->execute_action(
          EXPORTING iv_action_name            = iv_action_name
                    it_parameter              = it_parameter
                    io_tech_request_context   = io_tech_request_context
          CHANGING  co_entity                 = co_entity ).
    ENDCASE.

  ENDMETHOD.

  "=========================================================================
  " PRIVATE — ApproveSalesOrder implementation
  "=========================================================================
  METHOD approve_sales_order.

    " --- 1. Read the incoming parameters ------------------------------------
    DATA lv_order_id    TYPE vbeln_va.
    DATA lv_approved_by TYPE string.
    DATA lv_note        TYPE string.

    LOOP AT it_parameter INTO DATA(ls_param).
      CASE ls_param-name.
        WHEN 'OrderId'.    lv_order_id    = ls_param-value.
        WHEN 'ApprovedBy'. lv_approved_by = ls_param-value.
        WHEN 'Note'.       lv_note        = ls_param-value.
      ENDCASE.
    ENDLOOP.

    " --- 2. Validate --------------------------------------------------------
    IF lv_order_id IS INITIAL.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid  = /iwbep/cx_mgw_busi_exception=>business_error
          message = 'OrderId is required'.
    ENDIF.

    " Check the order exists
    SELECT SINGLE gbstk FROM vbak INTO @DATA(lv_current_status)
      WHERE vbeln = @lv_order_id.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid = /iwbep/cx_mgw_busi_exception=>entity_not_found.
    ENDIF.

    IF lv_current_status = 'C'.   " 'C' = Completed
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid  = /iwbep/cx_mgw_busi_exception=>business_error
          message = 'Order is already completed — cannot approve'.
    ENDIF.

    " --- 3. Business logic — call the SAP standard BAPI --------------------
    "     In a real project you'd call BAPI_SALESORDER_CHANGE or a custom FM.
    "     Here we show the pattern with a direct DB update (illustration only).
    "     Production code always goes through BAPIs for change documents.
    DATA lt_return TYPE TABLE OF bapiret2.

    " Simulate BAPI call pattern:
    "   CALL FUNCTION 'Z_APPROVE_SALES_ORDER'
    "     EXPORTING iv_vbeln       = lv_order_id
    "               iv_approved_by = lv_approved_by
    "               iv_note        = lv_note
    "     TABLES    return         = lt_return.
    "
    "   LOOP AT lt_return INTO DATA(ls_ret) WHERE type CA 'EAX'.
    "     RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
    "       EXPORTING message = ls_ret-message.
    "   ENDLOOP.
    "
    "   COMMIT WORK AND WAIT.

    " For the demo — update status field directly
    UPDATE vbak SET gbstk = 'B'
      WHERE vbeln = lv_order_id.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid  = /iwbep/cx_mgw_busi_exception=>business_error
          message = 'Could not update order status'.
    ENDIF.

    COMMIT WORK AND WAIT.

    " --- 4. Read the updated header to return in the response --------------
    SELECT SINGLE *
      FROM vbak
      INTO @DATA(ls_vbak)
      WHERE vbeln = @lv_order_id.

    " Map to the OData entity type
    DATA(ls_entity) = VALUE zcl_zsalesorder_srv_mpc=>ts_salesorderheader(
      order_id   = ls_vbak-vbeln
      customer   = ls_vbak-kunnr
      order_date = ls_vbak-audat
      net_amount = ls_vbak-netwr
      currency   = ls_vbak-waerk
      status     = ls_vbak-gbstk   " Now 'B' (approved)
    ).

    " --- 5. Write the result to the output reference -----------------------
    "     co_entity is a REF TO data — we assign our structure to it
    GET REFERENCE OF ls_entity INTO co_entity.

  ENDMETHOD.

ENDCLASS.
```

> ⚠️ **C#/Python gotcha:** The `co_entity` output parameter is a `REF TO data` — a generic pointer. You don't cast it to the right type; instead you put your typed structure into a local variable and do `GET REFERENCE OF ls_entity INTO co_entity`. The framework knows what type to expect from the metadata and will cast it internally. If the types don't match, you'll get a runtime dump — the SEGW-generated type `ts_salesorderheader` must match exactly.

> ⚠️ **C#/Python gotcha:** Unlike a C# action method where you `return BadRequest(message)`, ABAP exceptions carry the error message. `RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception` with `message = '...'` is how you send an HTTP 400/422-style error back to the client. The framework converts it to an OData error response automatically.

---

## 28.5 Testing in /IWFND/GW_CLIENT and the request format 🎯

### In /IWFND/GW_CLIENT

1. Open `/IWFND/GW_CLIENT`.
2. Set **Service** = `ZSALESORDER_SRV`.
3. Set **HTTP Method** = `POST`.
4. **Request URI**:

```
/sap/opu/odata/sap/ZSALESORDER_SRV/ApproveSalesOrder?OrderId='0000001001'&ApprovedBy='JDOE'&Note='Approved after credit check'
```

For POST function imports with parameters, the parameters go in the query string (not the body). The request body is empty.

5. Hit **Execute** (F8). You should get HTTP 200 with the updated header entity.

### Alternatively — Postman / curl

```http
POST /sap/opu/odata/sap/ZSALESORDER_SRV/ApproveSalesOrder?OrderId='0000001001'&ApprovedBy='JDOE'&Note='Approved'
Authorization: Basic <base64>
Accept: application/json
X-CSRF-Token: <fetched token>
Content-Length: 0
```

```json
{
  "d": {
    "__metadata": { "type": "ZSALESORDER_SRV.SalesOrderHeader" },
    "OrderId":    "0000001001",
    "Customer":   "0000001000",
    "OrderDate":  "/Date(1716508800000)/",
    "NetAmount":  "2429.99",
    "Currency":   "USD",
    "Status":     "B"
  }
}
```

Notice `"Status": "B"` — the order has been approved.

### Error response (business validation failure)

```json
{
  "error": {
    "code":    "SY/530",
    "message": {
      "lang":  "en",
      "value": "Order is already completed — cannot approve"
    }
  }
}
```

> 🧭 **On the job:** You'll almost always need to fetch a CSRF token before calling POST-based function imports. The pattern: first do `GET /sap/opu/odata/sap/<SRV>/$metadata` with header `X-CSRF-Token: Fetch` — the response comes back with `X-CSRF-Token: <token>`. Then use that token value in the POST. Fiori handles this automatically; Postman / test code does not.

---

## 🧠 Recap

- **Function Imports** are the OData v2 way to expose named operations (commands) that don't map cleanly to CRUD.
- Define them in SEGW: name, HTTP method (POST for state changes), return type, parameters.
- All function imports share a single `EXECUTE_ACTION` method. Branch on `iv_action_name` with a `CASE` statement.
- Read parameters from `it_parameter` (name/value pairs). Raise `/iwbep/cx_mgw_busi_exception` for business errors.
- Return results by assigning your structure to `co_entity` via `GET REFERENCE OF`.
- Always use HTTP POST for side-effecting function imports — GET is for reads only.
- CSRF tokens are required for all non-GET calls from external clients.

*[← Contents](../content.md) | [← Previous: Header + Item Data in One Service](27-odata-header-and-item.md) | [Next: CREATE_DEEP_ENTITY →](29-odata-create-deep-entity.md)*
