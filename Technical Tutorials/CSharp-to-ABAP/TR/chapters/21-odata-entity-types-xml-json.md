# Kısım 21: Entity Type, Entity Set, XML & JSON

*SAP'ın veri modeli kavramları zaten bildiklerinizle nasıl örtüşüyor — ve wire üzerinde gerçekte neler taşınıyor.*

---

## ☕ Sahneyi kuralım

C# veya Python'da daha önce REST API'lar geliştirdiniz. Bir `Product` sınıfının tek bir ürünün şeklini tanımladığını, `/api/products` endpoint'inin ise bunların listesini döndürdüğünü biliyorsunuz. OData tam olarak bu sezgiyle çalışır — sadece bu kavramlara farklı isimler verir ve her şeyi, istemcinizin ilk veri çağrısından önce okuyabileceği bir XML belgesinde resmîleştirir.

Bu kısım, söz dizimi ve veri modeli üzerinedir: *entity type* nedir, SEGW'de (SAP'ın klasik OData oluşturucu aracı) nasıl tanımlanır, metadata belgesi nasıl görünür ve hem XML hem JSON olarak wire payload nasıl şekillenir.

Bu kitabın bu bölümündeki her kısımın üzerine inşa edileceği örnek servisimizin — Z önekli ürün/satış siparişi servisi — veri modelini burada oluşturacağız. Kısımın sonunda, entity type'ları tanımlı ve `$metadata` URL'i gerçek EDMX döndüren, çalışan bir SEGW projeniz olacak.

---

## 21.1 Entity Type - Entity Set — sınıf ve koleksiyon

### 1️⃣ Benzetme

**Entity Type**, tek bir kayıt için *şablondur*: "Bir Product'ın ProductID, Name, Price, Currency alanları vardır." **Entity Set** ise o kayıtların bir URL'de yaşayan *adlandırılmış koleksiyonudur*: `/Products`. Biri tip tanımıdır; diğeri çağırabileceğiniz bir kaynaktır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — sınıf, Entity Type'tır
public class Product
{
    public string ProductId { get; set; }   // anahtar
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Currency { get; set; }
}

// DbSet, Entity Set'tir (EF Core)
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }   // → /Products
}

// Ve controller route buna eşlenir:
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                // GET /api/Products   → koleksiyon
    public IEnumerable<Product> GetAll() { ... }

    [HttpGet("{id}")]        // GET /api/Products('P001') → tek entity
    public Product GetById(string id) { ... }
}
```

```python
# Python / FastAPI — aynı fikir
from pydantic import BaseModel

class Product(BaseModel):           # ← Entity Type
    product_id: str
    name: str
    price: float
    currency: str

@app.get("/Products")               # ← Entity Set URL
def get_products() -> list[Product]: ...

@app.get("/Products/{id}")          # ← tek entity
def get_product(id: str) -> Product: ...
```

### 3️⃣ ABAP'taki karşılığı

| OData terimi | C# karşılığı | ABAP / SEGW konumu |
|---|---|---|
| Entity Type | Sınıf / kayıt | SEGW model ağacında oluşturduğunuz düğüm |
| Entity Set | DbSet + controller route | Entity Type adından otomatik oluşturulur |
| Property | Sınıf özelliği | SEGW → Properties sekmesindeki sütun |
| Key Property | `[Key]` / birincil anahtar | SEGW'de "Key" olarak işaretlenir |
| Navigation Property | `ICollection<>` referansı | Association eklendiğinde tanımlanır (kısım 26) |

SEGW'de sol tarafta şöyle bir ağaç görürsünüz:

```
ZMY_PRODUCT_SRV
└── Data Model
    ├── Entity Types
    │   ├── Product          ← tip
    │   │   └── Properties: ProductId*, Name, Price, Currency
    │   └── SalesOrderHeader
    ├── Entity Sets
    │   ├── ProductSet       ← koleksiyon (otomatik adlandırılır)
    │   └── SalesOrderHeaderSet
    └── Associations        (kısım 26)
