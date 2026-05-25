# Kısım 32: Google Form → SAP Entegrasyonu

*Ücretsiz bir Google Form'u gerçek bir SAP veri giriş hattına dönüştürmek — ve özgeçmişini öne çıkarmak.*

---

## ☕ Bu kısım neden var

Portföy projeleri, "ABAP hakkında okudum" ile "bir şeyler inşa ettim" arasındaki farkı ortaya koyar. Google Form'un veriyi SAP'a beslediği bir proje tam da bir işe alım yöneticisinin yüzlerce kez görmediği türden bir şeydir. Her iki dünyayı da anladığını, HTTP üzerinden birbirine bağlayabildiğini ve bunu yapmak için altı haneli bir middleware aracına ihtiyaç duymadığını gösterir.

İşte ne inşa ediyoruz: Diyelim ki bir **servis talebi** (teknisyen adı, sorun açıklaması, öncelik) yakalayan bir Google Form. Biri formu gönderdiği anda küçük bir Google Apps Script devreye giriyor, veriyi JSON olarak paketliyor ve SAP'ta daha önce inşa ettiğin **inbound OData veya REST servisine** POST ediyor. SAP bunu bir Z-tablosuna kaydediyor. Bu kadar. Bu, dünya genelinde küçük ve orta ölçekli SAP mağazalarında kullanılan gerçek bir entegrasyon modelidir.

---

## 32.1 Senaryo

Saha teknisyenlerinin telefonlarından servis talebi göndermesini isteyen bir şirkette çalıştığını düşün — ancak şirketin SAP bilet sistemi mobil uyumlu değil (şaşırtıcı, biliyorum). Ekip karar veriyor: Google Forms kullan (ücretsiz, her telefonda çalışır, uygulama kurulumu gerekmez) ve SAP ile entegre et.

Gereksinimler:
- Teknisyen bir Google Form dolduruyor.
- Gönderimde veri, SAP'ta `ZTECHSVC_REQ` (Z-tablonuz) içine kayıt olarak giriyor.
- SAP ekibi kayıtları bir raporda veya Fiori uygulamasında görebiliyor.

Bu bir **push entegrasyonu**: Google, SAP'a iter. SAP yoklamaz. Bu, SAP yükünü minimal tutar ve gecikmeyi düşük kılar.

---

## 32.2 Entegrasyon Akışı

Bir satır kod yazmadan önce her şeyi haritalayalım.

```mermaid
sequenceDiagram
    participant T as Teknisyen (Tarayıcı)
    participant GF as Google Form
    participant GAS as Google Apps Script<br/>(onSubmit tetikleyicisi)
    participant SAP as SAP ICF Uç Noktası<br/>(REST/OData)
    participant DB as SAP Z-Tablosu

    T->>GF: Form gönder
    GF->>GAS: onFormSubmit tetikleyicisi ateşleniyor
    GAS->>GAS: JSON yükü oluştur
    GAS->>SAP: HTTP POST /sap/opu/odata/sap/ZSR_SRV/ServiceRequestSet<br/>(Authorization: Basic ...)
    SAP->>SAP: Kimlik doğrula, gövdeyi ayrıştır
    SAP->>DB: ZTECHSVC_REQ'e INSERT
    SAP-->>GAS: 201 Created + yanıt JSON
    GAS-->>GAS: Başarıyı logla / hata durumunda uyar
```

Yığın çarpıcı biçimde sade:

| Katman | Teknoloji | Senin analojin |
|---|---|---|
| Form UI | Google Forms | Google tarafından barındırılan bir HTML `<form>` |
| Yapıştırıcı mantık | Google Apps Script | Küçük bir Node.js tarzı JavaScript çalışma zamanı, yine Google tarafından barındırılıyor — ücretsiz |
| Taşıma | HTTPS POST | Düz REST, herhangi bir API'yi çağırdığın gibi |
| Gelen işleyici | SAP ICF / OData servisi | ASP.NET Web API `[HttpPost]` denetleyicin |
| Kalıcılık | ABAP + Z-tablosu | EF `DbContext.SaveChanges()` |

