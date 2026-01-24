# Kısım 12: BTP'de Çoklu Müşteri Yönetimi

> *Danışmanın Hayatta Kalma Rehberi*

---

Eğer bir danışman, iş ortağı veya paylaşılan hizmetler ekibinde çalışıyorsanız, birden fazla müşteri için BTP'yi yöneteceksiniz. Bu bölüm, aklınızı kaybetmeden bunu nasıl yapacağınızı gösterir.

---

## 12.1 Klasik Tuzak: Her Şey Tek Subaccount'ta

### İnsanların Yanlış Yaptığı Şey

```mermaid
graph TD
    subgraph "❌ Anti-Pattern: Her Şey Bir Arada"
        GA[Global Account]
        GA --> SA[Tek Subaccount]
        SA --> A1[Müşteri A'nın Joule Agent'ı]
        SA --> A2[Müşteri B'nin Destination'ları]
        SA --> A3[Müşteri C'nin Test Uygulaması]
        SA --> A4[Sizin Oyun Alanınız]
        SA --> A5[2023'ten POC]
        SA --> A6[Bu ne kim bilir?]
    end

    style SA fill:#f44336,color:white
```

### Bunun Yarattığı Sorunlar

| Sorun | Etkisi |
|-------|--------|
| **Yetkilendirme kaos** | Kimin hangi kotayı kullandığı anlaşılamıyor |
| **Güvenlik kabusu** | Müşteri A, Müşteri B'nin destination'larını görebiliyor |
| **İsim çakışmaları** | `S4_ORDERS` — kimin S4'ü? |
| **Faturalandırma imkansız** | Müşteri başına maliyet aktarılamıyor |
| **Devir sorunları** | "İşte her şeye erişim" |
| **Temizlik felci** | Bir şeyi silmekten korkuluyor |

---

## 12.2 En İyi Uygulama: Müşteri Başına Bir Subaccount

### Önerilen Yapı

```mermaid
graph TD
    subgraph "Global Account: Partner Account"
        DIR1[Directory: EMEA Müşterileri]
        DIR2[Directory: APAC Müşterileri]
        DIR3[Directory: Dahili]

        subgraph "EMEA"
            SA1[ACME_PROD]
            SA2[ACME_DEV]
            SA3[GLOBEX_PROD]
            SA4[GLOBEX_DEV]
        end

        subgraph "APAC"
            SA5[TOSHIBA_PROD]
            SA6[TOSHIBA_DEV]
        end

        subgraph "Dahili"
            SA7[TEMPLATES]
            SA8[DEMOS]
            SA9[TRAINING]
        end
    end

    DIR1 --> EMEA
    DIR2 --> APAC
    DIR3 --> Dahili

    style SA1 fill:#4CAF50,color:white
    style SA3 fill:#4CAF50,color:white
    style SA5 fill:#4CAF50,color:white
```

### Faydaları

| Fayda | Açıklama |
|-------|----------|
| **İzolasyon** | Müşteriler birbirlerinin verilerini göremez |
| **Net faturalandırma** | Kullanım subaccount başına takip edilir |
| **Kolay devir** | Tüm subaccount müşteriye aktarılabilir |
| **Temiz isimlendirme** | Subaccount içinde önek gerekmiyor |
| **Daha basit güvenlik** | Subaccount başına rol atama |

---

## 12.3 Ölçeklenen İsimlendirme Kuralları

### Subaccount İsimlendirme

**Kalıp:** `{BÖLGE}_{MÜŞTERİ}_{ORTAM}`

```
Örnekler:
TR_ACME_PROD       → Türkiye, Acme Corp, Üretim
TR_ACME_DEV        → Türkiye, Acme Corp, Geliştirme
EU_GLOBEX_PROD     → Avrupa, Globex Inc, Üretim
US_INITECH_POC     → ABD, Initech, Kavram Kanıtı
```

### Destination İsimlendirme

Müşteri subaccount'u içinde daha basit isimler çalışır:

```mermaid
graph TD
    subgraph "Subaccount: TR_ACME_PROD"
        D1[S4_SALES]
        D2[S4_MATERIALS]
        D3[SF_EMPLOYEES]
        D4[ARIBA_PROCUREMENT]
    end

    subgraph "Subaccount: TR_GLOBEX_PROD"
        D5[S4_SALES]
        D6[S4_MATERIALS]
        D7[CRM_CUSTOMERS]
    end
```

**Bunun neden işe yaradığı:**
- ACME'nin subaccount'u içinde `S4_SALES` belirsizlik yaratmaz
- GLOBEX'in `S4_SALES` ile çakışma yok (farklı subaccount)

### Paylaşılan Subaccount'lar İçin (mecbur kalırsanız)

Paylaşılan bir subaccount'unuz varsa, tam önek kullanın:

```
ACME_S4_PROD_SALES
GLOBEX_S4_PROD_SALES
INITECH_S4_DEV_ORDERS
```

---

## 12.4 Güvenlik ve Erişim Yönetimi

### Kimlik Sağlayıcı Kurulumu

