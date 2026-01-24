# Kısım 3: RISE with SAP Açıklandı

> *Bir Ürün Değil—Bir Paket!*

---

Bu, en çok ABAP geliştiricisinin kafasını karıştıran kavram. Net bir şekilde açıklayalım.

---

## 3.1 RISE Aslında Nedir (Bir Ürün Değil!)

**RISE with SAP** ayrı bir ürün veya platform DEĞİLDİR.

**S/4HANA'yı temel alan** işinizi buluta taşımanıza yardımcı olan SAP'tan **komple bir paket teklifi**—bir "dönüşüm paketi"dir.

```mermaid
graph TD
    subgraph "RISE with SAP Paketi"
        S4[S/4HANA Cloud<br/>ERP Sistemi]
        INFRA[Altyapı & Operasyonlar<br/>SAP Her Şeyi Yönetir]
        TRANS[İş Dönüşüm Servisleri<br/>Migrasyon Araçları]
        CLEAN[Clean Core Yaklaşımı<br/>Uzantılar Dışarıda Kalır]
        BTP[BTP Kredileri<br/>Uzantılar İçin]
    end

    RISE[RISE with SAP] --> S4
    RISE --> INFRA
    RISE --> TRANS
    RISE --> CLEAN
    RISE --> BTP

    style RISE fill:#FF9800,color:white
    style S4 fill:#4CAF50,color:white
    style BTP fill:#2196F3,color:white
```

> **RISE'ı şöyle düşünün**: SAP'ın, *"Evi inşa etme ve bakımıyla uğraşmayı bırak. Sana tamamen bizim tarafımızdan yönetilen hazır, modern bir bulut evi (S/4HANA Cloud) ve onu akıllıca yenileme araçları vereceğiz."* demesi.

### RISE Neleri İçerir

1. **SAP S/4HANA Cloud** (ERP sisteminin kendisi)
2. **Altyapı & Operasyonlar** (SAP her şeyi yönetir)
3. **İş Dönüşüm Servisleri** (migrasyona yardımcı araçlar)
4. **Clean Core Yaklaşımı** (uzantılar core dışında kalır)
5. **BTP Kredileri** (uzantılar oluşturmak için)

---

## 3.2 S/4HANA Cloud Private Edition vs. Public Edition

Eski ABAP'çıların kafasının karıştığı yer burası. İki çeşit var:

```mermaid
graph LR
    subgraph "Public Edition"
        PUB[Çok Kiracılı]
        PUB --> PUB1[Paylaşımlı Sistem]
        PUB --> PUB2[Çok Sınırlı Özelleştirme]
        PUB --> PUB3[Zorunlu Üç Aylık Güncellemeler]
        PUB --> PUB4[Özel ABAP Yok]
    end

    subgraph "Private Edition"
        PRIV[Tek Kiracılı]
        PRIV --> PRIV1[Adanmış Instance]
        PRIV --> PRIV2[Daha Fazla Özelleştirme]
        PRIV --> PRIV3[Kontrollü Yükseltme Zamanlaması]
        PRIV --> PRIV4[Özel ABAP'a İzin Var]
    end

    style PUB fill:#2196F3,color:white
    style PRIV fill:#4CAF50,color:white
```

### Public Edition (Çok Kiracılı)

- SAP büyük bir sistem çalıştırır, siz bir "dilim" (tenant) alırsınız
- Çok sınırlı özelleştirme
- Üç aylık güncellemeler size zorla yüklenir
- Şunlar için iyi: Standart süreçler, küçük şirketler

### Private Edition (Tek Kiracılı)