> 🧭 **İş hayatında:** Bu model aynı zamanda Microsoft Forms (Power Automate aracılığıyla), Typeform (webhook'lar aracılığıyla) veya gönderimde bir URL çağırabilen herhangi bir form aracı için de çalışır. SAP tarafını inşa ettikten sonra form aracını değiştirmek önemsiz bir iştir.

---

## 32.3 Google Tarafı — Apps Script

### Form ve elektronik tabloyu oluştur

1. [forms.google.com](https://forms.google.com) adresine git, şu alanlarla bir form oluştur: **TechnicianName** (Kısa yanıt), **CustomerName** (Kısa yanıt), **Issue** (Paragraf), **Priority** (Çoktan seçmeli: Low / Medium / High).
2. Yeşil Sheets simgesine tıkla (Yanıtlar → Sheets'e bağla). Bu, bağlantılı bir elektronik tablo oluşturur — Apps Script burada yaşar.
3. Elektronik tabloda: **Uzantılar → Apps Script**.

### onSubmit tetikleyici betiği

```javascript
// Google Apps Script — Code.gs
// Bağlı form her gönderildiğinde çalışır.

const SAP_ENDPOINT = "https://your-sap-host.example.com/sap/opu/odata/sap/ZSR_SRV/ServiceRequestSet";
const SAP_USER     = "APIUSER";          // SAP'ta ayrılmış RFC/API kullanıcısı
const SAP_PASS     = "YourPassword123";  // Canlıda Script Properties'te sakla!

/**
 * Tetikleyici: Form → onSubmit
 * e.namedValues bize { "Alan Etiketi": ["değer"], ... } verir
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
      "x-csrf-token":  fetchCsrfToken()   // SAP OData, değiştirici çağrılarda CSRF token gerektirir
    },
    payload:     JSON.stringify(payload),
    muteHttpExceptions: true  // hata gövdesini okuyabilmek için
  };

  try {
    const response = UrlFetchApp.fetch(SAP_ENDPOINT, options);
    const code     = response.getResponseCode();

    if (code >= 200 && code < 300) {
      Logger.log("SAP kaydı oluşturuldu: " + response.getContentText());
    } else {
      Logger.log("SAP hatası " + code + ": " + response.getContentText());
      // Canlıda: bir yöneticiye uyarı e-postası gönder
      MailApp.sendEmail("admin@example.com", "SAP Form Entegrasyon Hatası",
        "HTTP " + code + "\n" + response.getContentText());
    }
  } catch (err) {
    Logger.log("SAP çağrısında istisna: " + err.toString());
  }
}

/**
 * OData servisleri POST/PUT/DELETE'i CSRF token ile korur.
 * x-csrf-token: Fetch içeren bir HEAD/GET ile al.
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

### Tetikleyiciyi bağla

Apps Script'te: **Tetikleyiciler → Tetikleyici Ekle → onFormSubmit → Formdan → Form gönderiminde**. Kaydet. Formu göndererek ve Execution log'unu kontrol ederek test et.

> ⚠️ **C#/Python tuzağı:** `e.namedValues` değerleri alan bir kez görünse bile her zaman **dizi** şeklindedir — `vals["Alan"][0]`, `vals["Alan"]` değil. Bunu unutmak SAP'ta `"undefined"` dizileri almanı sağlar.

> 💡 **Güvenlik ipucu:** Apps Script kaynak koduna asla parola gömmeni. **Dosya → Proje Özellikleri → Script Properties** (anahtar-değer deposu) kullan ve `PropertiesService.getScriptProperties().getProperty("SAP_PASS")` ile oku. Script kaynağı, elektronik tabloda düzenleyici erişimine sahip herkes tarafından görülebilir.

---

## 32.4 SAP Tarafı — Gelen Servis

Gelen uç nokta için iki gerçekçi seçeneğin var:

| Seçenek | Ne zaman kullanılır | Notlar |
|---|---|---|
| **OData servisi (SEGW)** | Veri, Fiori/UI5 aracılığıyla da okuyacağın bir varlıksa | Daha fazla kurulum, ama `$metadata`, Fiori-hazır, filtre/genişletme bedava gelir |
| **ICF işleyici (IF_HTTP_EXTENSION)** | Hızlı REST uç noktası, OData yükü olmadan | Daha az altyapı, saf push entegrasyonları için mükemmel |

Her ikisini de yapacağız — önce OData (çünkü SEGW'yi Bölüm VI'dan biliyorsun), sonra ICF yaklaşımı.

### Seçenek A: OData CREATE_ENTITY (SEGW)

Bunu Kısım 25'te inşa ettin. Bu entegrasyonun SAP tarafı, `ServiceRequest` varlık tipinde standart bir `CREATE_ENTITY`'dir. Tek fark Fiori uygulaması yerine Google'dan JSON alıyor olman.

```abap
" ZSR_DPC_EXT — 'ServiceRequest' varlık tipi için CREATE_ENTITY
METHOD serviceRequestset_create_entity.

  DATA: ls_req  TYPE ztechsvc_req,  " Varlıkla eşleşen Z-yapın
        ls_data TYPE zcl_zsr_mpc=>ts_servicerequest.

  " 1. JSON gövdesini varlık yapısına seri dışı bırak
  io_data_provider->read_entry_data( IMPORTING es_data = ls_data ).

  " 2. DB yapısına eşle
  ls_req-guid          = cl_system_uuid=>create_uuid_c32_static( ).
  ls_req-tech_name     = ls_data-technicianname.
  ls_req-cust_name     = ls_data-customername.
  ls_req-issue         = ls_data-issue.
  ls_req-priority      = ls_data-priority.
  ls_req-submit_ts     = ls_data-submittimestamp.
  ls_req-created_by    = sy-uname.
  ls_req-created_at    = sy-datum.

  " 3. Ekle — yinelenen kayıtta ABAP istisna fırlatsın
  INSERT ztechsvc_req FROM ls_req.
  IF sy-subrc <> 0.
    RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
      EXPORTING
        textid  = /iwbep/cx_mgw_busi_exception=>business_error
        message = 'Ekleme başarısız — yinelenen anahtar?'.
  ENDIF.

  " 4. Oluşturulan varlığı OData çerçevesine geri ver
  er_entity = ls_data.

ENDMETHOD.
```

### Seçenek B: ICF REST işleyici (IF_HTTP_EXTENSION)

Bazen OData törenine gerek duymadan düz bir HTTP uç noktası isteyebilirsin. `IF_HTTP_EXTENSION` uygulayan `ZCL_HTTP_SR_HANDLER` adında bir sınıf oluştur, SICF'te bir düğüm altına kaydet (örn. `/sap/zgform/sr`) ve yayınla.

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

    " ── 1. Yalnızca POST kabul et ───────────────────────────────────────
    IF server->request->get_method( ) <> 'POST'.
      server->response->set_status( code = 405 reason = 'Method Not Allowed' ).
      RETURN.
    ENDIF.

    " ── 2. Ham JSON gövdesini oku ────────────────────────────────────────
    lv_body = server->request->get_cdata( ).

    " ── 3. JSON → ABAP yapısına ayrıştır ─────────────────────────────────
    " CL_SXML_STRING_READER + dönüşüm veya daha sade
    " /UI2/CL_JSON (SAP_BASIS 7.40+ ile kullanılabilir) kullanıyoruz.
    /ui2/cl_json=>deserialize(
      EXPORTING json = lv_body
      CHANGING  data = ls_req ).

    " ── 4. Sistem alanlarını damgala ─────────────────────────────────────
    ls_req-guid       = cl_system_uuid=>create_uuid_c32_static( ).
    ls_req-created_by = server->request->get_header_field( 'x-sap-user' ).
    ls_req-created_at = sy-datum.

    " ── 5. Kaydet ────────────────────────────────────────────────────────
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

> ⚠️ **C#/Python tuzağı:** SAP'ın ICF işleyicisinin `handle_request` metodunun **dönüş tipi yok** — başarıyı/başarısızlığı tamamen `server` nesnesi üzerinde yanıt durumunu ve gövdeyi ayarlayarak iletiyorsun. `set_status` çağrısını unutmak, hiçbir şey yapmamış olsan bile istemcinin kafa karıştırıcı bir 200 almasına neden olur.

### Z-tablosu

```abap
" SE11 → Veritabanı Tablosu → ZTECHSVC_REQ
" Anahtar alan: GUID (CHAR 32, UUID)
" Diğer alanlar:
"   TECH_NAME  CHAR 80   (Teknisyen adı)
"   CUST_NAME  CHAR 80   (Müşteri adı)
"   ISSUE      STRING    (Sorun açıklaması)
"   PRIORITY   CHAR 6    (LOW / MEDIUM / HIGH)
"   SUBMIT_TS  CHAR 27   (Google'dan ISO zaman damgası)
"   CREATED_BY UNAME     (sy-uname)
"   CREATED_AT DATS      (sy-datum)
"   STATUS     CHAR 10   (OPEN/INPROG/CLOSED — varsayılan OPEN)
```

> 🧭 **İş hayatında:** Gerçek bir projede `ISSUE` alanının STRING olması bazı eski ABAP araçlarında sorun yaratır (ABAP string'leri yığın tahsislidir, satır içi değil). Uzun metin için `CHAR 1333` veya metin tablosunu tercih et. Bir DDIC tablosunda STRING seçmeden önce kıdemlinden görüş al.

---

## 32.5 Uçtan Uca Test + Güvenlik Notları

### Google'ı bağlamadan önce manuel test

Favori REST istemcini kullan (Postman, Insomnia veya VS Code REST uzantısı):

```http
POST https://your-sap-host/sap/opu/odata/sap/ZSR_SRV/ServiceRequestSet
Authorization: Basic QVBJVVNFUJI6UGFzc3dvcmQxMjM=
Content-Type: application/json
x-csrf-token: <metadata-çağrısından-alınan-token>

{
  "TechnicianName": "Jane Smith",
  "CustomerName": "Acme Corp",
  "Issue": "2. kattaki yazıcı çevrimdışı",
  "Priority": "MEDIUM",
  "SubmitTimestamp": "2025-10-01T09:30:00Z"
}
```

Beklenen: `HTTP 201 Created` + yanıt gövdesinde yeni varlık.

### Uçtan uca test

1. Google Form'u tarayıcıda aç.
2. Test verisi gir ve gönder.
3. Apps Script tetikleyicisinin ateşlenmesi için ~5 saniye bekle.
4. SAP'ta: `SE16N → ZTECHSVC_REQ` — kaydın orada olmalı.
5. Görünmüyorsa Apps Script **Execution log**'unu (Görüntüle → Yürütmeler) kontrol et.

### Güvenlik kontrol listesi

| Endişe | Öneri |
|---|---|
| **Apps Script'teki kimlik bilgileri** | Script Properties kullan, asla sabit kodlama |
| **SAP API kullanıcısı** | Yalnızca `SRV_OData` için minimal yetkilendirmelerle ayrılmış bir kullanıcı (tür `S` veya `B`) oluştur; diyalog oturumu açma |
| **Ağ maruziyeti** | SAP sisteminizi internete doğrudan port 443 üzerinden açmak yerine SAP Cloud Connector + BTP bağlantısını tercih et |
| **CSRF token** | OData V2, POST/PUT/DELETE'de gerektirir — her zaman önce `x-csrf-token: Fetch` ile al |
| **Girdi doğrulama** | Eklemeden önce `Priority`'nin LOW/MEDIUM/HIGH'dan biri olduğunu doğrula; aksi takdirde ABAP formun gönderdiği her şeyi ekler |
| **Yalnızca HTTPS** | Düz HTTP üzerinden Basic Auth asla gönderme — sadece Base64, şifreleme değil |

> 💡 Üretime hazır kurulumlar için Basic Auth'u SAP BTP'nin API Management'ı veya servis anahtarı aracılığıyla **OAuth 2.0 Client Credentials** ile değiştir. Ancak ayrılmış bir API kullanıcısıyla Basic Auth, dahili araçlar için gayet kabul edilebilir.

---

## 🧠 Özet

- Google Apps Script'in `onFormSubmit` tetikleyicisi, ücretsiz, barındırılan webhook yapıştırıcı katmanındır.
- CSRF token dansı (`x-csrf-token: Fetch` → dönen değeri POST'ta kullan) SAP OData V2'ye özgüdür — her zaman unutma.
- SAP tarafı ya bir SEGW `CREATE_ENTITY` (OData, Fiori-hazır) ya da bir ICF `IF_HTTP_EXTENSION` işleyicisidir (hafif REST).
- Ayrılmış API kullanıcısı, kimlik bilgileri için Script Properties kullan ve internet maruziyeti için SAP Cloud Connector'ı tercih et.
- Bu proje — Google Forms → SAP — gerçek bir CV farklılaştırıcısıdır. GitHub'a bir README ve ekran görüntüsüyle ekle.

---

*[← İçindekiler](../content.md) | [← Önceki: OData'da Dosya Yükleme & İndirme](31-odata-file-upload-download.md) | [Sonraki: ABAP'tan WhatsApp Entegrasyonu →](33-whatsapp-integration.md)*