```

> 💡 **İsimlendirme kuralı:** SEGW, varsayılan olarak Entity Set adını Entity Type adına `Set` ekleyerek oluşturur. `Product` → `ProductSet`. İstediğinizde yeniden adlandırabilirsiniz, ama çoğu ekip varsayılanı bırakır.

> ⚠️ **C#/Python tuzağı:** REST'te önce URL'yi tasarlarsınız, sınıf bunu izler. OData / SEGW'de ise önce *tipi* tasarlarsınız ve URL (`/ProductSet`) bundan türetilir. Entity Set'i yayınladıktan sonra yeniden adlandırırsanız tüm istemci URL'leri bozulur — bu yüzden erken karar verin.

---

## 21.2 SEGW'de veri modelini tanımlama

### Projeyi oluşturma

`SEGW` transaction'ını açın. **Create Project**'e tıklayın.

| Alan | Değer |
|---|---|
| Project | `ZMY_PRODUCT_SRV` |
| Description | `My Product & Sales Service` |
| Package | `$TMP` (veya Z paketiniz) |
| Generation Strategy | `Standard` |

### Entity Type ekleme

**Entity Types**'a sağ tıklayın → **Create**. Adı `Product` olarak girin.

Ardından **Product → Properties**'e sağ tıklayın → her alan için **Create**:

| Özellik Adı | ABAP Tipi | EDM Tipi | Anahtar? |
|---|---|---|---|
| ProductId | `MATNR` | `Edm.String` | ✓ |
| Name | `MAKTX` | `Edm.String` | |
| Price | `KBETR` | `Edm.Decimal` | |
| Currency | `WAERS` | `Edm.String` | |
| CreatedOn | `DATS` | `Edm.DateTime` | |

### ☕ DDIC yapısından içe aktarma (gerçek yol)

Gerçek projelerde kimse özellikleri tek tek elle girmez. Entity şeklinizi yansıtan bir DDIC yapısı (`SE11`) oluşturursunuz, ardından bunu içe aktarırsınız.

```abap
" SE11 — ZS_PRODUCT yapısını oluştur
@EndUserText.label: 'OData için Ürün yapısı'
TYPES: BEGIN OF zs_product,
         product_id TYPE matnr,
         name       TYPE maktx,
         price      TYPE p DECIMALS 2,
         currency   TYPE waers,
         created_on TYPE dats,
       END OF zs_product.
