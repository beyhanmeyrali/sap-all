# Kısım 4: RISE ve BTP Nasıl Birlikte Çalışır

> *Mahalle Görünümü*

---

## 4.1 Mahalle Görünümü: RISE Evi + BTP Uzantı Kanadı

İşte tıklamasını sağlayan resim:

```mermaid
graph LR
    subgraph "BULUT MAHALLESİ"
        subgraph "RISE EVİ - S/4HANA Cloud"
            S4_STD[Standart ERP]
            S4_MD[Master Data]
            S4_TX[Temel İşlemler]
            S4_MGD[SAP Tarafından Yönetilen]
        end

        subgraph "BTP UZANTI KANADI"
            BTP_FIORI[Özel Fiori Uygulamaları]
            BTP_RAP[RAP ABAP Servisleri]
            BTP_JOULE[Joule Agent'ları/Skill'leri]
            BTP_INT[Entegrasyonlar]
            BTP_WF[İş Akışları]
        end
    end

    S4_STD --> |"OData API'leri"| BTP_FIORI
    S4_MD --> |"Destination'lar"| BTP_RAP
    S4_TX --> |"Event'ler"| BTP_JOULE

    style S4_STD fill:#4CAF50,color:white
    style BTP_FIORI fill:#2196F3,color:white
    style BTP_JOULE fill:#FF9800,color:white
```

- **RISE** = Tamamen yönetilen S/4HANA bulut evi (ERP çekirdeği)
- **BTP** = Esnek uzantı kanadı/garaj/atölye
- **Destination'lar** = Onları birbirine bağlayan köprü

**Sıkı sıkıya birlikte çalışmak üzere tasarlandılar**:
- BTP'deki özel Fiori uygulaması OData aracılığıyla S/4'ten veri çeker
- Joule skill'i RISE S/4'ten satış siparişlerini okur
- BTP'deki entegrasyon akışı RISE'ı harici sistemlere bağlar

---

## 4.2 RISE Sözleşmelerinde BTP Kredileri

İşte çoğunun fark etmediği bir şey:

> Her RISE sözleşmesi, uzantılar oluşturmak için **BTP kredileri** (ücretsiz yakıt kuponları gibi) içerir.

```mermaid
flowchart TD
    CONTRACT[RISE Sözleşmesi] --> |"İçerir"| BTP_CREDITS[BTP Kredi Havuzu]

    BTP_CREDITS --> |"Şunlar için kullan"| USE1[Özel Uygulamalar]
    BTP_CREDITS --> |"Şunlar için kullan"| USE2[Integration Suite]
    BTP_CREDITS --> |"Şunlar için kullan"| USE3[AI Core / Joule]
    BTP_CREDITS --> |"Şunlar için kullan"| USE4[ABAP Environment]
    BTP_CREDITS --> |"Şunlar için kullan"| USE5[HANA Cloud]

    USE1 --> MONITOR[Kullanımı İzle!<br/>Krediler sınırsız değil]

    style BTP_CREDITS fill:#4CAF50,color:white
    style MONITOR fill:#f44336,color:white
```

### Ne Alırsınız

- Özel bir BTP subaccount'u (genellikle "BTP for RISE" olarak adlandırılır)
- Yaygın servisler için entitlement'lar (sözleşmeye bağlı)
- BTP servislerini tüketmek için krediler

### Tipik Dahil Edilen Servisler

| Servis | Ne İçin |
|--------|---------|
| SAP Build (Work Zone, Apps, PA) | Low-code geliştirme |
| Integration Suite | Sistemleri bağlama |
| ABAP Environment | Bulut ABAP geliştirme |
| AI Core / Joule | AI yetenekleri |
| HANA Cloud | Ek veritabanları |

### Dikkat!

