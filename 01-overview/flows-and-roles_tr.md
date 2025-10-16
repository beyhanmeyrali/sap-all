# SAP İş Akışları ve Organizasyonel Roller

## İçindekiler
1. [Organizasyon Yapısı ve Sorumluluklar](#organizasyon-yapısı-ve-sorumluluklar)
2. [Ana İş Süreci Akışları](#ana-iş-süreci-akışları)
3. [Rol Bazlı T-Code Matrisi](#rol-bazlı-t-code-matrisi)
4. [Üretim Tipleri ve Stratejileri](#üretim-tipleri-ve-stratejileri)
5. [Transfer Tipleri ve Şirketlerarası Süreçler](#transfer-tipleri-ve-şirketlerarası-süreçler)
6. [Entegrasyon Noktaları](#entegrasyon-noktaları)

---

## Organizasyon Yapısı ve Sorumluluklar

### Kurumsal Organizasyon Şeması
```
                        Şirket Yapısı
                            CEO
                             |
        ┌───────────────────┼───────────────────┐
        |                  |                   |
   Operasyonlar        Ticari Birim      Finans & İdari
        |                  |                   |
   ┌────┼────┐        ┌────┼────┐        ┌────┼────┐
   |    |    |        |    |    |        |    |    |
  MM   PP   QM       SD   CRM  LE       FI   CO   HCM
  |    |    |        |    |    |        |    |    |
Malzeme Üretim Kalite Satış Pazarlama Lojistik Finans Kontrolü İK
Müdürü Müdürü Müdürü Müdürü Müdürü  Müdürü  Müdürü Müdürü   Müdürü
```

### Departman Sorumlulukları ve Ana T-Kodları

| Departman | Rol | Ana Modüller | Anahtar T-Kodları | Sorumluluklar |
|-----------|-----|--------------|------------------|---------------|
| **Malzeme Yönetimi** | Satın Alma ve Stok | MM, WM | ME21N, MIGO, MB52 | Satın alma siparişleri, mal kabul, stok yönetimi |
| **Üretim Planlama** | Üretim Kontrolü | PP, QM | CO01, MD02, CO11N | Üretim emirleri, MRP, atölye yönetimi |
| **Satış ve Dağıtım** | Müşteri Yönetimi | SD, CRM | VA01, VF01, VL01N | Satış siparişleri, faturalandırma, teslimat |
| **Mali Muhasebe** | Mali Kontrol | FI, AP, AR | FB01, F-53, F-22 | Genel muhasebe, borç/alacak hesapları |
| **Kontroling** | Maliyet Yönetimi | CO, PA | KS01, KO01, KE30 | Masraf yerleri, iç siparişler, karlılık |
| **İnsan Kaynakları** | Personel Yönetimi | HCM, PY | PA30, PC00, CAT2 | Personel yönetimi, bordro, mesai |
| **Kalite Yönetimi** | Kalite Güvencesi | QM | QE01, QM01, QC01 | Muayene, kalite bildirimleri, sertifikalar |
| **Tesis Bakımı** | Varlık Yönetimi | PM, CS | IW31, IW37N, IP02 | İş emirleri, önleyici bakım |
| **Lojistik Yürütme** | Depo Operasyonları | LE, WM | LT01, VL02N, LI01 | Toplama, paketleme, sevkiyat |

---

## Ana İş Süreci Akışları

### 1. Teklif-Tahsilat (Q2C) Süreç Akışı

```
Teklif-Tahsilat Akışı:
Müşteri Talebi → Teklif → Satış Siparişi → Teslimat → Faturalandırma → Ödeme

Adım 1: Teklif (Satış Departmanı)
┌─────────────────────────────────────────────────────────────┐
│ Rol: Satış Temsilcisi                                       │
│ T-Kodu: VA21 (Teklif Oluştur)                             │
│ Girdi: Müşteri gereksinimleri, ürün özellikleri           │
│ Çıktı: Fiyatlandırma ve teslimat şartlarını içeren teklif  │
│ Süre: 1-3 gün                                              │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 2: Satış Siparişi (Satış Departmanı)
┌─────────────────────────────────────────────────────────────┐
│ Rol: Satış Siparişi İşlemcisi                              │
│ T-Kodu: VA01 (Satış Siparişi Oluştur)                     │
│ Girdi: Müşteri kabulü, teklif referansı                   │
│ Çıktı: Teslimat programı ile satış siparişi               │
│ Entegrasyon: SD → PP (üretim gereksinimleri)              │
│              SD → MM (malzeme rezervasyonları)             │
│              SD → FI (gelir muhasebeleştirme)              │
│ Süre: Aynı gün                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 3: ATP Kontrolü ve Planlama
┌─────────────────────────────────────────────────────────────┐
│ Rol: Üretim Planlayıcısı                                   │
│ T-Kodu: MD04 (Stok/Gereksinim Listesi)                    │
│         CO09 (Teslim Edilebilir Miktar)                   │
│ Süreç: Malzeme varlığı ve üretim kapasitesi kontrolü       │
│ Çıktı: Onaylanmış teslimat tarihi                         │
│ Süre: Gerçek zamanlı - 1 gün                              │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 4: Üretim/Satın Alma
┌─────────────────────────────────────────────────────────────┐
│ Rol: Üretim Müdürü / Satın Alma Müdürü                    │
│ T-Kodu: CO01 (Üretim Emri Oluştur) veya                   │
│         ME21N (Satın Alma Siparişi Oluştur)               │
│ Süreç: Üretim veya dış tedarik                            │
│ Süre: Karmaşıklığa bağlı olarak 1-8 hafta                 │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 5: Teslimat İşleme
┌─────────────────────────────────────────────────────────────┐
│ Rol: Depo/Sevkiyat Koordinatörü                           │
│ T-Kodu: VL01N (Teslimat Oluştur)                          │
│         VL02N (Toplama ve Mal Çıkışı)                     │
│ Süreç: Müşteriye toplama, paketleme, sevkiyat             │
│ Entegrasyon: LE → FI (satılan malın maliyeti kaydı)       │
│ Süre: 1-3 gün                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 6: Faturalandırma
┌─────────────────────────────────────────────────────────────┐
│ Rol: Fatura Memuru                                         │
│ T-Kodu: VF01 (Fatura Belgesi Oluştur)                     │
│ Süreç: Teslimata dayalı fatura üretimi                    │
│ Entegrasyon: SD → FI (gelir ve alacak kaydı)              │
│ Süre: Teslimat ile aynı gün                               │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 7: Ödeme İşleme
┌─────────────────────────────────────────────────────────────┐
│ Rol: Müşteri Alacakları Memuru                            │
│ T-Kodu: F-28 (Müşteri Ödemesi Kaydet)                     │
│         F-32 (Müşteri Hesabını Kapat)                     │
│ Süreç: Müşteri ödemesini faturaya uygula                  │
│ Süre: Ödeme şartlarına bağlı (0-90 gün)                   │
└─────────────────────────────────────────────────────────────┘
```

### 2. Satın Al-Öde (P2P) Süreç Akışı

```
Satın Al-Öde Akışı:
Gereksinim → Satın Alma Talebi → Satın Alma Siparişi → Mal Kabul → Fatura → Ödeme

Adım 1: Satın Alma Talebi
┌─────────────────────────────────────────────────────────────┐
│ Rol: Talep Eden (Departman Kullanıcısı)                    │
│ T-Kodu: ME51N (Satın Alma Talebi Oluştur)                 │
│ Girdi: MRP'den veya manuel talep olan malzeme gereksinimi  │
│ Çıktı: Onay için satın alma talebi                        │
│ Onay: Departman müdürü onayı gerekli                      │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 2: Satın Alma Siparişi Oluşturma
┌─────────────────────────────────────────────────────────────┐
│ Rol: Satın Almacı/Alıcı                                    │
│ T-Kodu: ME21N (Satın Alma Siparişi Oluştur)               │
│ Süreç: Kaynak seçimi, fiyat müzakeresi, sipariş oluşturma │
│ Çıktı: Tedarikçiye gönderilen satın alma siparişi         │
│ Entegrasyon: MM → FI (taahhüt oluşturma)                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 3: Mal Kabul
┌─────────────────────────────────────────────────────────────┐
│ Rol: Depo Memuru                                           │
│ T-Kodu: MIGO (Mal Kabul)                                  │
│ Süreç: Mal kabul, kalite kontrolü, stok kaydı             │
│ Entegrasyon: MM → FI (stok değeri artışı)                 │
│              MM → QM (gerekirse kalite muayenesi)         │
│ Hareket Tipi: 101 (Siparişe karşı mal kabul)              │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 4: Fatura Girişi
┌─────────────────────────────────────────────────────────────┐
│ Rol: Satın Alma Borçları Memuru                           │
│ T-Kodu: MIRO (Fatura Girişi)                              │
│ Süreç: 3'lü eşleştirme (Sipariş, Mal Kabul, Fatura)      │
│ Entegrasyon: MM → FI (borç yükümlülüğü oluşturma)         │
│ Tolerans: Fiyat ve miktar farkı kontrolü                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 5: Ödeme İşleme
┌─────────────────────────────────────────────────────────────┐
│ Rol: Ödeme İşlemcisi                                       │
│ T-Kodu: F110 (Otomatik Ödeme)                             │
│ Süreç: Vade tarihlerine ve şartlara göre ödeme çalıştırma │
│ Entegrasyon: FI → Banka arayüzü ödeme dosyası için        │
│ Çıktı: Tedarikçiye ödeme bildirimi                        │
└─────────────────────────────────────────────────────────────┘
```

### 3. Planla-Üret (P2P) Üretim Akışı

```
Planla-Üret Akışı:
Talep Planlama → MRP → Üretim Emri → Üretim → Mal Kabul

Adım 1: Talep Planlama
┌─────────────────────────────────────────────────────────────┐
│ Rol: Talep Planlayıcısı                                    │
│ T-Kodu: MD61 (Planlı Bağımsız Gereksinimler Oluştur)      │
│ Girdi: Satış tahmini, geçmiş veriler, pazar trendleri     │
│ Çıktı: Mamul ürünler için talep planı                     │
│ Sıklık: Aylık/Haftalık                                     │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 2: MRP Çalışması
┌─────────────────────────────────────────────────────────────┐
│ Rol: Üretim Planlayıcısı                                   │
│ T-Kodu: MD02 (MRP Çalışması)                              │
│ Süreç: Talebe dayalı malzeme gereksinimlerini hesapla     │
│ Çıktı: Üretim ve satın alma için planlı siparişler       │
│ Entegrasyon: PP → MM (satın alma talepleri)               │
│ Sıklık: Günlük                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 3: Üretim Emri Oluşturma
┌─────────────────────────────────────────────────────────────┐
│ Rol: Üretim Müdürü                                         │
│ T-Kodu: CO01 (Üretim Emri Oluştur)                        │
│ Girdi: MRP'den planlı siparişler                          │
│ Çıktı: Rota ve BOM ile üretim emri                        │
│ Entegrasyon: PP → CO (maliyet nesnesi oluşturma)          │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 4: Malzeme Çıkışı
┌─────────────────────────────────────────────────────────────┐
│ Rol: Depo/Üretim Personeli                                │
│ T-Kodu: CO11N (Emir için Mal Çıkışı)                     │
│ Süreç: Hammaddeleri üretime çıkar                         │
│ Hareket Tipi: 261 (Emire çıkış)                           │
│ Entegrasyon: PP → CO (fiili maliyetler)                   │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 5: Üretim Onayı
┌─────────────────────────────────────────────────────────────┐
│ Rol: Atölye Operatörü                                      │
│ T-Kodu: CO15 (Üretim Emrini Onayla)                       │
│ Süreç: Üretilen miktarları ve harcanan zamanı raporla     │
│ Entegrasyon: PP → CO (faaliyet nakli)                     │
│              PP → QM (kalite sonuçları)                   │
└─────────────────────────────────────────────────────────────┘
                          ↓
Adım 6: Mal Kabul
┌─────────────────────────────────────────────────────────────┐
│ Rol: Üretim/Depo Personeli                                │
│ T-Kodu: CO15 (MK ile final onay)                          │
│ Süreç: Mamul ürünleri stoka al                            │
│ Hareket Tipi: 101 (Stoka giriş)                           │
│ Entegrasyon: PP → FI (mamul ürün stoğu)                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Rol Bazlı T-Code Matrisi

### Yönetici Seviyesi T-Kodları
| Rol | Anahtar T-Kodları | Amaç |
|-----|------------------|------|
| **CEO/Genel Müdür** | S_ALV_31000002, MC.1, MC.2 | Yönetici panoları, karlılık analizi |
| **CFO** | S_ALR_87012277, F.01, KSV5 | Mali tablolar, dönem sonu kapanışı |
| **COO** | COOIS, MD04, VL06O | Operasyon genel bakış, üretim durumu |
| **Satış VP** | VA05, VF04, SD01 | Satış analizi, fatura genel bakış |
| **Üretim VP** | CO05N, CM21, QM06 | Üretim kontrolü, kapasite planlama |

### Departman Müdürü Seviyesi T-Kodları
| Departman | Rol | Anahtar T-Kodları | Amaç |
|-----------|-----|------------------|------|
| **Satın Alma** | Satın Alma Müdürü | ME2L, ME2M, ME80FN | Sipariş takibi, tedarikçi değerlendirme |
| **Üretim** | Üretim Müdürü | COOIS, CM21, CO40 | Üretim kontrolü, kapasite yönetimi |
| **Satış** | Satış Müdürü | VA05, VF04, VKM4 | Satış takibi, kredi yönetimi |
| **Finans** | Finans Müdürü | FBL1N, FBL5N, F110 | Borç/alacak takibi, ödeme işleme |
| **Depo** | Depo Müdürü | LX02, MB52, MI20 | Stok takibi, fiziksel envanter |

### Operasyonel Seviye T-Kodları
| Fonksiyon | Rol | Ana T-Kodları | Günlük Görevler |
|-----------|-----|---------------|-----------------|
| **Satın Alma** | Alıcı | ME21N, ME22N, ME23N, MIRO | Sipariş oluştur/değiştir, fatura işleme |
| **Üretim** | Planlayıcı | MD02, CO01, CO02, CO15 | MRP, üretim emirleri, onaylar |
| **Satış** | Satış Temsilcisi | VA01, VA02, VA03, VF01 | Satış siparişleri, faturalandırma |
| **Muhasebe** | Muhasebeci | FB01, FB50, F-53, F-22 | Genel muhasebe nakli, tedarikçi/müşteri faturaları |
| **Depo** | Depo Memuru | MIGO, LT01, VL02N, MB51 | Mal hareketleri, toplama, sevkiyat |
| **Kalite** | Kalite Muayenecisi | QE01, QE02, QM01, QC01 | Muayeneler, kalite bildirimleri |
| **Bakım** | Teknisyen | IW31, IW32, IW37N, IP30 | İş emirleri, bakım uygulaması |
| **İK** | İK Uzmanı | PA30, PA40, PC00, CAT2 | Personel yönetimi, bordro, mesai |

---

## Üretim Tipleri ve Stratejileri

### Üretim Stratejisi Konfigürasyonu

#### 1. Stoka Üretim (MTS)
```
Konfigürasyon:
- Strateji Grubu: 10
- MRP Tipi: PD
- Tüketim Modu: İleri tüketim
- Planlama Stratejisi: Stoka üretim
- Üretim Modu: Seri üretim

Süreç Akışı:
Tahmin → MRP Çalışması → Üretim Emirleri → Üretim → Stok

T-Kodları:
- MD61: Bağımsız gereksinimler oluştur
- MD02: MRP çalışması
- CO01: Üretim emri oluştur
- CO15: Onayla ve stoka al

İş Kullanımı: Yüksek hacimli standart ürünler
Örnek: Standart laptop modelleri
```

#### 2. Siparişe Üretim (MTO)
```
Konfigürasyon:
- Strateji Grubu: 20
- MRP Tipi: PD
- Kalem Kategorisi: TAK
- Bireysel/Toplu Gereksinimler: Bireysel
- Stok tutulmaz

Süreç Akışı:
Satış Siparişi → Gereksinimler → Üretim Emri → Üretim → Doğrudan Teslimat

T-Kodları:
- VA01: Satış siparişi oluştur (gereksinim tetikler)
- CO01: Özel müşteri için üretim emri oluştur
- CO15: Onayla ve doğrudan teslim et

İş Kullanımı: Özelleştirilmiş ürünler
Örnek: Özel konfigürasyonlu oyun laptopları
```

#### 3. Mühendislik-Siparişe (ETO)
```
Konfigürasyon:
- Strateji Grubu: 50
- Özel tedarik: Proje bazlı
- Maliyet toplama için WBS elemanları
- Mühendislik BOM açılımı

Süreç Akışı:
Müşteri Spesifikasyonu → Mühendislik → BOM Oluşturma → Üretim → Teslimat

T-Kodları:
- CJ01: Proje oluştur
- CS01: Mühendislik BOM'u oluştur
- CA01: Rota oluştur
- CO01: Üretim emri oluştur

İş Kullanımı: Benzersiz mühendislik ürünleri
Örnek: Endüstriyel otomasyon sistemleri
```

#### 4. Montaj-Siparişe (ATO)
```
Konfigürasyon:
- Strateji Grubu: 40
- Varyant BOM konfigürasyonu
- Bileşen seviyesinde planlama
- Siparişte final montaj

Süreç Akışı:
Varyantlı Satış Siparişi → Bileşen Montajı → Final Konfigürasyon → Teslimat

T-Kodları:
- VA01: Varyant konfigürasyonlu satış siparişi
- CU41: Varyant konfigürasyonu
- CO01: Montaj emri
- CO15: Final montaj onayı

İş Kullanımı: Seçenekli modüler ürünler
Örnek: Farklı CPU/RAM kombinasyonlu sunucular
```

### Gelişmiş Üretim Senaryoları

#### Tekrarlı Üretim
```
Kurulum:
- REM profil konfigürasyonu
- Geri tüketim kurulumu
- Üretim versiyonları
- Oran bazlı planlama

T-Kodları:
- C223: Üretim versiyonları oluştur
- MF50: Ana reçete oluştur
- MFBF: Geri tüketim işleme
- MF60: Üretim lot işleme

Süreç: Otomatik malzeme tüketimi ile sürekli üretim
```

#### Proses Üretimi
```
Kurulum:
- Üretim emirleri yerine proses emirleri
- Reçete yönetimi
- Yan ürün işleme
- Lot yönetimi

T-Kodları:
- COR1: Proses emri oluştur
- C201: Ana reçete oluştur
- MSC1N: Lot oluştur
- COR6N: Proses emri onayı

Süreç: Kimya/gıda endüstrileri için reçete güdümlü üretim
```

---

## Transfer Tipleri ve Şirketlerarası Süreçler

### Stok Taşıma Emirleri (STO)

#### Standart STO Süreci
```
Amaç: Aynı şirket kodu içinde tesisler arası malzeme transferi

Süreç Akışı:
Gönderen Tesis → STO Oluşturma → Sevkiyat → Transit Stok → Alan Tesis

Adım 1: STO Oluşturma
┌─────────────────────────────────────────────────────────────┐
│ T-Kodu: ME21N (Belge Tipi: UB)                            │
│ Tedarikçi Tesis: 1000                                      │
│ Alan Tesis: 2000                                           │
│ Hareket Tipi: 351 (Sevkiyat) / 101 (Alma)                │
│ Stok Transferi: Doğrudan veya transit stok üzerinden      │
└─────────────────────────────────────────────────────────────┘

Adım 2: Sevkiyat Süreci
┌─────────────────────────────────────────────────────────────┐
│ T-Kodu: VL10B (STO için Teslimat Oluştur)                 │
│ Süreç: Malzemeleri topla, paketle ve sevk et              │
│ Hareket: 351 (Transit stok)                               │
│ Sonuç: Transit stokta malzeme                             │
└─────────────────────────────────────────────────────────────┘

Adım 3: Alma Süreci
┌─────────────────────────────────────────────────────────────┐
│ T-Kodu: MIGO (Mal Kabul)                                  │
│ Hareket Tipi: 101 (Transferden alma)                      │
│ Sonuç: Alan tesiste malzeme mevcut                        │
│ Otomatik: Transit stok temizlendi                         │
└─────────────────────────────────────────────────────────────┘
```

### Şirketlerarası Transferler

#### Satış Fiyatında Transfer (TAS)
```
Amaç: Kar tanıma ile şirketlerarası satış

Konfigürasyon:
- Şirketlerarası fatura belge tipi: IV
- Transfer fiyatlandırması: Pazar bazlı fiyatlandırma
- Her iki şirket için kar merkezi ataması

Süreç:
Şirket A (Satıcı) → Transfer Emri → Şirket B (Alıcı)

Mali Etki:
Şirket A:
Borç Şirketlerarası Alacaklar
Alacak Şirketlerarası Gelir
Borç Satılan Malın Maliyeti
Alacak Mamul Ürünler

Şirket B:
Borç Hammadde/Stok
Alacak Şirketlerarası Borçlar

T-Kodları:
- VF01: Şirketlerarası faturalandırma oluştur
- FB01: Manuel şirketlerarası kayıtlar
- F-53: Şirketlerarası bakiyeleri kapat
```

#### Maliyette Transfer (TAC)
```
Amaç: Kar olmadan masraf yeri yeniden dağıtımı

Konfigürasyon:
- Kar marjı olmayan fiyatlandırma
- Masraf yeri kayıtları
- Hizmet tipi faturalandırma

Süreç:
Masraf Yeri A → Hizmet Dağıtımı → Masraf Yeri B

Mali Etki:
Gönderen:
Borç Masraf Yeri (Alacak)
Alacak Hizmet Geliri

Alan:
Borç Masraf Yeri (Borç)
Alacak Hizmet Maliyetleri

T-Kodları:
- KB11N: Manuel masraf yeri nakli
- KSV5: Masraf yeri dağıtımı
- F-22: Hizmet faturalandırması
```

### Gelişmiş Transfer Senaryoları

#### Konsinye İşleme
```
Konsinye Stok Yönetimi:

Tedarikçi Konsinye:
- Malzeme tüketilene kadar tedarikçiye ait
- Kullanılana kadar ödeme yok
- Hareket Tipleri: 501 (Giriş), 411 (Çekme)

Müşteri Konsinye:
- Müşteri tüketene kadar şirkete ait malzeme
- Tüketimde faturalandırma
- Hareket Tipleri: 631 (Çıkış), 632 (İade)

T-Kodları:
- ME21N: Konsinye siparişi (Kalem Kategorisi K)
- MIGO: Konsinye hareketleri
- MRKO: Konsinye kapatma
```

#### Alt Yüklenici
```
Alt Yüklenici Süreci:

Amaç: İşleme için tedarikçiye malzeme gönderme
Konfigürasyon: Özel tedarik tipi 30
Süreç: Gönderilen malzemeler → İşleme → İade edilen mamul ürünler

T-Kodları:
- ME21N: Alt yüklenici siparişi (Kalem Kategorisi L)
- ME2O: Alt yüklenici kokpiti
- MIGO: Malzeme gönderme ve alma
- ML81N: Alt yüklenici kapatma
```

### Şirket Kodları Arası İşlemler

#### Şirketlerarası Satış Siparişi İşleme
```
İki Aşamalı Süreç:
1. Satıcı şirkette satış siparişi
2. Alıcı şirkette satın alma siparişi

Konfigürasyon:
- Şirketlerarası müşteri ana verisi
- Şirketlerarası tedarikçi ana verisi
- Otomatik sipariş oluşturma

Süreç Akışı:
Satıcı Şirket: VA01 (Satış Siparişi) → VF01 (Faturalandırma)
Alıcı Şirket: ME21N (Sipariş) → MIGO (MK) → MIRO (FG)

Kapatma:
- Otomatik kapatma kayıtları
- Şirketlerarası kar eliminasyonu
- Konsolide raporlama ayarlamaları
```

---

## Entegrasyon Noktaları

### Gerçek Zamanlı Entegrasyon İzleme

#### SD-FI Entegrasyon Noktaları
```
Entegrasyon Tetikleyicisi: Satış siparişi oluşturma
Veri Akışı: SD → FI
Süreç: Gelir tanıma kurulumu

Ana Entegrasyon Noktaları:
1. Müşteri ana veri senkronizasyonu
2. Kredi limiti kontrolü
3. Gelir hesabı belirleme
4. Vergi hesaplama ve nakli
5. Ödeme şartları transferi

İzleme T-Kodları:
- VKM4: Kredi yönetimi genel bakış
- FB03: SD'den FI belgelerini görüntüle
- V/08: Fiyatlandırma prosedürü konfigürasyonu
- OVK1: Hesap belirleme konfigürasyonu
```

#### MM-FI Entegrasyon Noktaları
```
Entegrasyon Tetikleyicisi: Mal hareketleri
Veri Akışı: MM → FI
Süreç: Stok değerleme ve maliyet nakli

Ana Entegrasyon Noktaları:
1. Satın alma siparişi taahhütleri
2. Mal kabul stok nakli
3. Fatura girişi borç yükümlülüğü
4. Ödeme işleme
5. Stok yeniden değerleme

Hareket Tipleri ve Mali Etki:
- 101 (MK): Borç Stok, Alacak MK/FG Kliring
- 102 (İptal): Yukarıdaki kayıtları tersine çevir
- 201 (Çıkış): Borç Tüketim Hesabı, Alacak Stok
- 261 (Emire Çıkış): Borç Üretim Emri, Alacak Stok

İzleme T-Kodları:
- MB51: Malzeme belge listesi
- FB03: Muhasebe belgelerini görüntüle
- OBYC: Otomatik hesap belirleme
- MMRV: Malzeme yeniden değerleme
```

#### PP-CO Entegrasyon Noktaları
```
Entegrasyon Tetikleyicisi: Üretim emri işleme
Veri Akışı: PP → CO
Süreç: Ürün maliyetlendirme ve maliyet dağıtımı

Ana Entegrasyon Noktaları:
1. Üretim emri maliyet toplama
2. Masraf yerlerine faaliyet dağıtımı
3. Genel gider dağıtımı
4. Varyans hesaplama
5. Maliyet nesnesi kapatma

Maliyet Akışı:
Malzeme Maliyetleri → Üretim Emri → Masraf Yerleri → Ürün Maliyetleri

İzleme T-Kodları:
- CO03: Üretim emri maliyetlerini görüntüle
- KSB1: Masraf yeri kalem detayları
- CK40N: Ürün maliyet tahmini
- CO40: Emir kapatma
- KKBC: Maliyet nesnesi kontroling kokpiti
```

#### QM Tüm Modüllerle Entegrasyon
```
QM-MM Entegrasyonu:
- Mal kabulünde muayene lotu oluşturma
- Stokta kalite blokları
- Tedarikçi kalite değerlendirme

QM-PP Entegrasyonu:
- Süreç içi muayene
- Üretimde kalite tutma
- Sertifika gereksinimleri

QM-SD Entegrasyonu:
- Teslimat için kalite sertifikaları
- Müşteri kalite bildirimleri
- İade işleme

T-Kodları:
- QE51N: Muayeneli mal kabul
- QA32: Üretim lotu muayenesi
- QC01: Kalite sertifikası oluşturma
- QM03: Kalite bildirimini görüntüle
```

### Sistem Entegrasyon Mimarisi

#### Veri Akışı Mimarisi
```
Gerçek Zamanlı Entegrasyon:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│       SD        │───▶│       FI        │◀───│       MM        │
│ Satış Siparişleri│    │ Mali Belgeler   │    │ Satın Alma Siparişleri│
│ Fatura Belgeleri │    │ Genel Muhasebe  │    │ Mal Hareketleri │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │              ┌─────────────────┐              │
         └─────────────▶│       CO        │◀─────────────┘
                        │ Masraf Yerleri  │
                        │ Maliyet Nesneleri│
                        └─────────────────┘
                                 │
                        ┌─────────────────┐
                        │       PP        │
                        │ Üretim          │
                        │ Emirleri        │
                        └─────────────────┘

Toplu Entegrasyon (Gecelik):
- Masraf yeri dağıtımları
- Genel gider hesaplamaları
- Mali konsolidasyon
- Veri ambarı güncellemeleri
```

#### Entegrasyon İzleme Panosu
```
Sistem Entegrasyon Sağlık Kontrolü:

Arayüz Durumu:
┌────────────────────────────────────────────────────────────┐
│ Arayüz Adı              │ Durum  │ Son Çalışma │ Kayıtlar  │
├────────────────────────────────────────────────────────────┤
│ SD-FI Gelir Arayüzü     │ ✅ İYİ │ 2 dk önce   │ 1,247     │
│ MM-FI Stok Arayüzü      │ ✅ İYİ │ 1 dk önce   │ 3,456     │
│ PP-CO Maliyet Arayüzü   │ ⚠️ UYARI│ 15 dk önce  │ 892       │
│ HCM-CO İşçilik Arayüzü  │ ✅ İYİ │ 5 dk önce   │ 2,134     │
│ QM-Tümü Kalite Arayüzü  │ ✅ İYİ │ 3 dk önce   │ 45        │
└────────────────────────────────────────────────────────────┘

Performans Metrikleri:
- Entegrasyon Başarı Oranı: %99.7
- Ortalama İşleme Süresi: 45 saniye
- Hata Çözüm Süresi: < 15 dakika
- Günlük İşlem Hacmi: 125,000
```

---

## Temel Performans Göstergeleri (KPI'lar)

### Süreç Performans Metrikleri

#### Teklif-Tahsilat KPI'ları
```
Metrik                     | Hedef    | Mevcut   | Durum
────────────────────────────────────────────────────
Teklif Yanıt Süresi        | < 24 saat| 18 saat  | ✅ İyi
Sipariş-Teslimat Süresi    | < 10 gün | 8.5 gün  | ✅ İyi
Fatura Doğruluğu          | > %99    | %99.3    | ✅ İyi
Tahsilat Süresi           | < 45 gün | 42 gün   | ✅ İyi
Müşteri Memnuniyeti       | > 4.5/5  | 4.7/5    | ✅ Mükemmel
```

#### Satın Al-Öde KPI'ları
```
Metrik                     | Hedef    | Mevcut   | Durum
────────────────────────────────────────────────────
Sipariş İşleme Süresi      | < 2 gün  | 1.5 gün  | ✅ İyi
3'lü Eşleştirme Oranı     | > %95    | %97.2    | ✅ İyi
Erken Ödeme İndirimi      | > %85    | %88      | ✅ İyi
Tedarikçi Zamanında Teslimat| > %95   | %96.8    | ✅ İyi
Süreç Dışı Satın Alma Oranı| < %5    | %3.2     | ✅ İyi
```

#### Planla-Üret KPI'ları
```
Metrik                     | Hedef    | Mevcut   | Durum
────────────────────────────────────────────────────
Üretim Program Uyumu      | > %95    | %93.5    | ⚠️ Uyarı
İlk Geçiş Verimi          | > %98    | %98.7    | ✅ İyi
Genel Ekipman Etkinliği   | > %85    | %87.3    | ✅ İyi
Stok Devir Hızı           | > 12     | 14.2     | ✅ Mükemmel
Üretim Maliyet Varyansı   | < ±%3    | +%1.8    | ✅ İyi
```

### Sistem Performans Metrikleri

#### Teknik KPI'lar
```
Sistem Erişilebilirliği   | Hedef    | Mevcut   | Durum
────────────────────────────────────────────────────
SAP Sistem Çalışma Süresi | > %99.5  | %99.8    | ✅ Mükemmel
Yanıt Süresi (Diyalog)    | < 1 sn   | 0.7 sn   | ✅ İyi
Yanıt Süresi (Raporlar)   | < 10 sn  | 8.2 sn   | ✅ İyi
Toplu İş Başarı Oranı     | > %99    | %99.5    | ✅ İyi
Veri Entegrasyon Başarısı | > %99    | %99.7    | ✅ Mükemmel
```

---

## Sonuç

Bu belge SAP iş süreçleri, organizasyonel roller ve sistem entegrasyon noktalarının kapsamlı bir genel bakışını sağlar. Etkili SAP operasyonları için anahtar başarı faktörleri şunlardır:

1. **Net Rol Tanımı**: Her rolün özel sorumlulukları ve uygun sistem erişimi vardır
2. **Süreç Standardizasyonu**: Organizasyon genelinde tutarlı iş süreci yürütme
3. **Entegrasyon Mükemmelliği**: Modüller arası sorunsuz veri akışı veri doğruluğunu sağlar
4. **Sürekli İzleme**: Düzenli performans takibi ve iyileştirme girişimleri
5. **Değişim Yönetimi**: Sistem güncellemeleri ve süreç iyileştirmelerinin uygun işlenmesi

Başarılı SAP uygulama ve operasyonları için organizasyonlar şunlara odaklanmalıdır:
- Role özel T-kodları konusunda kapsamlı kullanıcı eğitimi
- Düzenli süreç incelemeleri ve optimizasyon
- Entegrasyon noktalarının proaktif izlenmesi
- Performans ölçümü ve iyileştirme
- Dokümantasyon bakımı ve güncellemeleri

Bu çerçeve SAP mükemmelliği için bir temel oluşturur ve özel endüstri gereksinimlerine ve organizasyonel yapılara uyarlanabilir.