```

SEGW'de: **Entity Types**'a sağ tıklayın → **Create** → **Import from DDIC Structure** seçeneğini işaretleyin → `ZS_PRODUCT` girin. SEGW tüm alanları okur, ABAP tiplerini EDM tiplerine eşler ve 20 dakikalık tıklamadan sizi kurtarır.

> 🧭 **İş hayatında:** Gerçek projelerde neredeyse her zaman DDIC yapısından içe aktarırsınız. DDIC yapısı aynı zamanda DTO'nun ABAP karşılığı işlevi görür — hem `SELECT` ile doldurduğunuz hem de `er_entity`'yi beslediğiniz tiptir.

### SalesOrderHeader ve SalesOrderItem ekleme (devam eden örneğimiz)

Kısım 23'ten itibaren kullanacağımız iki entity type daha için aynı işlemi tekrarlayın:

**SalesOrderHeader** (`VBAK`'a yakın bir eşlemeyle):

| Özellik | EDM Tipi | Anahtar? |
|---|---|---|
| SalesOrderId | Edm.String | ✓ |
| CustomerId | Edm.String | |
| NetValue | Edm.Decimal | |
| Currency | Edm.String | |
| DocDate | Edm.DateTime | |

**SalesOrderItem** (`VBAP`'a yakın bir eşlemeyle):

| Özellik | EDM Tipi | Anahtar? |
|---|---|---|
| SalesOrderId | Edm.String | ✓ |
| ItemNumber | Edm.String | ✓ |
| ProductId | Edm.String | |
| Quantity | Edm.Decimal | |
| NetValue | Edm.Decimal | |

### Servis sınıflarını oluşturma

Kaydettikten sonra **Generate** (araç çubuğundaki dişli simgesi) düğmesine tıklayın. SEGW dört ABAP sınıfı oluşturur:

| Sınıf | Rol | Dokunuyor musunuz? |
|---|---|---|
| `ZMY_PRODUCT_SRV_MPC` | Model Provider (taban) | Asla — yeniden oluşturulur |
| `ZMY_PRODUCT_SRV_MPC_EXT` | Model Provider Extension | Yalnızca dinamik metadata gerekirse |
| `ZMY_PRODUCT_SRV_DPC` | Data Provider (taban) | Asla — yeniden oluşturulur |
| `ZMY_PRODUCT_SRV_DPC_EXT` | Data Provider Extension | **Her zaman** — mantığınız burada yaşar |

> ⚠️ **C#/Python tuzağı:** `_EXT` soneki "uzantı" anlamına gelir. `_DPC_EXT`'yi, metodları override ettiğiniz bir alt sınıf olarak düşünün. Taban sınıf (`_DPC`) projeyi her yeniden oluşturduğunuzda tekrar üretilir, dolayısıyla taban sınıfa yazdığınız kod silinir. Her şeyi `_EXT`'ye koyun. Bu, klasik SEGW geliştirmesinin tek en önemli kuralıdır.

### Servisi kaydetme

Oluşturmadan sonra `/IWFND/MAINT_SERVICE`'e gidin (ya da SEGW menüsünden **Service Maintenance → SID → Add Service**). Servisi kaydedin ve bir sistem takma adına atayın. Kaydedildiğinde metadata URL kullanıma açılır:

```
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/$metadata
```

---

## 21.3 Metadata belgesi ($metadata)

### Nedir?

`$metadata` endpoint'i bir **EDMX** (Entity Data Model XML) belgesi döndürür. Bu belge; servisinizdeki her entity type'ı, her özelliği, her anahtarı, her navigation property'yi ve her function import'u açıklar. SAP Fiori veya bir .NET OData istemcisi gibi istemciler, veri çağrısı yapmadan önce bu belgeyi okur; böylece verinin tam olarak ne şekil alacağını bilirler.

Bunu bir **Swagger/OpenAPI belgesi** gibi düşünün — ama XML formatında ve istemci kütüphanesi bundan otomatik olarak kod üretebilir.

> 🧭 **İş hayatında:** Fiori geliştiricisi "metadata yanlış" dediğinde bu belgeyi kasteder. Entegrasyon, eksik bir alan nedeniyle başarısız olduğunda, önce bu belgeyi kontrol edersiniz.

### Servisimize ait gerçek bir EDMX parçası

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
        <!-- Kısım 26'da item'lara NavigationProperty ekleniyor -->
      </EntityType>

      <!-- ══ Entity Set tanımları ════════════════════════════════════ -->
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

### EDMX'i okumak — rehberli bir tur

| EDMX öğesi | Anlamı |
|---|---|
| `EntityType Name="Product"` | Tek bir ürün kaydının şablonu |
| `Key / PropertyRef` | Birincil anahtarı oluşturan özellik(ler) |
| `Property Type="Edm.Decimal"` | Wire tipi — istemcinin gördüğü (ABAP tipi değil) |
| `sap:label` | SAP Fiori'nin görüntüleyeceği UI etiketi |
| `sap:updatable="false"` | Bu alan PUT/PATCH'te salt okunurdur |
| `sap:semantics="currency-code"` | Fiori'ye bunu para birimi seçici olarak işlemesini söyler |
| `EntityContainer` | Servis ad alanı — `DbContext` olarak düşünün |
| `EntitySet` | Bir URL'deki adlandırılmış koleksiyon (`/ProductSet`) |
| `sap:creatable/updatable/deletable` | Yetenek bayrakları — Fiori düğmeleri gizlemek/göstermek için bunları okur |

> 💡 `sap:*` öznitelikleri, SAP'ın OData v2 annotasyon'larıdır. Fiori uygulamalarının UI kontrollerini otomatik yapılandırmak için okuduğu şeylerdir. Bir Fiori geliştiricisi "bu alanı salt okunur yap" dediğinde, SEGW'de özelliğe sağ tıklayıp **Edit Annotations**'ı seçerek `sap:updatable="false"` ayarlarsınız.

---

## 21.4 XML ve JSON yanıtları

OData v2 hem XML (Atom) hem de JSON yanıtlarını destekler. Varsayılan olarak servis XML döndürür. JSON almak için `?$format=json` ekleyin (ya da `Accept: application/json` başlığını gönderin).

### İstek

```http
GET /sap/opu/odata/sap/ZMY_PRODUCT_SRV/ProductSet('P-100')
    ?$format=json
