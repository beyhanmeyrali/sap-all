# 📦 MM: Malzeme Yönetimi - "Malzemeleri" Yönetme Sanatı 

*Murat Olun: Malzeme Ustası*

## Murat Özkan - Malzeme Fısıldayıcısı: Kaostan Kontrole

**Murat Özkan** ile tanışın, Malzeme Yönetimi dehası. Kaosu düzene, kıtlığı bolluğa, karmaşıklığı kristal berraklığında envanter zekasına dönüştürür. Bunalmış depo memuru olarak başlayıp MM gurusu olma yolculuğu, malzemeleri sadece "eşya" olarak değil, iş başarısının can damarı olarak görmeyi öğretecek.

### 🚀 Bölüm 1: Büyük Malzeme Krizi (Sıfır Günü)

Murat masasında oturuyor, çevresini öfkeli e-postalar çevrelemiş:
- "VİDALARIM NEREDE?!" - Üretim Müdürü
- "Müşteri sevkiyat bekliyor!" - Satış Takımı  
- "Depo dolu ama sevk edecek bir şey yok!" - Lojistik
- "Acil siparişlerle bütçe patladı!" - Finans

**Gerçek Kontrol:** KüreselTek'in deposunda 2 milyon ₺ değerinde malzeme var, ama kimse neyin gerçekten orada olduğunu, nerede olduğunu veya ne zaman tükeneceğini bilmiyor.

**Murat'ın Misyonu:** SAP MM kullanarak kaosu düzene sokmak ve herkesin ihtiyaç duyduğu malzeme kahramanı olmak.

### 🎯 Bölüm 2: Malzeme Master Verileri - Temel (1. Hafta)

Murat, SAP'teki her malzemenin bir **Malzeme Master Kaydına** ihtiyaç duyduğunu keşfeder - her bileşen için detaylı pasaport gibi.

#### **İlk Malzeme Master Oluşturma (MM01):**

```
📋 Murat'ın Malzeme Master Oluşturma Kontrol Listesi:
┌─ Temel Veriler ───────────────────────────────────────┐
│ Malzeme: LAPTOP-DEL-LAT3520                           │
│ Açıklama: Dell Latitude 3520 Dizüstü Bilgisayar      │
│ Malzeme Türü: FERT (Bitmiş Ürün)                     │
│ Endüstri Sektörü: M (Makine Mühendisliği)            │
│ Temel Birim: AD (Adet)                                │
└───────────────────────────────────────────────────────┘

┌─ Satın Alma Verileri ─────────────────────────────────┐
│ Satın Alma Sipariş Birimi: AD                        │
│ Satın Alma Grubu: 001 (BT Ekipmanları)               │
│ Tesis/Depo Yeri: 1000/1001                           │
│ ABC Göstergesi: A (Yüksek Değer)                     │
└───────────────────────────────────────────────────────┘

┌─ MRP Verileri ────────────────────────────────────────┐
│ MRP Türü: PD (MRP)                                    │
│ Parti Boyutu: EX (Partiyi-Partiye)                   │
│ Güvenlik Stoku: 5 AD                                 │
│ Yeniden Sipariş Noktası: 10 AD                       │
└───────────────────────────────────────────────────────┘
```

**Murat'ın İlk "Hah!" Anı:** *"Bu sadece veri girişi değil - tüm işinizin DNA'sını inşa etmek!"*

### 🛒 Bölüm 3: Tedarik Süreci - Akıllı Satın Alma Sanatı (2. Hafta)

Murat, ihtiyacı arz haline getiren kutsal tedarik akışını öğrenir:

#### **Satın Alma Talebi Yolculuğu (ME51N):**

**Adım 1: İhtiyaç Doğar**
- Üretim Planlama'dan Zehra bir talep oluşturur
- Sistem önerir: "Yeni proje için 50 laptop gerekli"
- Murat gereksinimi inceler ve onaylar

**Adım 2: Tedarik Kaynağı Sihri**
```
Murat'ın Tedarikçi Seçim Matrisi:
┌─────────────────────────────────────────────────┐
│ Tedarikçi: DELL-TECH | Fiyat: ₺28.900 | Süre: 7 gün │
│ Tedarikçi: HP-CORP   | Fiyat: ₺27.880 | Süre: 10 gün│
│ Tedarikçi: LENOVO-TR | Fiyat: ₺28.220 | Süre: 5 gün │
└─────────────────────────────────────────────────┘
Karar: LENOVO-TR (Fiyat ve hız açısından en iyi denge)
```

