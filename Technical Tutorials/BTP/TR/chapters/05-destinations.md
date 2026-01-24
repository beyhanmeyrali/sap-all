# Kısım 5: Destination'lar – Bağlantılar İçin Telefon Rehberi

> *ABAP Geliştiricileri İçin En Önemli BTP Kavramı*

---

Bu kitaptan tek bir şey hatırlayacaksanız, destination'lar olsun. BTP'de oluşturduğunuz her Fiori uygulaması, her Joule skill'i, her entegrasyon destination kullanacak. Bulut dünyasında tüm bağlantının temeli bunlar.

---

## 5.1 Destination Nedir?

BTP'deki bir **Destination**, **adlandırılmış, merkezi olarak yönetilen bir bağlantı konfigürasyonudur**.

> **Telefon Rehberi Benzetmesi**
>
> Akıllı telefon kişilerinizi düşünün:
> - Her aradığınızda annenizin telefon numarasını yazmazsınız
> - Sadece "Anneyi Ara"ya dokunursunuz
> - Arka planda: +90 212 555 1234
>
> BTP'de, URL'leri, kullanıcı adlarını ve şifreleri her yere hardcode etmek yerine `MY_S4_SYSTEM`'e referans verirsiniz.

### Eski Yol vs. Yeni Yol

**Eski Yol (Programlarda hardcode):**
```abap
" Klasik ABAP - BUNU YAPMAYIN
DATA: lv_url TYPE string VALUE 'https://api.acme.com:443/orders'.
DATA: lv_user TYPE string VALUE 'API_USER'.
DATA: lv_pass TYPE string VALUE 'SecretPassword123!'.

" Artık kodda şifreler var - kabus!
```