```mermaid
graph TD
    subgraph "Seçenekler"
        O1[SAP IAS<br/>Sizin yönettiğiniz IdP]
        O2[Müşterinin IdP'si<br/>Azure AD, Okta, vb.]
        O3[Karışık<br/>Müşteri başına farklı]
    end

    subgraph "Müşteri Türüne Göre Öneri"
        R1[Danışmanlık Projesi → SAP IAS]
        R2[Yönetilen Hizmet → Müşterinin IdP'si]
        R3[POC/Demo → SAP IAS]
    end
```

### Rol Atamaları

**Ekibiniz (Danışmanlar):**
```yaml
Global Account Seviyesi:
  - Global Account Administrator (siz)
  - Directory Administrator (ekip liderleri)

Subaccount Seviyesi (müşteri başına):
  - Subaccount Administrator
  - Cloud Foundry Space Developer
  - Destination Administrator
```

**Müşteri Ekibi:**
```yaml
Subaccount Seviyesi (sadece kendi subaccount'ları):
  - Subaccount Viewer (salt okunur) veya
  - Subaccount Administrator (tam kontrol)
  - Uygulamalarına özgü roller
```

### Müşteri Devir Erişim Kalıbı

```mermaid
sequenceDiagram
    participant C as Danışman
    participant SA as Müşteri Subaccount
    participant CL as Müşteri Admin

    C->>SA: Subaccount oluştur ve yapılandır
    Note over C,SA: Proje süresince tam admin

    C->>CL: Müşteriyi Subaccount Admin olarak ekle
    CL->>SA: Müşteri erişim kazanır

    Note over CL,SA: Geçiş dönemi: İkisi de erişime sahip

    CL->>C: Danışman erişimini kaldır
    Note over CL,SA: Müşteri subaccount'a sahip olur
```

---

## 12.5 Yetkilendirme Dağıtımı

### Yetkilendirmeleri Görselleştirme

```mermaid
pie title "AI Units Dağılımı"
    "ACME_PROD" : 40
    "ACME_DEV" : 10
    "GLOBEX_PROD" : 30
    "GLOBEX_DEV" : 10
    "Dahili" : 10
```

### Kotaları Yönetme

BTP Cockpit'te:

1. **Global Account → Entitlements**
2. **Entity Assignments** (eski adıyla "Manage Quota")
3. Subaccount'lara dağıtın:

```yaml
AI Core (AI Units):
  Toplam: 1000 ünite/ay
  Dağılım:
    - ACME_PROD: 400
    - ACME_DEV: 100
    - GLOBEX_PROD: 300
    - GLOBEX_DEV: 100
    - Dahili: 100

Cloud Foundry Runtime:
  Toplam: 16 GB
  Dağılım:
    - ACME_PROD: 4 GB
    - ACME_DEV: 2 GB
    - GLOBEX_PROD: 4 GB
    - GLOBEX_DEV: 2 GB
    - Dahili: 4 GB
```

---

## 12.6 Maliyet Tahsisi ve Geri Yükleme

### Müşteri Başına Maliyetleri Takip Etme

```mermaid
graph TD
    subgraph "BTP Cockpit"
        UA[Usage Analytics]
    end

    subgraph "Subaccount Başına Metrikler"
        M1[Tüketilen AI Units]
        M2[CF Runtime saatleri]
        M3[HANA depolama]
        M4[API çağrıları]
    end

    UA --> M1
    UA --> M2
    UA --> M3
    UA --> M4

    subgraph "Müşteri Faturası"
        I[Müşteri başına aylık maliyet raporu]
    end

    M1 --> I
    M2 --> I
    M3 --> I
    M4 --> I
```

### Dışa Aktarma ve Raporlama

1. **BTP Cockpit → Usage Analytics**
2. **Subaccount seçin** (veya directory)
3. **CSV'ye dışa aktar**
4. **Müşteri başına aylık raporlar oluşturun**

### Örnek Maliyet Raporu Yapısı

```markdown
# Aylık BTP Maliyet Raporu - ACME Corp

Dönem: Ocak 2026

| Hizmet | Kullanım | Birim Maliyet | Toplam |
|--------|----------|---------------|--------|
| AI Core | 350 AI Units | €0.10 | €35.00 |
| CF Runtime | 720 GB-saat | €0.05 | €36.00 |
| HANA Cloud | 50 GB | €2.00 | €100.00 |
| Destination Service | 10,000 çağrı | €0.001 | €10.00 |
| **Toplam** | | | **€181.00** |
```

---

## 12.7 Şablon Tabanlı Kurulum

### Altın Şablon Yaklaşımı

```mermaid
flowchart TD
    subgraph "Şablon Subaccount"
        T[TEMPLATES]
        T --> TA[Standart Agent'lar]
        T --> TD[Destination Şablonları]
        T --> TF[Fiori Uygulama Şablonları]
    end

    subgraph "Yeni Müşteri Kurulumu"
        C[Müşteri Subaccount Oluştur]
        C --> |"Buradan kopyala"| T
        C --> CUST[Müşteri için Özelleştir]
        CUST --> DEP[Dağıt]
    end
```