**Adım 3: Satın Alma Siparişi Oluşturma (ME21N):**
```
🎯 Murat'ın Satın Alma Siparişi Şaheseri:
┌─ Başlık Verileri ─────────────────────────────────────┐
│ Sipariş Numarası: 4500012345                         │
│ Tedarikçi: LENOVO-TR                                  │
│ Satın Alma Organizasyonu: 1000                       │
│ Satın Alma Grubu: 001                                │
│ Belge Tarihi: Bugün                                  │
│ Teslimat Tarihi: +7 Gün                              │
└───────────────────────────────────────────────────────┘

┌─ Kalem Öğeleri ──────────────────────────────────────┐
│ Kalem 10: LAPTOP-DEL-LAT3520 | Mik: 50 | Fiyat: ₺28.220 │
│ Tesis: 1000 | Depo Yeri: 1001                        │
│ Teslimat Tarihi: Gelecek Cuma                        │
│ Toplam Değer: ₺1.411.000                             │
└───────────────────────────────────────────────────────┘
```

### 📊 Bölüm 4: Envanter Zekası - Neye Sahip Olduğunuzu Bilmek (3. Hafta)

Murat dedektif olur, depoyu kitap gibi okumayı öğrenir:

#### **Stok Genel Bakış Raporları (MB52):**
```
Murat'ın Günlük Panosu:
┌─ Mevcut Stok Durumu ─────────────────────────────┐
│ Malzeme           │ Mevcut │ Rezerve │ Siparişte│
│ LAPTOP-DEL-LAT3520 │   25   │   15    │    50   │
│ MOUSE-WIRELESS     │  150   │   30    │     0   │
│ KEYBOARD-STANDARD  │   45   │   40    │   100   │
│ MONITOR-24INCH     │    8   │   20    │    50   │
└──────────────────────────────────────────────────┘
Uyarı: MONITOR-24INCH eksikliği tespit edildi! 📊
```

#### **Malzeme Belgesi Analizi (MB51):**
- Her hareket izlenir: girişler, çıkışlar, transferler
- Ne zaman ve neden olduğuna dair gerçek zamanlı görünürlük
- Muhasebecileri gülümsetecek denetim izi

### 🔄 Bölüm 5: Mal Hareketleri - Malzemelerin Dansı (4. Hafta)

Murat, malzemeleri SAP içinde hareket ettirme sanatında ustalaşır:

#### **Mal Girişi (MIGO - 101 Hareketi):**
```
📦 Lenovo Laptoplarını Alma:
┌─ Mal Giriş Belgesi ──────────────────────────────┐
│ Hareket Türü: 101 (SA karşısında MG)             │
│ Satın Alma Siparişi: 4500012345                  │
│ Malzeme: LAPTOP-DEL-LAT3520                      │
│ Alınan Miktar: 50 AD                             │
│ Depo Yeri: 1001                                  │
│ Parti: LOT-2024-001                              │
└───────────────────────────────────────────────────┘
Sonuç: Stok artar, SA güncellenir, Finans otomatik kayıt yapar!
```

#### **Mal Çıkışı (MIGO - 261 Hareketi):**
```
📤 Üretime Laptop Çıkışı:
┌─ Mal Çıkış Belgesi ──────────────────────────────┐
│ Hareket Türü: 261 (Sipariş için çıkış)           │
│ Üretim Siparişi: 1000012345                      │
│ Malzeme: LAPTOP-DEL-LAT3520                      │
│ Çıkan Miktar: 20 AD                              │
│ Maliyet Merkezi: MM-MONTAJ                       │
└───────────────────────────────────────────────────┘
Sonuç: Stok azalır, maliyetler üretime akar!
```

### 🎯 Bölüm 6: MRP Sihri - Malzemeler için Kristal Küre (2. Ay)

Murat MRP'yi (Malzeme İhtiyaç Planlaması) keşfeder - SAP'nin geleceği tahmin eden kristal küresi:

