# Kısım 19: RFC Web Servisleri — Consumer Proxy (SPROXY)

*ABAP'ın harici SOAP servislerini çağırması — "Add Service Reference" ama SAP lezzetinde.*

---

## ☕ Önce zihinsel model

Şimdiye kadar SAP'ın istemcilere veri *sunmasından* bahsettik. Ama trafik iki yönlü: ABAP programınızın çoğunlukla bir dış sistemi *çağırması* gerekir — bir ödeme ağ geçidi, bir tedarikçinin ERP'si, bir lojistik ortağının SOAP servisi, bir gümrük otoritesi API'si.

.NET'te projeye sağ tıklayıp "Add Service Reference" seçer, WSDL URL'sini yapıştırır ve Visual Studio tipli bir istemci sınıf üretirdi. Python'da WSDL'i çözümleyip istemci nesnesi almak için `zeep` kullanırdınız. ABAP'ta fikir aynıdır: **SPROXY** bir WSDL okur, yerel bir proxy sınıfı oluşturur ve siz metodlarını sanki sıradan bir ABAP sınıfı çağırır gibi çağırırsınız. Runtime, tüm SOAP zarf oluşturmayı, XML serileştirmeyi ve HTTP plümbingi arka planda halleder.

> 🎯 **Ne zaman kullanacaksınız:** ABAP'ın bir dış web servisini *çağırması* gerektiğinde — bankalar, vergi otoriteleri, lojistik sağlayıcılar ya da web servis olarak sunulan eski SAP sistemleriyle klasik SOAP/WSDL tabanlı entegrasyonlar. Bu giden entegrasyon desenidir.

---

## 19.1 Senaryo: ABAP harici web servis çağırıyor

### 1️⃣ Benzetme

Dış bir sistem WSDL'li bir SOAP web servisi yayımlar. WSDL bir sözleşmedir — "Bir `TrackingNumber` alıp `ShipmentStatus` yanıtı döndüren `GetShippingStatus` adlı bir işlemim var" der.

Bunu ABAP'tan çağırmak istiyorsunuz. İki seçeneğiniz var:
1. HTTP isteği ve XML zarfını elle oluşturmak — acı verici ve hataya açık.
2. WSDL'den **consumer proxy** üretmek — uzak servisi temsil eden yerel bir ABAP sınıfı. Sınıfı çağırırsınız; proxy plümbingi halleder.

SPROXY, 2. seçeneği üretir.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — "Add Service Reference" / WCF proxy (klasik)
// Visual Studio WSDL'i okur ve ShippingServiceClient'ı üretir

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
// C# modern — tipli HttpClient + Refit / NSwag üretilmiş istemci
// Aynı fikir — sözleşmeden (WSDL veya OpenAPI) tipli istemci üretiliyor
var client = new ShippingApiClient(httpClient);
var status = await client.GetShippingStatusAsync("TRK123456");
```

```python
# Python — zeep kütüphanesi (SOAP istemcisi, çalışma zamanında WSDL okur)
from zeep import Client

client = Client("https://shipping.partner.com/api?wsdl")
result = client.service.GetShippingStatus(
    TrackingNumber="TRK123456",
    Carrier="DHL"
)
print(f"Status: {result.Status}")
```

### 3️⃣ ABAP'taki karşılığı

ABAP'ta SPROXY, Visual Studio'nun "Add Service Reference" işleminin yaptığını yapar: WSDL'i okur, proxy sınıfları üretir. SOAMANAGER, endpoint URL'sini ve kimlik doğrulamayı yapılandırdığınız yerdir — WCF istemciniz için `appsettings.json` gibi ya da .NET'teki HttpClient taban adresi gibi.

---

## 19.2 SPROXY — consumer proxy üretimi

### SPROXY ne yapar

**T-kodu:** `SPROXY` (arka uç sistemde)

1. SPROXY'ye bir WSDL URL'si verirsiniz veya bir WSDL dosyası yüklersiniz.
2. SPROXY WSDL'i çözümler ve şunları üretir:
   - Bir **interface** (taslak: WSDL işlemleriyle eşleşen metot imzaları)
   - Bir **proxy sınıfı** (uygulama: HTTP, SOAP zarfları, XML serileştirmeyi halleder)
   - WSDL'de tanımlı tüm istek/yanıt yapıları ve karmaşık tipler için **DDIC tipleri**

3. Tamamen tipli bir ABAP proxy sınıfı elde edersiniz — iş kodunuzda XML çözümleme yoktur.

### Üretilen yapılar

Tek `ShipmentStatus_Query` işlemli bir WSDL için SPROXY şunları üretebilir:

```
Paket: ZSHIPPING_PROXY