- Kendi adanmış S/4HANA instance'ınızı alırsınız
- Daha fazla özelleştirme mümkün (on-prem'e daha yakın hissettiriyor)
- Yükseltme zamanlamasını siz kontrol edersiniz (sınırlar dahilinde)
- Özel ABAP kodu çalışabilir (kısıtlamalarla)
- Şunlar için iyi: Özel ihtiyaçları olan büyük kurumsal şirketler

> **ABAP geliştiricileri için**: Private Edition genellikle RISE projelerinde karşılaşacağınız şeydir çünkü daha fazla özelleştirmeye izin verir.

### Karşılaştırma Tablosu

| Özellik | Public Edition | Private Edition |
|---------|---------------|-----------------|
| Kiracılık | Çok kiracılı (paylaşımlı) | Tek kiracılı (adanmış) |
| Özelleştirme | Çok sınırlı | Daha esnek |
| Özel ABAP | Yok | İzin var (kurallarla) |
| Yükseltmeler | Zorunlu üç aylık | Kontrollü zamanlama |
| Tipik müşteri | KOBİ, greenfield | Büyük kurumsal, brownfield |

---

## 3.3 RISE Paketi: Neler Dahil

Bir müşteri RISE sözleşmesi imzaladığında, şunları alır:

```mermaid
mindmap
  root((RISE Paketi))
    S/4HANA Cloud
      ERP Sistemi
      On-prem S/4 eşdeğeriniz
    SAP HANA Veritabanı
      In-memory veritabanı
      Dahil, ayrı değil
    Altyapı
      Sunucular
      Depolama
      Ağ
      Hepsi SAP tarafından yönetilir
    Operasyonlar
      Yama
      Yedekleme
      İzleme
      Yüksek Erişilebilirlik
    Signavio
      Süreç Madenciliği
      Süreç Tasarımı
    Learning Hub
      Eğitim Erişimi
    BTP Kredileri
      Uzantı Platformu
      Özel uygulama oluştur
```

| Bileşen | Nedir | Eski Dünya Karşılığı |
|---------|-------|---------------------|
| **S/4HANA Cloud** | ERP sistemi | On-prem S/4 veya ECC'niz |
| **SAP HANA Veritabanı** | In-memory veritabanı | HANA/Oracle/DB2'niz |
| **Altyapı** | Sunucular, depolama, ağ | Veri merkeziniz |
| **Operasyonlar** | Yama, yedekleme, izleme | Basis ekibiniz |
| **Signavio** | Süreç madenciliği & tasarım | Yok |
| **Learning Hub** | Eğitim erişimi | Yok |
| **BTP Kredileri** | Uzantı platformu | Daha önce ayrı satın alma |

---

## 3.4 Altyapı & Operasyonlar – Basis Endişelerine Veda

Eski SAP dükkanları için en büyük değişikliklerden biri:

```mermaid
graph TD
    subgraph "SAP'ın Yönettiği"
        M1[Sunucu Provisioning & Ölçekleme]
        M2[İşletim Sistemi Yaması]
        M3[Veritabanı Yedekleme & Kurtarma]
        M4[Yüksek Erişilebilirlik Kurulumu]
        M5[Felaket Kurtarma]
        M6[Performans İzleme]
        M7[Güvenlik Yamaları]
    end

    subgraph "Sizin Hala Yaptığınız"
        Y1[Fonksiyonel Konfigürasyon]
        Y2[Kullanıcı Yönetimi]
        Y3[BTP'de Özel Geliştirme]
        Y4[Test & Değişiklik Yönetimi]
    end

    SAP[SAP Sorumluluğu] --> M1
    SAP --> M2
    SAP --> M3
    SAP --> M4
    SAP --> M5
    SAP --> M6
    SAP --> M7

    YOU[Sizin Sorumluluğunuz] --> Y1
    YOU --> Y2
    YOU --> Y3
    YOU --> Y4

    style SAP fill:#4CAF50,color:white
    style YOU fill:#2196F3,color:white
```

### RISE'ta SAP'ın Yönettiği

- ✅ Sunucu provisioning ve ölçekleme
- ✅ İşletim sistemi yaması
- ✅ Veritabanı yedekleme ve kurtarma
- ✅ Yüksek erişilebilirlik kurulumu
- ✅ Felaket kurtarma
- ✅ Performans izleme
- ✅ Güvenlik yamaları

### Sizin Hala Yaptığınız

- ✅ Fonksiyonel konfigürasyon (işlemler, master data)
- ✅ Kullanıcı yönetimi
- ✅ Özel geliştirme (BTP'de, core'da değil)
- ✅ Test ve değişiklik yönetimi

> **Basis ekipleri için**: Rolünüz "sunucuları çalışır tutma"dan "mimari ve yönetişim"e kayar.

---

## 3.5 Clean Core Sözü

Bu temel zihniyet değişimi:

### Eski Dünya: Her Şeyi Değiştir

```mermaid
graph TD
    subgraph "Klasik On-Prem Yaklaşımı"
        S4OLD[S/4HANA On-Prem]
        S4OLD --> MOD1[SAP Kodunda Modifikasyonlar]
        S4OLD --> MOD2[Her Yerde Z-Tablolar]
        S4OLD --> MOD3[User Exit'ler]
        S4OLD --> MOD4[Core'da BADI'ler]
        S4OLD --> MOD5[Özel Fonksiyon Modülleri]
    end

    MOD1 --> PAIN[Yükseltme Kabusu]
    MOD2 --> PAIN
    MOD3 --> PAIN
    MOD4 --> PAIN
    MOD5 --> PAIN

    style PAIN fill:#f44336,color:white
```

Sorunlar:
- Yükseltmeler kabusa dönüşüyor
- Destek karmaşıklaşıyor
- Test sonsuza kadar sürüyor

### RISE Dünyası: Core'u Temiz Tut

```mermaid
graph TD
    subgraph "S/4HANA Cloud - Clean Core"
        S4[Yalnızca Standart SAP Kodu]
        S4 --> API[API'ler: OData, SOAP, RFC]
    end

    API --> |"Destination'lar ile Bağlan"| BTP

    subgraph "BTP Uzantı Katmanı"
        BTP --> RAP[RAP tabanlı ABAP]
        BTP --> FIORI[Fiori Uygulamaları]
        BTP --> LOGIC[Özel Mantık]
        BTP --> INT[Entegrasyonlar]
    end

    style S4 fill:#4CAF50,color:white
    style BTP fill:#2196F3,color:white
```

### "Clean Core" Pratikte Ne Anlama Geliyor

| İzin Verilen | İzin Verilmeyen |
|--------------|-----------------|
| Released API'leri kullanma | Standart kodu değiştirme |
| Key User genişletilebilirliği | S/4'te özel dictionary objeleri |
| Yan yana uzantılar (BTP) | Released olmayan fonksiyon modülleri |
| Özel CDS view'lar (released objeler) | Doğrudan tablo değişiklikleri |

Clean Core etkilerini Kısım 17'de daha derinlemesine inceleyeceğiz.

---

## 3.6 RISE Karar Akış Şeması

```mermaid
flowchart TD
    START[Buluta mı Geçiyorsunuz?] --> Q1{Mevcut Sistem?}

    Q1 --> |"SAP ECC"| PATH1[Greenfield veya Brownfield?]
    Q1 --> |"S/4HANA On-Prem"| PATH2[RISE'a Dönüştür]
    Q1 --> |"SAP Dışı"| PATH3[Yeni Uygulama]

    PATH1 --> |"Greenfield - Yeni Başla"| PUBLIC[Public Edition'ı Değerlendir]
    PATH1 --> |"Brownfield - Özelleştirmeleri Koru"| PRIVATE[Private Edition]

    PATH2 --> PRIVATE
    PATH3 --> PUBLIC

    PUBLIC --> BTP1[Minimal BTP Uzantıları]
    PRIVATE --> BTP2[Daha Fazla BTP Uzantısı Gerekli]

    style PRIVATE fill:#4CAF50,color:white
    style PUBLIC fill:#2196F3,color:white
    style BTP2 fill:#FF9800,color:white
```

---

## Temel Çıkarımlar

1. **RISE bir paket, ürün değil** — S/4HANA + altyapı + servisler + BTP kredileri
2. **Private Edition** özelleştirmeye ihtiyaç duyan kurumsal şirketler için
3. **SAP altyapıyı yönetir** — Basis rolünüz değişir
4. **Clean Core zorunlu** — uzantılar S/4 core'a değil BTP'ye gider
5. **BTP kredileri dahil** — özel geliştirme için kullanın

---

## Sırada Ne Var?

Şimdi RISE ve BTP'nin tam olarak nasıl bağlandığını görelim. Geliştirme yaparken "BTP uzantı kanadı" gerçekte neye benziyor?

---

*[Önceki: Kısım 2 – BTP Mimarisi](02-btp-architecture.md) | [Sonraki: Kısım 4 – RISE ve BTP Nasıl Birlikte Çalışır](04-rise-and-btp.md)*

*[İçindekilere Dön](../content.md)*

---

**Yazar:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Hikaye Anlatıcısı & Dijital Dönüşüm Savunucusu

*Dünya genelindeki SAP öğrencileri için ❤️ ile oluşturuldu*