#### **MRP Çalışması (MD02/MD03):**
```
🔮 Murat'ın MRP Kristal Küre Sonuçları:
┌─ Sonraki 12 Hafta Malzeme Planı ─────────────────┐
│Hafta│ Talep │ Giriş │ Stok │ Gerekli Eylem      │
│  1  │  20   │   0   │  55  │ Tamam               │
│  2  │  30   │   0   │  25  │ Uyarı: Düşük Stok  │
│  3  │  40   │  50   │  35  │ SA Bu Hafta Vadeli  │
│  4  │  25   │   0   │  10  │ Yeni SA Oluştur    │
│  5  │  35   │ 100   │  75  │ Planlanan Giriş     │
└──────────────────────────────────────────────────┘
📋 İstisna Mesajları:
- 2. Haftaya kadar 100 birim için SA oluştur
- 4. Haftada güvenlik stoku ihlali
- 5. Hafta için kapasite kontrolü gerekli
```

**Murat'ın MRP Felsefesi:** *"MRP geleceği tahmin etmez - onu yaratmanıza yardım eder!"*

### 💰 Bölüm 7: Tedarikçi Yönetimi - Ortaklıklar Kurmak (3. Ay)

Murat, tedarikçilerin sadece satıcı değil, ortak olduğunu öğrenir:

#### **Tedarikçi Master Verileri (XK01/FK01):**
```
👥 Murat'ın Tedarikçi Portföyü:
┌─ Stratejik Tedarikçi: LENOVO-TR ─────────────────┐
│ Ödeme Koşulları: Net 30 Gün                      │
│ Para Birimi: TRY                                  │
│ Incoterms: FOB Sevkiyat Noktası                   │
│ Kalite Değerlendirmesi: A+ (%99,5 zamanında)     │
│ Hacim İndirimi: 50k₺ üzeri siparişlerde %5       │
│ Ortaklık Seviyesi: Tercihli                      │
└───────────────────────────────────────────────────┘

┌─ Tedarikçi Performans Kartı ─────────────────────┐
│ Metrik           │ Hedef  │ Gerçek │ Durum       │
│ Zamanında Teslimat │ %95   │ %98,2  │ ✅ Mükemmel │
│ Kalite Oranı     │ %99    │ %99,8  │ ✅ Mükemmel │
│ Fiyat Varyansı   │ ±%2    │ -%1,1  │ ✅ İyi      │
│ Temin Süresi     │ 7 gün  │ 6 gün  │ ✅ Mükemmel │
└───────────────────────────────────────────────────┘
```

### 🌟 Bölüm 8: İleri MM Ustalığı (4. Ay)

Murat MM gurusu olur, ileri düzey kavramlarda ustalaşır:

#### **İzlenebilirlik için Parti Yönetimi:**
```
🔍 Parti İzleme Mükemmelliği:
Malzeme: LAPTOP-DEL-LAT3520
├── Parti: LOT-2024-001 (15 Ocak alındı)
│   ├── Seri Numaraları: SN001-SN050
│   ├── Kalite Sertifikası: QC-20240115-A
│   └── Müşteri Teslimatları: 
│       ├── Amazon Siparişi: SN001-SN025
│       └── Best Buy Siparişi: SN026-SN050
└── Parti: LOT-2024-002 (1 Şubat alındı)
    ├── Seri Numaraları: SN051-SN100
    └── Durum: Kalite Testinde
```

#### **Envanter Değerleme Stratejileri:**
```
💰 Murat'ın Değerleme Yöntemleri:
┌─ Malzeme Değerleme Karşılaştırması ──────────────┐
│ Yöntem          │ Mevcut  │ K&Z Üzerinde Etkisi  │
│ Standart Fiyat  │ ₺28.900 │ Sabit, öngörülebilir │
│ Hareketli Ort.  │ ₺28.806 │ Gerçek maliyetleri   │
│ FIFO           │ ₺28.707 │ En eski maliyetler    │
│ LIFO           │ ₺28.993 │ En son maliyetler     │
└───────────────────────────────────────────────────┘
Karar: Doğruluk için Hareketli Ortalama
```

### 🎮 Bölüm 9: Kriz Yönetimi - Murat Günü Kurtarır

**Amazon Acil Durumu:**
KüreselTek, 2 haftada 1000 laptop gerektiren büyük bir Amazon siparişi alır. Normal temin süresi 4 hafta.