Interface:       CO_ZSHIPPING_PROXY (port type interface)
Proxy sınıfı:    CL_ZSHIPPING_PROXY (interface'i implemente eder)
İstek tipi:      ZSHIPPING_STATUS_QUERY_REQUEST   (DDIC yapısı)
Yanıt tipi:      ZSHIPPING_STATUS_QUERY_RESPONSE  (DDIC yapısı)
Hata tipi:       ZSHIPPING_FAULT_TYPE             (DDIC yapısı)
```

> 🛠️ **SPROXY adım adım:**
> 1. SAP GUI'de `SPROXY` açın.
> 2. **Create** → "Service Consumer" seçin → WSDL URL'sini yapıştırın veya WSDL dosyası içe aktarın.
> 3. Proxy adı ve hedef paketi girin.
> 4. SPROXY tüm nesneleri üretir ve etkinleştirir.
> 5. Üretilen interface ve sınıfı inceleyin — işlem metodlarını ve DDIC tiplerini göreceksiniz.

> ⚠️ **C#/Python tuzağı:** SPROXY klasik WS-* / document-literal SOAP ile iyi çalışır. Dış servis standart dışı WS-Security uzantıları veya egzotik zarf yapılarıyla SOAP kullanıyorsa üretilen proxy ince ayar gerektirebilir. SPROXY tüm WSDL lehçelerini mükemmel işlemez — özellikle çok eski RPC-encoded WSDL stilleri için. Bunlarda manuel XML ile `CL_HTTP_CLIENT`'a geri dönmeniz gerekebilir (bölüm 19.5).

---

## 19.3 Logical port'lar ve SOAMANAGER

Üretilen proxy sınıfı endpoint URL'sini doğrudan kodlamaz. Bu, **SOAMANAGER** (t-kodu `SOAMANAGER`) içinde yapılandırılan bir **logical port**'ta yaşar. Bunu `"ShippingServiceUrl": "https://..."` tutan `appsettings.json` gibi düşünün — her ortam için (DEV, QAS, PRD) bir kez yapılandırılır, kod hiç değişmez.

### 1️⃣ Benzetme

```csharp
// C# — adlandırılmış istemci + endpoint yapılandırması için IOptions ile HttpClient
// Program.cs:
builder.Services.AddHttpClient("ShippingAPI", client =>
{
    client.BaseAddress = new Uri(config["ShippingService:Url"]);
});
// appsettings.json:
// "ShippingService": { "Url": "https://shipping.partner.com/api" }

// İstemci kodu URL'yi bilmez — DI/config'den alır
var client = httpClientFactory.CreateClient("ShippingAPI");
```

SOAMANAGER ABAP karşılığıdır: endpoint URL'sini, kimlik doğrulamayı ve zaman aşımlarını bir logical port başına bir kez ayarlarsınız. Proxy kodu sadece port adına başvurur.

### SOAMANAGER'da logical port yapılandırma

**T-kodu:** `SOAMANAGER`

1. `SOAMANAGER` açın → **Web Service Administration → Manage Business System**.
2. Proxy sınıfınızı bulun (örn. `CL_ZSHIPPING_PROXY`).
3. **Create Logical Port** tıklayın.
4. Şunları girin:
   - **Port adı:** örn. `ZSHIPPING_DEFAULT`
   - **Açıklama:** Kargo ortağı — varsayılan
   - **Is Default:** Evet (proxy, başka bir belirtilmedikçe bu portu kullanır)
   - **WSDL URL veya Endpoint URL:** `https://shipping.partner.com/soap/v2`
5. **Authentication** sekmesinde: Basic Auth veya SSL istemci sertifikası seçin, kimlik bilgilerini girin.
6. Kaydedin ve etkinleştirin.

> 🧭 **İş hayatında:** Logical port'lar ortama özgü yapılandırmadır. DEV, QAS ve PRD'nin farklı endpoint'leri ve kimlik bilgileri olacaktır. SAP Basis bunları production'da yönetir. Sizin işiniz, ABAP kodunda doğru port adını belirtmek ve transport isteği notlarında servisin hangi logical port'lara ihtiyaç duyduğunu belgelemektir.

---

## 19.4 Proxy metodunu ABAP'tan çağırma

### 3️⃣ ABAP'taki karşılığı — eksiksiz bir proxy çağrısı örneği

```abap
"! Üretilen SPROXY consumer proxy aracılığıyla harici kargo web servisi çağırır.
"! İstek oluşturmayı, yanıt işlemeyi ve hata yakalamayı gösterir.
METHOD check_shipment_status.

  " ── 1. Proxy sınıfını örnekle ────────────────────────────────────
  DATA lo_proxy   TYPE REF TO cl_zshipping_proxy.
  DATA ls_request TYPE zshipping_status_query_request.
  DATA ls_response TYPE zshipping_status_query_response.

  TRY.
      " Constructor, opsiyonel logical port adı alır.
      " Atlanırsa SOAMANAGER'daki varsayılan port kullanılır.
      CREATE OBJECT lo_proxy
        EXPORTING
          logical_port_name = 'ZSHIPPING_DEFAULT'.

    CATCH cx_ai_system_fault INTO DATA(lx_create).
      " Endpoint'e bağlanılamadı — SOAMANAGER'da yapılandırma sorunu
      RAISE EXCEPTION TYPE zcx_shipping_error
        EXPORTING
          message = lx_create->get_text( ).
  ENDTRY.

  " ── 2. İstek yapısını doldur (üretilmiş DDIC tipi) ──────────────
  ls_request-tracking_number = iv_tracking_number.
  ls_request-carrier_code    = iv_carrier.

  " ── 3. Proxy metodunu çağır ─────────────────────────────────────
  TRY.
      lo_proxy->shipment_status_query(
        EXPORTING
          input  = ls_request
        IMPORTING
          output = ls_response
      ).

      " ── 4. Yanıtı oku ───────────────────────────────────────────
      es_status-status      = ls_response-status_code.
      es_status-description = ls_response-status_text.
      es_status-eta         = ls_response-estimated_delivery_date.

    CATCH cx_ai_application_fault INTO DATA(lx_app_fault).
      " ── 5a. Uygulama düzeyinde hata — servis, WSDL'de tanımlı
      "        tipli hata mesajıyla bir SOAP Fault döndürdü
      DATA ls_fault TYPE zshipping_fault_type.
      lx_app_fault->get_fault_detail( IMPORTING fault_detail = ls_fault ).
      RAISE EXCEPTION TYPE zcx_shipping_error
        EXPORTING
          message     = ls_fault-error_text
          error_code  = ls_fault-error_code.

    CATCH cx_ai_system_fault INTO DATA(lx_sys_fault).
      " ── 5b. Sistem düzeyinde hata — HTTP hatası, zaman aşımı, SSL hatası vb.
      RAISE EXCEPTION TYPE zcx_shipping_error
        EXPORTING
          message = lx_sys_fault->get_text( ).
  ENDTRY.

ENDMETHOD.
```

### İki istisna sınıfı

| İstisna | Ne zaman | C# karşılığı |
|---------|---------|--------------|
| `CX_AI_SYSTEM_FAULT` | HTTP hatası, zaman aşımı, SSL hatası, bağlantı reddedildi, endpoint ulaşılamaz | `HttpRequestException`, `TaskCanceledException` (zaman aşımı) |
| `CX_AI_APPLICATION_FAULT` | Servis bir SOAP Fault öğesi döndürdü (uzak taraf isteği işledi ama bir iş hatası döndürdü) | `FaultException<TDetail>` (WCF) |

> ⚠️ **C#/Python tuzağı:** İstisna ayrımı önemlidir. `CX_AI_SYSTEM_FAULT` "servisten hiç uygun yanıt alamadık" demektir. `CX_AI_APPLICATION_FAULT` "servis yanıtladı ama hata mesajıyla" demektir. Kurtarma stratejileri farklı olduğu için bunları ayrı ayrı işleyin: sistem hatası yeniden denemeyi gerektirebilir; uygulama hatası genellikle veri/iş mantığı sorunudur.

---

## 19.5 Proxy olmadan doğrudan REST/JSON çağırma

SPROXY SOAP/WSDL içindir. Dış servis REST/JSON ise (WSDL yok), modern ABAP'ta iki seçeneğiniz vardır:

### Seçenek A: `CL_HTTP_CLIENT` (klasik)

```abap
" Klasik HTTP istemcisi — her yerde çalışır (eski sistemlerde de)
DATA lo_http    TYPE REF TO if_http_client.
DATA lo_request TYPE REF TO if_http_request.

" HTTP istemcisi oluştur
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
  " hatayı işle
ENDIF.

" Başlıkları ayarla
lo_http->request->set_method( if_http_request=>co_request_method_get ).
lo_http->request->set_header_field(
  name  = 'Authorization'
  value = |Bearer { lv_token }|
).
lo_http->request->set_header_field(
  name  = 'Accept'
  value = 'application/json'
).

" Gönder
lo_http->send( ).
lo_http->receive( ).

" Yanıtı oku
DATA lv_json   TYPE string.
DATA lv_status TYPE i.
lv_json   = lo_http->response->get_cdata( ).
lv_status = lo_http->response->get_status_code( ).

lo_http->close( ).

" JSON çözümle — SAP JSON kütüphanesinden sınıf kullanarak
DATA lo_json TYPE REF TO cl_trex_json_deserializer.
" ... veya daha basit bir interface için /ui2/cl_json kullanın
```

### Seçenek B: `IF_WEB_HTTP_CLIENT` (modern, ABAP 7.54+)

```abap
" Modern HTTP istemcisi — daha temiz API, S/4HANA 2020+ sürümlerinde mevcut
DATA lo_dest   TYPE REF TO if_http_destination.
DATA lo_client TYPE REF TO if_web_http_client.

lo_dest = cl_http_destination_provider=>create_by_url(
              'https://api.partner.com' ).

lo_client = cl_web_http_client_manager=>create_by_http_destination(
                lo_dest ).

" İstek oluştur
DATA lo_req TYPE REF TO if_web_http_request.
lo_req = lo_client->get_http_request( ).
lo_req->set_header_field( i_name  = 'Authorization'
                          i_value = |Bearer { lv_token }| ).

" GET çalıştır
DATA lo_resp TYPE REF TO if_web_http_response.
lo_resp = lo_client->execute( if_web_http_client=>get ).

" JSON yanıtı oku
DATA lv_body TYPE string.
lv_body = lo_resp->get_text( ).

" /ui2/cl_json veya sxml deserializer ile çözümle
DATA ls_result TYPE zty_shipment_status.
/ui2/cl_json=>deserialize(
  EXPORTING json = lv_body
  CHANGING  data = ls_result
).
```

> 💡 **Ne zaman neyi kullanmalı:**
> - Dış servisin WSDL'i var → **SPROXY** (tip güvenli, üretilmiş, bakımı kolay).
> - Dış servis REST/JSON, modern sistem → **IF_WEB_HTTP_CLIENT**.
> - Dış servis REST/JSON, eski sistem → **CL_HTTP_CLIENT**.
> - Dış servis başka bir SAP sistemi → **RFC / BAPI** (kısım 12) veya web servis olarak sunulmuşsa **SPROXY**.

---

## 🧠 Özet

| Kavram | C#/Python karşılığı | ABAP |
|--------|---------------------|------|
| "Add Service Reference" / WSDL import | WCF Add Service Reference / NSwag | `SPROXY` |
| Üretilmiş tipli istemci sınıfı | `ShippingServiceClient` (WCF) | `CL_ZSHIPPING_PROXY` (üretilmiş) |
| Endpoint URL & kimlik doğrulama yapılandırması | `appsettings.json` + HttpClient yapılandırması | `SOAMANAGER`'da logical port |
| Web servis işlemi çağırma | `await client.GetShippingStatusAsync(req)` | `lo_proxy->shipment_status_query(...)` |
| HTTP/transport hatası | `HttpRequestException` | `CX_AI_SYSTEM_FAULT` |
| Servis düzeyinde iş hatası | `FaultException<TDetail>` | `CX_AI_APPLICATION_FAULT` |
| WSDL olmadan REST/JSON | `HttpClient` / `requests` | `IF_WEB_HTTP_CLIENT` / `CL_HTTP_CLIENT` |

**Hatırlanacak beş şey:**
1. SPROXY, WSDL'den tipli bir ABAP proxy sınıfı üretir — "Add Service Reference" ile aynı kavram.
2. SOAMANAGER, ortam başına endpoint URL'lerini ve kimlik bilgilerini tutar — asla doğrudan koda gömmeyin.
3. Her zaman hem `CX_AI_SYSTEM_FAULT` (transport hatası) hem `CX_AI_APPLICATION_FAULT` (SOAP Fault) için ayrı ayrı yakalayın.
4. SPROXY SOAP/WSDL içindir; REST/JSON için modern sistemlerde `IF_WEB_HTTP_CLIENT`, klasik sistemlerde `CL_HTTP_CLIENT` kullanın.
5. Proxy sınıfı üretilmiştir — elle değiştirmeyin. WSDL değişirse SPROXY'de yeniden üretin.

---

*[← İçindekiler](../content.md) | [← Önceki: OData & REST: Büyük Resim](18-odata-and-rest-intro.md) | [Sonraki: ALE & IDoc'lar →](20-ale-idocs.md)*