**Yeni Yol (BTP Destination'ları):**
```javascript
// Uygulamanız sadece destination adına referans verir
const destination = "ACME_ORDER_API";
// BTP tüm karmaşık detayları halleder
```

### Destination'da Ne Saklanır

| Özellik | Açıklama | Örnek |
|---------|----------|-------|
| **Name** | Benzersiz tanımlayıcı | `ACME_S4_PROD_ORDERS` |
| **Type** | Protokol türü | HTTP, RFC, LDAP, MAIL |
| **URL** | Hedef endpoint | `https://s4.acme.com:443/sap/opu/odata/...` |
| **Proxy Type** | Nasıl ulaşılacağı | Internet, OnPremise, PrivateLink |
| **Authentication** | Nasıl kimlik doğrulanacağı | Basic, OAuth2, Certificate |
| **Credentials** | Gerçek gizli bilgiler | User/Password, Client ID/Secret |
| **Additional Properties** | Ekstra ayarlar | `sap-client=100`, timeout'lar |

---

## 5.2 Destination'lar Neden Var (Güvenlik & Yönetişim)

### Çözdükleri Problemler

| Problem | Destination'sız | Destination'lı |
|---------|-----------------|----------------|
| **Kodda şifre** | Güvenlik kabusu, denetim başarısızlığı | Kimlik bilgileri merkezi, şifreli saklanır |
| **URL değişiklikleri** | Tüm uygulamaları yeniden deploy et | Tek destination'ı güncelle |
| **Şifre rotasyonu** | Her yeri bul ve güncelle | Bir kez değiştir, uygulamalar otomatik kullanır |
| **Ortam yönetimi** | Hardcode dev/prod URL'leri | Subaccount başına farklı destination'lar |
| **Denetim izi** | Görünürlük yok | Kimin ne kullandığını, ne zaman gör |
| **Erişim kontrolü** | Herkes hardcode edebilir | Destination'ların erişim politikaları var |

### SM59 Karşılaştırması

ABAP geliştiricileri için, destination'lar kavramsal olarak SM59 RFC destination'larına benzer, ama HTTP/REST için:

| SM59 (Klasik ABAP) | BTP Destination'ları |
|---------------------|----------------------|
| RFC bağlantıları | HTTP, RFC, LDAP, MAIL |
| SAP sisteminde saklanır | BTP subaccount'ta saklanır |
| Transaction tabanlı config | Cockpit veya API config |
| Sınırlı auth seçenekleri | Modern auth (OAuth2, SAML, vb.) |

---

## 5.3 İlk Destination'ınızı Oluşturma

### BTP Cockpit'te Adım Adım

1. **Subaccount'unuza gidin**
   - BTP Cockpit → Global Account'unuz → Subaccount'unuz

2. **Connectivity → Destinations'a gidin**
   - Sol menü → Connectivity → Destinations

3. **"New Destination"a tıklayın**

4. **Temel özellikleri doldurun:**

```
Name:           ACME_S4_PROD_ORDERS
Type:           HTTP
Description:    S/4HANA Production - Satış Siparişleri API
URL:            https://s4-prod.acme.com:443/sap/opu/odata/sap/API_SALES_ORDER_SRV
Proxy Type:     Internet
Authentication: OAuth2ClientCredentials
```

5. **Kimlik doğrulama detaylarını ekleyin** (OAuth2 için):

```
Client ID:      your-client-id-from-s4
Client Secret:  your-client-secret
Token Service URL: https://s4-prod.acme.com/sap/bc/sec/oauth2/token
```

6. **Ek özellikler ekleyin** ("New Property"ye tıklayın):

```
sap-client:                 100
HTML5.DynamicDestination:   true
WebIDEEnabled:              true
WebIDEUsage:                odata_abap
```

7. **Kaydet ve "Check Connection" ile test et**

---

## 5.4 Kimlik Doğrulama Türleri Açıklandı

### 1. NoAuthentication

**Şunlar için kullan:** Sadece public API'ler (kurumsal ortamda nadir)

```
Authentication: NoAuthentication
```

**Örnek:** Public hava durumu API'si, test mock sunucu

**Uyarı:** İç API'ler için asla kullanmayın!

---

### 2. BasicAuthentication

**Şunlar için kullan:** Basit API'ler, legacy sistemler, geliştirme/test

```
Authentication: BasicAuthentication
User:           API_USER
Password:       ********
```

**Ne zaman kullanılmalı:**
- Test için hızlı kurulum
- OAuth desteklemeyen legacy sistemler
- Güvenilir ağlardaki iç API'ler

**Ne zaman KULLANILMAMALI:**
- İnternet'e açık prod API'leri
- Kullanıcı bağlamı propagasyonu gerektiğinde

---

### 3. OAuth2ClientCredentials

**Şunlar için kullan:** Modern sunucudan sunucuya iletişim (en yaygın!)

```
Authentication: OAuth2ClientCredentials
Client ID:      abc123-client-id
Client Secret:  ********
Token Service URL: https://target.com/oauth/token
```

**Nasıl çalışır:**
```
1. Uygulamanız Client ID + Secret'ı Token URL'ye gönderir
2. Bir access token alır (X dakika geçerli)
3. Token'ı gerçek API'yi çağırmak için kullanır
4. Token süresi dolar → otomatik yenisini alır
```

**Şunlar için önerilen kimlik doğrulama:**
- S/4HANA Cloud API'leri
- SuccessFactors API'leri
- Çoğu modern SAP servisi
- Harici API'leri çağıran Joule skill'leri

---

### 4. OAuth2SAMLBearerAssertion

**Şunlar için kullan:** Kullanıcı bağlamı propagasyonu ile SAP'tan SAP'a

```
Authentication: OAuth2SAMLBearerAssertion
Client ID:      abc123-client-id
Client Secret:  ********
Token Service URL: https://s4.acme.com/sap/bc/sec/oauth2/token
```

**ClientCredentials'tan farkı:**
- **ClientCredentials**: Teknik kullanıcı, gerçek kullanıcı bağlamı yok
- **SAMLBearer**: Oturum açmış kullanıcının kimliği backend'e akar

**Ne zaman kullanılmalı:**
- Kullanıcının backend'de spesifik yetkilere ihtiyacı var
- Denetim logları gerçek kullanıcıyı göstermeli
- Veri erişimi kullanıcı rollerine bağlı

---

### 5. PrincipalPropagation

**Şunlar için kullan:** Cloud Connector aracılığıyla on-premise'e kullanıcı kimliği iletme

```
Authentication: PrincipalPropagation
```

**Nasıl çalışır:**
```
BTP Kullanıcısı → Cloud Connector → On-Prem SAP
                  (Kullanıcı kimliği korunur)
```

---

### Kimlik Doğrulama Hızlı Referans

| Tür | Kullanıcı Bağlamı | En İyi Kullanım | Karmaşıklık |
|-----|-------------------|-----------------|-------------|
| NoAuthentication | Yok | Public API'ler | Basit |
| BasicAuthentication | Teknik | Legacy, Dev | Kolay |
| OAuth2ClientCredentials | Teknik | Sunucudan sunucuya | Orta |
| OAuth2SAMLBearerAssertion | Gerçek kullanıcı | Kullanıcıya özel veri | Orta |
| ClientCertificateAuthentication | Sertifika | Yüksek güvenlik | Karmaşık |
| PrincipalPropagation | Gerçek kullanıcı | SSO ile on-prem | Karmaşık |

---

## 5.5 Cloud Connector ile On-Premise Erişimi

Hedef sisteminiz **güvenlik duvarı arkasında** (internet'ten erişilemez) olduğunda, Cloud Connector'a ihtiyacınız var.

### On-Premise İçin Destination Kurulumu

```
Name:           ACME_S4_ONPREM_ORDERS
Type:           HTTP
URL:            http://s4-virtual:443/sap/opu/odata/sap/API_SALES_ORDER_SRV
Proxy Type:     OnPremise     ← Kilit fark!
Authentication: PrincipalPropagation  (veya BasicAuthentication)
Location ID:    (opsiyonel, birden fazla Cloud Connector varsa)
```

**Önemli notlar:**
- URL, Cloud Connector'da yapılandırılan **sanal host**'u kullanır
- Gerçek iç IP adresi **değil**
- Cloud Connector sanal → gerçek host eşleştirmesi yapar

---

## 5.6 Akıl Sağlığınızı Koruyan İsimlendirme Kuralları

Düzinelerce müşteriyle çalıştıktan sonra, güvenin: isimlendirme önemli.

### Kalıp

```
{MÜŞTERİ}_{SİSTEM}_{ORTAM}_{AMAÇ}
```

### Örnekler

```
# Prodüksiyon destination'ları
ACME_S4_PROD_ORDERS
ACME_S4_PROD_MATERIALS
ACME_SF_PROD_EMPLOYEES

# Geliştirme destination'ları
ACME_S4_DEV_ORDERS
ACME_S4_DEV_MATERIALS

# Test/QA destination'ları
ACME_S4_TEST_ORDERS

# Sandbox/mock destination'ları
SANDBOX_MOCK_ORDERS
```

### Bu Kalıp Neden Çalışır

1. **Birlikte sıralanır**: Tüm ACME destination'ları cockpit'te gruplanır
2. **Ortam görünür**: Dev uygulamasından yanlışlıkla prod çağırma yok
3. **Amaç net**: Bu destination'ın hangi API'ye eriştiğini bil
4. **Aranabilir**: Büyük subaccount'larda bulmak kolay

---

## 5.7 Fiori Uygulamalarında Destination'lar

### Fiori Uygulamaları Destination'ları Nasıl Kullanır

Fiori uygulamanızın `manifest.json`'ında:

```json
{
  "sap.app": {
    "dataSources": {
      "mainService": {
        "uri": "/sap/opu/odata/sap/API_SALES_ORDER_SRV/",
        "type": "OData",
        "settings": {
          "odataVersion": "2.0"
        }
      }
    }
  }
}
```

### xs-app.json Route

Sihir `xs-app.json`'da (Application Router config) olur:

```json
{
  "routes": [
    {
      "source": "^/sap/opu/odata/sap/(.*)$",
      "target": "/sap/opu/odata/sap/$1",
      "destination": "ACME_S4_PROD_ORDERS",
      "authenticationType": "xsuaa"
    }
  ]
}
```

---

## 5.8 Joule Skill'lerinde Destination'lar

Joule skill'leri için destination'lar **zorunludur**. URL'leri hardcode edemezsiniz.

### Akış

```
Joule Agent → Skill → Action (Action Project'ten) → Destination → API
```

### Joule İçin Destination Kurulumu

```
Name:           JOULE_ACME_S4_ORDERS
Type:           HTTP
URL:            https://s4.acme.com/sap/opu/odata/sap/API_SALES_ORDER_SRV
Authentication: OAuth2ClientCredentials
Client ID:      ...
Client Secret:  ...
Token URL:      ...

# Önerilen ek özellikler
HTML5.DynamicDestination: true
```

---

## 5.9 Yaygın Destination Kalıpları

### Kalıp 1: Aynı API, Birden Fazla Ortam

```
# Üç destination, aynı uygulama, farklı config'ler
ACME_S4_DEV_ORDERS   → https://s4-dev.acme.com/...
ACME_S4_TEST_ORDERS  → https://s4-test.acme.com/...
ACME_S4_PROD_ORDERS  → https://s4-prod.acme.com/...
```

Uygulama kodunuz aynı kalır. Sadece eşleşen destination'larla farklı subaccount'lara deploy edin.

### Kalıp 2: Birden Fazla Müşteri (Danışman)

```
# Subaccount: CLIENT_A_PROD
CLIENTA_S4_PROD_ORDERS  → https://s4.clienta.com/...

# Subaccount: CLIENT_B_PROD
CLIENTB_S4_PROD_ORDERS  → https://s4.clientb.com/...
```

---

## 5.10 Destination Sorun Giderme

### "Check Connection" Butonu

Her zaman kullanın! Şunları test eder:
- URL erişilebilirliği
- Kimlik doğrulama
- SSL/TLS sertifikası
- Cloud Connector bağlantısı (OnPremise için)

### Yaygın Hatalar ve Çözümler

| Hata | Neden | Çözüm |
|------|-------|-------|
| Connection refused | URL yanlış veya sistem kapalı | URL'yi doğrula, sistem durumunu kontrol et |
| 401 Unauthorized | Kötü kimlik bilgileri | Kullanıcı/şifre veya OAuth token'larını doğrula |
| 403 Forbidden | Kimlik doğrulandı ama erişim yok | Kullanıcı yetkilerini kontrol et |
| Certificate error | SSL sorunu | Trust store'a sertifika ekle veya TrustAll kullan (sadece dev) |
| Timeout | Yavaş hedef veya ağ | Timeout özelliğini artır |
| Host not found | DNS sorunu veya yanlış sanal host | Cloud Connector eşleştirmesini kontrol et |

---

## 5.11 Destination En İyi Uygulamaları

### Yapın

- ✅ Tutarlı kalıpla açıklayıcı isimler kullanın
- ✅ Description alanında amacı belgeleyin
- ✅ Prodüksiyon için OAuth2 kullanın
- ✅ Kullanmadan önce "Check Connection" ile test edin
- ✅ Uygun timeout'lar ayarlayın
- ✅ Ortam başına ayrı destination'lar kullanın
- ✅ Güvenlik denetimlerinde destination'ları gözden geçirin

### Yapmayın

- ❌ Uygulamalarda asla kimlik bilgilerini hardcode etmeyin
- ❌ İnternet'e açık prod API'leri için BasicAuth kullanmayın
- ❌ İlgisiz uygulamalar arasında destination paylaşmayın
- ❌ Kimlik bilgilerini periyodik olarak döndürmeyi unutmayın
- ❌ Prodüksiyonda TrustAll=true kullanmayın
- ❌ Cloud Connector'da gereksiz path'leri açmayın

---

## Temel Çıkarımlar

1. **Destination'lar temeldir** — Her BTP bağlantısı onları kullanır
2. **SM59 gibi ama daha iyi** — HTTP, modern auth, bulut-native
3. **Kimlik bilgileri güvende kalır** — Asla kodda, her zaman destination'da
4. **Ortam ayrımı** — dev/test/prod için farklı destination'lar
5. **İsimlendirme önemli** — `{MÜŞTERİ}_{SİSTEM}_{ORTAM}_{AMAÇ}` kullanın
6. **OAuth2 tercih edilir** — Prodüksiyon için her zaman
7. **Cloud Connector** — ProxyType=OnPremise ile on-premise erişimi için

---

## Sırada Ne Var?

Artık destination'ları—tüm backend'lerinize köprü—anlıyorsunuz. Şimdi BTP ABAP Environment ile bulutta ABAP kodu yazmayı görelim.

---

*[Önceki: Kısım 4 – RISE ve BTP Nasıl Birlikte Çalışır](04-rise-and-btp.md) | [Sonraki: Kısım 6 – Bulutta ABAP](06-abap-cloud.md)*

*[İçindekilere Dön](../content.md)*

---

**Yazar:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Hikaye Anlatıcısı & Dijital Dönüşüm Savunucusu

*Dünya genelindeki SAP öğrencileri için ❤️ ile oluşturuldu*