Krediler sınırsız değil. Yüksek tüketimli servisler (AI ve büyük HANA instance'ları gibi) kredileri hızlıca tüketebilir. Kullanımı izleyin!

---

## 4.3 ABAP/Fiori Geliştiricileri İçin Yeni Gerçeklik

Ne değiştiğini somutlaştıralım:

### Eski Dünya (On-Prem S/4)

```mermaid
graph TD
    subgraph "S/4HANA On-Prem"
        SE80[SE80: Z-programlar]
        SE11[SE11: Z-tablolar]
        EXITS[User Exit'ler & BADI'ler]
        FLP[Gömülü Fiori Launchpad]

        SE80 --> SAME[Her Şey Tek Sistemde]
        SE11 --> SAME
        EXITS --> SAME
        FLP --> SAME
    end

    style SAME fill:#FF9800,color:white
```

### RISE Dünyası

```mermaid
graph TD
    subgraph "S/4HANA Cloud - RISE"
        S4[Clean Core]
        S4 --> |"❌"| NO1[Core değiştirilemez]
        S4 --> |"❌"| NO2[Z-tablo eklenemez]
        S4 --> |"❌"| NO3[Released olmayan API kullanılamaz]
        S4 --> |"✓"| YES1[Key User ile yapılandır]
        S4 --> |"✓"| YES2[Harici kullanım için API'ler sunar]
    end

    S4 --> |"API'ler<br/>OData, Event'ler"| BTP

    subgraph "BTP Subaccount"
        BTP[Atölyeniz]
        BTP --> |"✓"| CAN1[RAP ile özel ABAP]
        BTP --> |"✓"| CAN2[Özel Fiori Uygulamaları]
        BTP --> |"✓"| CAN3[Joule Agent'ları & Skill'leri]
        BTP --> |"✓"| CAN4[Entegrasyon Akışları]
        BTP --> |"✓"| CAN5[HANA'da Özel Tablolar]
    end

    style S4 fill:#f44336,color:white
    style BTP fill:#4CAF50,color:white
```

### Bu Günlük Olarak Ne Anlama Geliyor

| Görev | Eski Yol | RISE + BTP Yolu |
|-------|----------|-----------------|
| Özel alan ekle | SE11'de append structure | Key User genişletilebilirliği veya BTP |
| Özel rapor | SE38 programı | BTP ABAP Env'de RAP → Fiori uygulama |
| Özel tablo | S/4'te SE11 | BTP'de HANA Cloud tablosu |
| Karmaşık mantık | Core'da fonksiyon modülü | BTP'de servis, API ile çağır |
| Fiori uzantısı | WebIDE'de genişlet, FLP'ye deploy et | BAS'ta genişlet, BTP'ye deploy et |

---

## 4.4 Karşılaştırma Tablosu: Klasik On-Prem vs. RISE vs. BTP

```mermaid
graph TD
    subgraph "Karar Matrisi"
        Q1{Nerede yaşar?}

        Q1 --> |"Standart ERP Mantığı"| RISE[RISE S/4HANA]
        Q1 --> |"Özel Uzantılar"| BTP[BTP]
        Q1 --> |"Master Data"| RISE
        Q1 --> |"AI/Joule Agent'ları"| BTP
        Q1 --> |"Entegrasyonlar"| BTP
        Q1 --> |"Özel Raporlar"| BTP
    end

    style RISE fill:#4CAF50,color:white
    style BTP fill:#2196F3,color:white
```

| Özellik | Klasik On-Prem | RISE (S/4 Cloud) | BTP (Uzantılar) |
|---------|----------------|------------------|-----------------|
| **Altyapıyı kim yönetiyor?** | Basis ekibiniz | SAP | SAP |
| **S/4HANA versiyonu** | Siz kontrol edersiniz | SAP güncelleme yapar | Yok |
| **Özel ABAP** | Tam özgürlük (riskli) | Sadece Clean Core | ABAP Env'de RAP |
| **Fiori uygulamaları** | On-prem FLP'ye deploy | Gömülü FLP | BAS'ta oluştur/deploy et |
| **Uzantı konumu** | S/4 içinde | Dışarıda (BTP) | BTP budur! |
| **Maliyet modeli** | Donanım + lisanslar | Abonelik | Kredi/tüketim |
| **Tipik kullanım** | Legacy SAP mağazaları | Bulut dönüşümü | Tüm özel işler |

---

## 4.5 Pratik Örnek: Müşteri Özel Dashboard İstiyor

### Eski Yol (On-Prem)

```mermaid
sequenceDiagram
    participant DEV as Geliştirici
    participant SE80 as SE80
    participant SE11 as SE11
    participant WEBIDE as WebIDE
    participant FLP as On-Prem FLP

    DEV->>SE80: Veri için Z-program oluştur
    DEV->>SE11: Z-CDS view'lar oluştur
    DEV->>WEBIDE: Fiori uygulama oluştur
    DEV->>FLP: On-prem FLP'ye deploy et
    Note over DEV,FLP: Hepsi tek sistemde
```

### RISE + BTP Yolu

```mermaid
sequenceDiagram
    participant S4 as RISE S/4HANA
    participant BTP as BTP Subaccount
    participant BAS as Business App Studio
    participant WZ as SAP Build Work Zone

    S4->>S4: Standart OData servislerini etkinleştir
    BTP->>BTP: S/4'e destination oluştur
    BAS->>BAS: OData tüketen Fiori uygulama oluştur
    BAS->>BTP: BTP'ye deploy et
    BTP->>WZ: Work Zone'da yapılandır
    Note over S4,WZ: S/4 core temiz kalır!
```

Dashboard kullanıcılara **aynı görünür**, ama mimari temelden farklıdır.

---

## 4.6 Tam Mimari Resim

```mermaid
graph TD
    subgraph "Son Kullanıcılar"
        USER[İş Kullanıcıları]
    end

    subgraph "Erişim Katmanı"
        WZ[SAP Build Work Zone]
        JOULE[Joule Copilot]
    end

    subgraph "BTP Uzantı Katmanı"
        FIORI[Özel Fiori Uygulamaları]
        RAP[RAP Servisleri]
        AGENTS[Joule Agent'ları]
        CPI[Integration Suite]
    end

    subgraph "Bağlantı Katmanı"
        DEST[Destination'lar]
        CC[Cloud Connector]
    end

    subgraph "RISE S/4HANA Cloud"
        S4[Clean Core S/4HANA]
        API[Released OData API'leri]
    end

    subgraph "On-Premise Sistemler"
        ECC[Legacy ECC]
        THIRD[Üçüncü Taraf Sistemler]
    end

    USER --> WZ
    USER --> JOULE
    WZ --> FIORI
    JOULE --> AGENTS
    FIORI --> RAP
    AGENTS --> RAP
    RAP --> DEST
    DEST --> API
    API --> S4
    CPI --> CC
    CC --> ECC
    CC --> THIRD

    style S4 fill:#4CAF50,color:white
    style BTP fill:#2196F3,color:white
    style USER fill:#FF9800,color:white
```

---

## 4.7 Gerçek Dünya URL Örnekleri

RISE + BTP ile çalışırken şu URL kalıplarıyla karşılaşacaksınız:

| Sistem | URL Kalıbı | Örnek |
|--------|------------|-------|
| **BTP Cockpit** | `cockpit.btp.cloud.sap` | `https://cockpit.btp.cloud.sap/cockpit/?idp=...` |
| **S/4HANA Cloud** | `my{numara}.s4hana.ondemand.com` | `https://my300001.s4hana.ondemand.com` |
| **Business App Studio** | `{bölge}.applicationstudio.cloud.sap` | `https://eu10.applicationstudio.cloud.sap` |
| **Joule Studio** | `joule-studio-{bölge}.cfapps.{bölge}.hana.ondemand.com` | `https://joule-studio-eu10.cfapps.eu10.hana.ondemand.com` |
| **OData Servisi** | `{s4_url}/sap/opu/odata/sap/{servis}` | `https://my300001.s4hana.ondemand.com/sap/opu/odata/sap/API_SALES_ORDER_SRV` |

---

## Temel Çıkarımlar

1. **RISE yönetilen ERP çekirdeği** — SAP altyapıyı yöneten S/4HANA
2. **BTP uzantı platformu** — özel işlerinizin yaşadığı yer
3. **API'ler aracılığıyla bağlanırlar** — Destination'lar ikisini köprüler
4. **RISE BTP kredileri içerir** — uzantılar için kullanın
5. **Becerileriniz transfer olur** — ABAP hala ABAP, sadece yeni bir konumda

---

## Sırada Ne Var?

RISE + BTP'nin büyük resmini anlıyorsunuz. Şimdi pratik olalım. Bölüm III'te, günlük kullanacağınız temel BTP kavramlarına dalacağız, **Destination'lar**—sistemler arasındaki kritik köprü—ile başlayarak.

---

*[Önceki: Kısım 3 – RISE with SAP](03-rise-with-sap.md) | [Sonraki: Kısım 5 – Destination'lar](05-destinations.md)*

*[İçindekilere Dön](../content.md)*

---

**Yazar:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Hikaye Anlatıcısı & Dijital Dönüşüm Savunucusu

*Dünya genelindeki SAP öğrencileri için ❤️ ile oluşturuldu*
