# Chapter 21: Entity Types, Entity Sets, XML & JSON

*How SAP's data model vocabulary maps to what you already know — and what the wire actually looks like.*

---

## ☕ Setting the scene

You've already built REST APIs in C# or Python. You know that a `Product` class describes the shape of one product, and your `/api/products` endpoint returns a list of them. OData works on exactly that intuition — it just has different names for those ideas, and it codifies the whole thing in an XML document your client can read before the first data call.

This chapter is about the vocabulary and the data model: what an *entity type* is, how you define it in SEGW (SAP's classic OData builder), what the metadata document looks like, and what the wire payload looks like in both XML and JSON.

We'll build the data model for our running example — a Z-prefixed product/sales-order service — that every chapter in this part builds on. By the end of this chapter you have a working SEGW project with entity types defined and the `$metadata` URL returning real EDMX.

---

## 21.1 Entity Type vs Entity Set — class vs collection

### 1️⃣ The analogy

An **Entity Type** is the *blueprint* for one record: "a Product has a ProductID, a Name, a Price, a Currency." An **Entity Set** is the *named collection* of those records that lives at a URL: `/Products`. One is a type definition; the other is a resource you call.

### 2️⃣ You already know this

```csharp
// C# — the class IS the Entity Type
public class Product
{
    public string ProductId { get; set; }   // key
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Currency { get; set; }
}

// The DbSet IS the Entity Set (EF Core)
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }   // → /Products
}

// And the controller route maps to it:
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                // GET /api/Products   → the collection
    public IEnumerable<Product> GetAll() { ... }

    [HttpGet("{id}")]        // GET /api/Products('P001') → one entity
    public Product GetById(string id) { ... }
}
```

```python
# Python / FastAPI — same idea
from pydantic import BaseModel

class Product(BaseModel):           # ← Entity Type
    product_id: str
    name: str
    price: float
    currency: str

@app.get("/Products")               # ← Entity Set URL
def get_products() -> list[Product]: ...

@app.get("/Products/{id}")          # ← single entity
def get_product(id: str) -> Product: ...
```

### 3️⃣ The ABAP / OData way

| OData term | C# equivalent | ABAP / SEGW location |
|---|---|---|
| Entity Type | Class / record | Node you create in SEGW model tree |
| Entity Set | DbSet + controller route | Auto-created from the Entity Type name |
| Property | Class property | Column in SEGW → Properties tab |
| Key Property | `[Key]` / primary key | Ticked as "Key" in SEGW |
| Navigation Property | `ICollection<>` reference | Defined when you add an Association (ch 26) |

In SEGW you get a tree on the left side that looks like this:

```
ZMY_PRODUCT_SRV
└── Data Model
    ├── Entity Types
    │   ├── Product          ← the type
    │   │   └── Properties: ProductId*, Name, Price, Currency
    │   └── SalesOrderHeader
    ├── Entity Sets
    │   ├── ProductSet       ← the collection (auto-named)
    │   └── SalesOrderHeaderSet
    └── Associations        (ch 26)
```

> 💡 **Naming rule:** SEGW names the Entity Set by appending `Set` to the Entity Type name by default. `Product` → `ProductSet`. You can rename it, but most teams leave it.

> ⚠️ **C#/Python gotcha:** In REST you design the URL first and the class follows. In OData / SEGW you design the *type* first and the URL (`/ProductSet`) is derived. If you rename the Entity Set after publishing, all client URLs break — so decide early.

---

## 21.2 Defining the data model in SEGW

### Creating the project

Open transaction `SEGW`. Click **Create Project**.

| Field | Value |
|---|---|
| Project | `ZMY_PRODUCT_SRV` |
| Description | `My Product & Sales Service` |
| Package | `$TMP` (or your Z package) |
| Generation Strategy | `Standard` |

### Adding an Entity Type

Right-click **Entity Types** → **Create**. Give it the name `Product`.

Then right-click **Product → Properties** → **Create** for each field:

| Property Name | ABAP Type | EDM Type | Key? |
|---|---|---|---|
| ProductId | `MATNR` | `Edm.String` | ✓ |
| Name | `MAKTX` | `Edm.String` | |
| Price | `KBETR` | `Edm.Decimal` | |
| Currency | `WAERS` | `Edm.String` | |
| CreatedOn | `DATS` | `Edm.DateTime` | |

### ☕ Importing from a DDIC structure (the real way)

Nobody types properties by hand in a real project. You create a DDIC structure (`SE11`) that mirrors your entity shape, then import it.

```abap
" SE11 — create structure ZS_PRODUCT
@EndUserText.label: 'Product structure for OData'
TYPES: BEGIN OF zs_product,
         product_id TYPE matnr,
         name       TYPE maktx,
         price      TYPE p DECIMALS 2,
         currency   TYPE waers,
         created_on TYPE dats,
       END OF zs_product.
```

In SEGW: right-click **Entity Types** → **Create** → tick **Import from DDIC Structure** → enter `ZS_PRODUCT`. SEGW reads every field, maps the ABAP types to EDM types, and saves you 20 minutes of clicking.

> 🧭 **On the job:** You will almost always import from a DDIC structure on real projects. The DDIC structure also acts as the ABAP counterpart of your DTO — it's the type you `SELECT` into and the type you fill `er_entity` with.

### Adding SalesOrderHeader and SalesOrderItem (our running example)

Repeat the process for two more entity types we'll use from ch 23 onward:

**SalesOrderHeader** (maps loosely to `VBAK`):

| Property | EDM Type | Key? |
|---|---|---|
| SalesOrderId | Edm.String | ✓ |
| CustomerId | Edm.String | |
| NetValue | Edm.Decimal | |
| Currency | Edm.String | |
| DocDate | Edm.DateTime | |

**SalesOrderItem** (maps loosely to `VBAP`):

| Property | EDM Type | Key? |
|---|---|---|
| SalesOrderId | Edm.String | ✓ |
| ItemNumber | Edm.String | ✓ |
| ProductId | Edm.String | |
| Quantity | Edm.Decimal | |
| NetValue | Edm.Decimal | |

### Generating the service classes

After saving, click **Generate** (the cog icon on the toolbar). SEGW creates four ABAP classes:

| Class | Role | You touch it? |
|---|---|---|
| `ZMY_PRODUCT_SRV_MPC` | Model Provider (base) | Never — regenerated |
| `ZMY_PRODUCT_SRV_MPC_EXT` | Model Provider Extension | Only if you need dynamic metadata |
| `ZMY_PRODUCT_SRV_DPC` | Data Provider (base) | Never — regenerated |
| `ZMY_PRODUCT_SRV_DPC_EXT` | Data Provider Extension | **Always** — your logic lives here |

> ⚠️ **C#/Python gotcha:** The `_EXT` suffix means "extension". Think of `_DPC_EXT` as a subclass where you override methods. The base class (`_DPC`) is regenerated whenever you regenerate the project, so your code in the base is wiped. Put everything in `_EXT`. This is the single most important rule in classic SEGW development.

### Registering the service

After generation, go to `/IWFND/MAINT_SERVICE` (or run it from the SEGW menu **Service Maintenance → SID → Add Service**). Register the service and assign it to a system alias. Once registered, the metadata URL is live:

```
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/$metadata
```

---

## 21.3 The metadata document ($metadata)

### What it is

The `$metadata` endpoint returns an **EDMX** (Entity Data Model XML) document. It describes every entity type, every property, every key, every navigation property, and every function import in your service. Clients like SAP Fiori or a .NET OData client read this *before* making data calls, so they know exactly what shape the data will be.

Think of it as a **Swagger/OpenAPI document** — but in XML, and the client library can auto-generate code from it.

> 🧭 **On the job:** When a Fiori developer says "the metadata is wrong" they mean this document. When an integration fails because a field is missing, this document is where you check first.

### A real EDMX snippet for our service

```xml
<?xml version="1.0" encoding="utf-8"?>
<edmx:Edmx Version="1.0"
           xmlns:edmx="http://schemas.microsoft.com/ado/2007/06/edmx">
  <edmx:DataServices m:DataServiceVersion="2.0"
    xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata">

    <Schema Namespace="ZMY_PRODUCT_SRV"
            xmlns="http://schemas.microsoft.com/ado/2008/09/edm"
            xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices"
            xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata"
            xmlns:sap="http://www.sap.com/Protocols/SAPData">

      <!-- ══ Entity Type: Product ══════════════════════════════════ -->
      <EntityType Name="Product" sap:content-version="1">
        <Key>
          <PropertyRef Name="ProductId"/>
        </Key>
        <Property Name="ProductId"  Type="Edm.String"   Nullable="false"
                  MaxLength="18"  sap:label="Product ID" sap:updatable="false"/>
        <Property Name="Name"       Type="Edm.String"   MaxLength="40"
                  sap:label="Description"/>
        <Property Name="Price"      Type="Edm.Decimal"  Precision="11" Scale="2"
                  sap:label="Price"    sap:unit="Currency"/>
        <Property Name="Currency"   Type="Edm.String"   MaxLength="5"
                  sap:label="Currency" sap:semantics="currency-code"/>
        <Property Name="CreatedOn"  Type="Edm.DateTime" sap:label="Created On"
                  sap:display-format="Date"/>
      </EntityType>

      <!-- ══ Entity Type: SalesOrderHeader ═════════════════════════ -->
      <EntityType Name="SalesOrderHeader" sap:content-version="1">
        <Key>
          <PropertyRef Name="SalesOrderId"/>
        </Key>
        <Property Name="SalesOrderId" Type="Edm.String"  Nullable="false" MaxLength="10"/>
        <Property Name="CustomerId"   Type="Edm.String"  MaxLength="10"/>
        <Property Name="NetValue"     Type="Edm.Decimal" Precision="15" Scale="2"/>
        <Property Name="Currency"     Type="Edm.String"  MaxLength="5"/>
        <Property Name="DocDate"      Type="Edm.DateTime"/>
        <!-- NavigationProperty to items added in ch 26 -->
      </EntityType>

      <!-- ══ Entity Set definitions ════════════════════════════════ -->
      <EntityContainer Name="ZMY_PRODUCT_SRV_Entities"
                       m:IsDefaultEntityContainer="true">
        <EntitySet Name="ProductSet"
                   EntityType="ZMY_PRODUCT_SRV.Product"
                   sap:addressable="true"
                   sap:deletable="true"
                   sap:updatable="true"
                   sap:creatable="true"/>
        <EntitySet Name="SalesOrderHeaderSet"
                   EntityType="ZMY_PRODUCT_SRV.SalesOrderHeader"
                   sap:addressable="true"
                   sap:deletable="false"
                   sap:updatable="true"
                   sap:creatable="true"/>
      </EntityContainer>

    </Schema>
  </edmx:DataServices>
</edmx:Edmx>
```

### Reading the EDMX — a guided tour

| EDMX element | What it means |
|---|---|
| `EntityType Name="Product"` | The blueprint for one product record |
| `Key / PropertyRef` | Which property(ies) make up the primary key |
| `Property Type="Edm.Decimal"` | The wire type — what the client sees (not the ABAP type) |
| `sap:label` | The UI label SAP Fiori will display |
| `sap:updatable="false"` | This field is read-only on PUT/PATCH |
| `sap:semantics="currency-code"` | Fiori knows to treat this as a currency selector |
| `EntityContainer` | The service namespace — think `DbContext` |
| `EntitySet` | The named collection at a URL (`/ProductSet`) |
| `sap:creatable/updatable/deletable` | Capability flags — Fiori reads these to hide/show buttons |

> 💡 The `sap:*` attributes are SAP's OData v2 annotations. They're what Fiori apps read to auto-configure UI controls. When a Fiori dev says "make this field read-only", you set `sap:updatable="false"` on the property in SEGW (right-click property → Edit Annotations).

---

## 21.4 XML vs JSON responses

OData v2 supports both XML (Atom) and JSON responses. By default the service returns XML. Append `?$format=json` (or send `Accept: application/json`) to get JSON.

### The request

```http
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')
    ?$format=json
Host: mydev.example.com
```

Or test it right in the SAP system with `/IWFND/GW_CLIENT` (Gateway Client) — it's the Postman of the SAP world.

### XML response (Atom format — the default)

```xml
<?xml version="1.0" encoding="utf-8"?>
<entry xml:base="https://mydev/sap/opu/odata/sap/ZMY_PRODUCT_SRV/"
       xmlns="http://www.w3.org/2005/Atom"
       xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices"
       xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata">
  <id>https://mydev/.../ProductSet('P-100')</id>
  <title type="text">ProductSet('P-100')</title>
  <updated>2026-05-25T08:30:00Z</updated>
  <category term="ZMY_PRODUCT_SRV.Product"
            scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
  <content type="application/xml">
    <m:properties>
      <d:ProductId>P-100</d:ProductId>
      <d:Name>Precision Lathe T500</d:Name>
      <d:Price m:type="Edm.Decimal">4250.00</d:Price>
      <d:Currency>EUR</d:Currency>
      <d:CreatedOn m:type="Edm.DateTime">/Date(1748131200000)/</d:CreatedOn>
    </m:properties>
  </content>
</entry>
```

### JSON response ($format=json)

```json
{
  "d": {
    "ProductId": "P-100",
    "Name": "Precision Lathe T500",
    "Price": "4250.00",
    "Currency": "EUR",
    "CreatedOn": "/Date(1748131200000)/",
    "__metadata": {
      "id": "https://mydev/.../ProductSet('P-100')",
      "uri": "https://mydev/.../ProductSet('P-100')",
      "type": "ZMY_PRODUCT_SRV.Product"
    }
  }
}
```

### Collection response (EntitySet)

```json
{
  "d": {
    "results": [
      {
        "ProductId": "P-100",
        "Name": "Precision Lathe T500",
        "Price": "4250.00",
        "Currency": "EUR",
        "CreatedOn": "/Date(1748131200000)/",
        "__metadata": { "type": "ZMY_PRODUCT_SRV.Product", ... }
      },
      {
        "ProductId": "P-101",
        "Name": "CNC Mill X200",
        "Price": "18900.00",
        "Currency": "EUR",
        "CreatedOn": "/Date(1748217600000)/",
        "__metadata": { "type": "ZMY_PRODUCT_SRV.Product", ... }
      }
    ]
  }
}
```

### Key differences to absorb

| Thing | What it means |
|---|---|
| `"d": { ... }` wrapper | OData v2 JSON always wraps data in a `"d"` property. Don't forget this when parsing in JavaScript. |
| `"d": { "results": [] }` | Collections live inside `d.results`, not directly in `d`. |
| `/Date(1748131200000)/` | OData v2 DateTime is milliseconds-since-epoch in this ugly string format. OData v4 uses ISO 8601. |
| `"Price": "4250.00"` (string!) | `Edm.Decimal` comes over as a quoted string in OData v2 JSON to preserve precision. |
| `__metadata` | Every entity has this internal OData object with its canonical URI and type. |

> ⚠️ **C#/Python gotcha:** That `/Date(ms)/` format will bite every .NET/Python dev the first time. Don't try to `DateTime.Parse` it directly. SAP Fiori's UI5 framework handles it automatically; if you're writing a custom client, use a helper. The moment is `new DateTime(1970, 1, 1).AddMilliseconds(ms)` in C#.

> 🧭 **On the job:** JSON is preferred by virtually every modern client (Fiori, mobile, Python scripts). You'll rarely see the XML Atom format unless you're debugging. Always test with `/IWFND/GW_CLIENT` using `$format=json` — it's faster to read.

---

## 🧠 Recap

- An **Entity Type** = class/record blueprint. An **Entity Set** = named URL collection (`/ProductSet`). One is a type; the other is a resource.
- In **SEGW** you define entity types by typing properties or (better) importing from a DDIC structure. Generation creates four classes — you only ever code in `*_DPC_EXT`.
- The **`$metadata`** endpoint returns EDMX/XML: the machine-readable contract your service publishes. Read it like you'd read Swagger.
- SAP adds `sap:*` annotations (label, creatable, updatable) that Fiori reads to build its UI automatically.
- Wire format: XML (Atom) by default; JSON via `?$format=json`. Collections live in `d.results`; dates are the `/Date(ms)/` abomination; decimals come as strings.
- Test everything in **`/IWFND/GW_CLIENT`** — it's Postman, but it's already in the system.

---

*[← Contents](../content.md) | [← Previous: ALE & IDocs](20-ale-idocs.md) | [Next: Service Methods & Import Parameters →](22-odata-methods-and-parameters.md)*