### Standart Kurulum Kontrol Listesi

Yeni bir müşteri entegre ederken:

```yaml
Müşteri Entegrasyon Kontrol Listesi:
  Subaccount:
    - [ ] İsimlendirme kuralına uygun subaccount oluştur
    - [ ] Cloud Foundry'yi etkinleştir
    - [ ] Yetkilendirmeleri ata

  Bağlantı:
    - [ ] S/4HANA'ya destination oluştur
    - [ ] SuccessFactors'a destination oluştur (gerekirse)
    - [ ] Cloud Connector kur (on-prem ise)

  Kimlik:
    - [ ] IdP ile güven yapılandır
    - [ ] İlk kullanıcıları oluştur
    - [ ] Rol koleksiyonlarını ayarla

  Hizmetler:
    - [ ] Standart Fiori uygulamalarını dağıt
    - [ ] Şablondan Joule agent'larını kur
    - [ ] Work Zone'u yapılandır

  Dokümantasyon:
    - [ ] Subaccount detaylarını belgele
    - [ ] Kimlik bilgilerini belgele (güvenli depolama)
    - [ ] Devir paketi oluştur
```

---

## 12.8 Çoklu Müşteri Joule Agent'ları

### Seçenek 1: Müşteri Başına Ayrı Agent'lar

```mermaid
graph TD
    subgraph "ACME Subaccount"
        A1[Müşteri Hizmetleri Agent<br/>ACME için]
    end

    subgraph "GLOBEX Subaccount"
        A2[Müşteri Hizmetleri Agent<br/>GLOBEX için]
    end
```

**Artıları:** Tam izolasyon, müşteriye özel özelleştirme
**Eksileri:** Daha fazla bakım, değişiklikler manuel olarak tekrarlanır

### Seçenek 2: Paylaşılan Agent Şablonu

Bir kez oluştur, her birine dağıt:

```mermaid
flowchart TD
    subgraph "Şablon"
        T[Genel Müşteri Hizmetleri Agent]
    end

    subgraph "ACME"
        A1[Dağıtılmış Agent]
        A1G[ACME Grounding Dokümanları]
        A1D[ACME Destination'ları]
    end

    subgraph "GLOBEX"
        A2[Dağıtılmış Agent]
        A2G[GLOBEX Grounding Dokümanları]
        A2D[GLOBEX Destination'ları]
    end

    T --> |"Dağıt"| A1
    T --> |"Dağıt"| A2
    A1G --> A1
    A1D --> A1
    A2G --> A2
    A2D --> A2
```

---

## 12.9 Felaket Kurtarma Hususları

### Müşteri Başına Yedekleme Stratejisi

```yaml
Yedekleme Sıklığı:
  Destination'lar: Haftalık dışa aktarma
  Agent'lar: Her değişiklikten sonra
  Fiori Uygulamaları: Git deposu
  Yapılandırmalar: Belgeleme + dışa aktarma

Depolama:
  Konum: Güvenli şirket depolaması
  Format: JSON dışa aktarımları + dokümantasyon
  Saklama: Minimum 90 gün
```

### Hızlı Kurtarma Planı

```mermaid
flowchart TD
    DR[Felaket Tespit Edildi] --> ASSESS{Kapsamı Değerlendir}
    ASSESS --> |"Tek destination"| FIX1[Yedekten yeniden oluştur]
    ASSESS --> |"Tüm subaccount"| FIX2[Yeni oluştur + hepsini geri yükle]
    ASSESS --> |"Global account"| FIX3[SAP Destek + tam geri yükleme]

    FIX1 --> TEST[İşlevselliği test et]
    FIX2 --> TEST
    FIX3 --> TEST

    TEST --> DONE[Normale dön]
```

---

## Temel Çıkarımlar

1. **Müşteri başına bir subaccount** — İzolasyon için şart
2. **İsimlendirme kuralları** — `{BÖLGE}_{MÜŞTERİ}_{ORTAM}`
3. **Net yetkilendirme dağıtımı** — Müşteri başına kotaları takip edin
4. **Doğru erişim kontrolü** — Danışmanlar vs. müşteri rolleri
5. **Şablon yaklaşımı** — Standartlaştırın ve çoğaltın
6. **Maliyet takibi** — Geri yüklemeyi etkinleştirin

---

## Sırada Ne Var?

Şimdi aynı çözümü birden fazla müşteriye dağıtmaya bakalım—şablon tabanlı vs. çok kiracılı yaklaşımlar.

---

*[Önceki: Kısım 11 – Agent Yaşam Döngüsü ve Dağıtım](11-agent-lifecycle.md) | [Sonraki: Kısım 13 – Müşteriler Arası Dağıtımlar](13-cross-customer-deployments.md)*

*[İçindekilere Dön](../content.md)*

---

**Yazar:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Hikaye Anlatıcısı & Dijital Dönüşüm Savunucusu

*Dünya genelindeki SAP öğrencileri için ❤️ ile oluşturuldu*