Host: mydev.example.com
```

Ya da `/IWFND/GW_CLIENT` (Gateway Client) ile doğrudan SAP sisteminde test edin — SAP dünyasının Postman'ıdır.

### XML yanıtı (Atom formatı — varsayılan)

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

### JSON yanıtı ($format=json)

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

### Koleksiyon yanıtı (Entity Set)

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

### Özümsemeniz gereken temel farklar

| Konu | Anlamı |
|---|---|
| `"d": { ... }` sarmalayıcısı | OData v2 JSON verisi her zaman `"d"` özelliğinin içine sarmalanır. JavaScript'te ayrıştırırken bunu unutmayın. |
| `"d": { "results": [] }` | Koleksiyonlar doğrudan `d`'nin içinde değil, `d.results` içinde yer alır. |
| `/Date(1748131200000)/` | OData v2 DateTime, bu garip string formatında milisaniye-cinsinden-epoch'tur. OData v4 ISO 8601 kullanır. |
| `"Price": "4250.00"` (string!) | `Edm.Decimal`, OData v2 JSON'da hassasiyeti korumak amacıyla tırnaklı bir string olarak gelir. |
| `__metadata` | Her entity'de canonical URI'sini ve tipini içeren bu dahili OData nesnesi bulunur. |

> ⚠️ **C#/Python tuzağı:** O `/Date(ms)/` formatı, ilk karşılaşmada her .NET/Python geliştiricisini ısırır. Doğrudan `DateTime.Parse` ile parse etmeye çalışmayın. SAP Fiori'nin UI5 çerçevesi bunu otomatik işler; özel bir istemci yazıyorsanız bir yardımcı kullanın. C#'ta anlık değer şudur: `new DateTime(1970, 1, 1).AddMilliseconds(ms)`.

> 🧭 **İş hayatında:** JSON, neredeyse tüm modern istemciler (Fiori, mobil, Python betikleri) tarafından tercih edilir. XML Atom formatını yalnızca hata ayıklarken görürsünüz. Her testte `/IWFND/GW_CLIENT`'ı `$format=json` ile kullanın — okumak çok daha hızlıdır.

---

## 🧠 Özet

- **Entity Type** = sınıf/kayıt şablonu. **Entity Set** = adlandırılmış URL koleksiyonu (`/ProductSet`). Biri tip, diğeri kaynaktır.
- **SEGW**'de entity type'ları özellikleri tek tek girerek ya da (daha iyisi) DDIC yapısından içe aktararak tanımlarsınız. Oluşturma dört sınıf üretir — yalnızca `*_DPC_EXT`'ye kod yazarsınız.
- **`$metadata`** endpoint'i EDMX/XML döndürür: servisinizin yayınladığı makine tarafından okunabilir sözleşmedir. Tıpkı Swagger gibi okuyun.
- SAP, `sap:*` annotasyon'ları ekler (label, creatable, updatable) ve Fiori bunları UI'ını otomatik oluşturmak için okur.
- Wire formatı: varsayılan olarak XML (Atom); `?$format=json` ile JSON. Koleksiyonlar `d.results` içindedir; tarihler `/Date(ms)/` biçimindedir; ondalıklar string olarak gelir.
- Her şeyi **`/IWFND/GW_CLIENT`**'ta test edin — Postman gibidir ama sistem içinde zaten mevcuttur.

---

*[← İçindekiler](../content.md) | [← Önceki: ALE & IDoc'lar](20-ale-idocs.md) | [Sonraki: Servis Metotları & Import Parametreleri →](22-odata-methods-and-parameters.md)*