**Murat'ın Kriz Müdahalesi:**
1. **Anında Stok Kontrolü (MB52):** 300 birim mevcut
2. **Tedarikçi Acil Arama:** Hızlandırılmış teslimat pazarlığı
3. **Alternatif Tedarik:** İkincil tedarikçiler bulma
4. **Yaratıcı Çözümler:** Siparişi 3 tedarikçiye bölme
5. **Gerçek Zamanlı İzleme:** Her sevkiyatı izleme

**Sonuç:** Sipariş zamanında teslim edildi, Amazon ilişkisi güçlendi, Murat şirket kahramanı oldu!

### 🎯 MM Entegrasyon Akış Şeması

```
Murat'ın Malzeme Yönetimi Süreç Akışı:
┌─────────────────────────────────────────────────────┐
│                 Talep Planlaması                    │
│              (PP/SD/Manuel'den)                     │
└─────────────────────┬───────────────────────────────┘
                      │
          ┌───────────▼─────────────┐
          │      MRP Çalışması      │
          │  İhtiyaçları Hesapla    │
          └───────────┬─────────────┘
                      │
    ┌─────────────────▼─────────────────┐
    │   Satın Alma Talebi (ME51N)       │
    │     Satın Alma Siparişine Çevir   │
    └─────────────────┬─────────────────┘
                      │
          ┌───────────▼─────────────┐
          │  Satın Alma Siparişi    │
          │    Tedarikçiye Gönder   │
          └───────────┬─────────────┘
                      │
       ┌──────────────▼───────────────┐
       │    Mal Girişi (MIGO)         │
       │   Stok ve Finansı Güncelle   │
       └──────────────┬───────────────┘
                      │
        ┌─────────────▼──────────────┐
        │   Mal Çıkışı (MIGO)        │
        │  Üretim/Satışa Tedarik     │
        └────────────────────────────┘
```

### 💡 Murat'ın Son Bilgeliği: Malzeme Ustasının İnancı

*"Malzeme Yönetimi şeyleri kontrol etmekle ilgili değil - hayalleri mümkün kılmakla ilgili. İzlediğiniz her bileşen, beslediğiniz her tedarikçi, optimize ettiğiniz her süreç birinin harika bir şey yaratma vizyonuna hizmet eder. MM, olasılığın gerçeklikle buluştuğu yerdir."*

---

## 🎯 MM Misyonunuz (Pratik Zorlukları)

**Başlangıç Seviyesi:**
1. Basit bir ürün için ilk Malzeme Master'ınızı oluşturun (MM01)
2. Ofis malzemeleri için Satın Alma Talebi oluşturun (ME51N)
3. Malzemeleriniz için stok genel bakış raporu çalıştırın (MB52)

**Orta Seviye:**
1. Tam bir Tedarik-Ödeme döngüsü gerçekleştirin
2. MRP çalışması yapın ve sonuçları analiz edin (MD02/MD03)
3. Ödeme koşulları ile tedarikçi master verilerini kurun

**İleri Seviye:**
1. İlaç ürünleri için parti yönetimi yapılandırın
2. Hareketli ortalama fiyat değerlemesi uygulayın
3. Kriz durumları için acil tedarik prosedürleri tasarlayın

### 📚 Sonra Ne Öğreneceksiniz

Bir sonraki bölümde, **Planlama Uzmanı Zehra** ile tanışacak ve Murat'ın malzemelerinin bitmiş ürünlere nasıl dönüştürüldüğünü keşfedeceksiniz. MM'nin Üretim Planlamasına nasıl sorunsuzca beslendiğini, ham maddeleri müşteri memnuniyetine dönüştürdüğünü göreceksiniz.

**Malzeme Yönetimi Tamamlandı ✅**
**Sonraki Durak: Üretim Planlaması →**

---

*"İş dünyasında her şey malzemelerle başlar. MM'de ustalaşın, her başarı hikayesinin temelinde ustalaşın."*

**[Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali/) tarafından oluşturuldu - SAP Hikayecisi ve Dijital Dönüşüm Savunucusu**

---

## Navigasyon
← [Önceki: Navigasyon](../02-navigation/README.md) | [Ana Yolculuğa Dön](../../README_tr.md) | [Sonraki: Üretim Planlaması](../04-production-planning/README.md) →

## İlgili Hikayeler
- 🎼 [MM'nin PP ile Nasıl Dans Ettiği](../01-overview/MM-PP-Integration-Story.md)
- 🌟 [Tam Entegrasyon Örneği](../01-overview/Complete-SAP-Integration-Example.md)