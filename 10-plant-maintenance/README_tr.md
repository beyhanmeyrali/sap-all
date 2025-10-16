# 🔧 PM: Tesis Bakımı - Merve'nin Ekipman Ustalığı Yolculuğu

*Reaktif İtfaiyeciden Proaktif Bakım Stratejistine*

---

## Merve ile Tanışın - Makineleri Dinleyen Bakım Mühendisi

Merve, Amazon sipariş karşılama ekipmanlarının artan yoğunlukla çalıştığı üretim hattına bakıyor. KüreselTek'in Tesis Bakım Mühendisi olarak, reaktif yangın söndürücülükten proaktif bakım ustalığına dönüşüme tanık oldu. İlk günlerinde, Merve'nin telefonu acil arızalarla her saatte çalardı. Bugün ise, sorunları meydana gelmeden önce engelliyor ve SAP PM'i kullanarak öngörülü bakımın senfonisini yönetiyor.

*"Her makinenin kalp atışı vardır,"* diyor Merve, bakım panosunu kontrol ederken. *"SAP PM bu kalp atışını dinlememize ve atlamadan önce yanıt vermemize yardımcı oluyor."*

---

## Bölüm 1: Ekipman Ekosistemi Temeli

### Merve'nin Ekipman Ana Veri Mimarisi

Merve'nin ilk dersi, etkili bakımın ekipmanlarınızı içten dışa tamamen tanımakla başladığını anlamaktı.

```
SAP PM'DE EKİPMAN HİYERARŞİSİ
═══════════════════════════════════════

Tesis
├── Alan (Üretim Bölgesi)
│   ├── Sistem (Karşılama Hattı)
│   │   ├── Montaj (Konveyör Sistemi)
│   │   │   ├── Ekipman (Motor #001)
│   │   │   ├── Ekipman (Bant #001)
│   │   │   └── Ekipman (Sensör #001)
│   │   └── Montaj (Sıralama Sistemi)
│   │       ├── Ekipman (Tarayıcı #001)
│   │       └── Ekipman (Robot Kol #001)
│   └── Sistem (Kalite Kontrol)
└── Alan (Depo Bölgesi)
```

**Ekipman Yönetimi için Temel T-Kodları:**
- **IE01**: Ekipman Ana Kartı Oluştur
- **IE02**: Ekipman Ana Kartı Değiştir  
- **IE03**: Ekipman Ana Kartı Görüntüle
- **IH01**: Fonksiyonel Konum Oluştur
- **IH02**: Fonksiyonel Konum Değiştir
- **IH03**: Fonksiyonel Konum Görüntüle

Merve ilk kritik ekipmanını oluşturuyor:

**İşlem: IE01 (Ekipman Oluştur)**
```
Ekipman Numarası: EQ-CONV-001
Ekipman Kategorisi: M (Makine)
Açıklama: Ana Konveyör Motor - Karşılama Hattı 1
Fonksiyonel Konum: PL01-PROD-FL1-CONV
Üretici: Siemens
Model Numarası: 1LA7-133-6AA60
Seri Numarası: SM-2024-1001
Kurulum Tarihi: 15 Ocak 2024
Başlatma Tarihi: 20 Ocak 2024
```

---

## Bölüm 2: Bakım Planlama Devrimi

### Kaostan Stratejik Planlamaya

Merve, bakımın tamamen reaktif olduğu eski günleri hatırlıyor. Şimdi, kapsamlı bakım stratejileri oluşturmak için SAP PM kullanıyor.

```
BAKIM STRATEJİSİ ÇERÇEVESİ
═════════════════════════════════════

Strateji Çağrı Nesnesi
├── Bakım Görev Listesi
│   ├── Önleyici Görevler
│   │   ├── Günlük Kontroller
│   │   ├── Haftalık Yağlama
│   │   └── Aylık Kalibrasyon
│   ├── Öngörülü Görevler
│   │   ├── Titreşim Analizi
│   │   └── Sıcaklık İzleme
│   └── Düzeltici Görevler
└── Bakım Planı
    ├── Zamanlama Parametreleri
    ├── Çağrı Ufku
    └── İş Emri Oluşturma
```

**Temel Planlama T-Kodları:**
- **IA01**: Görev Listesi Oluştur
- **IA02**: Görev Listesi Değiştir
- **IA11**: Görev Listesi Oluştur (Genel)
- **IP01**: Bakım Planı Oluştur
- **IP02**: Bakım Planı Değiştir
- **IP10**: Bakım Planı Zamanlaması

Merve kritik konveyör motor için önleyici bakım stratejisi oluşturuyor:

**İşlem: IA01 (Görev Listesi Oluştur)**
```
Görev Listesi Grubu: TL-MOTOR-001
Görev Listesi Türü: E (Ekipman Görev Listesi)
Ekipman: EQ-CONV-001

Operasyonlar:
010 - Görsel Kontrol (Süre: 0.5 saat)
020 - Yağlama Kontrolü (Süre: 0.25 saat)  
030 - Kayış Gerginlik Kontrolü (Süre: 0.25 saat)
040 - Titreşim Ölçümü (Süre: 0.5 saat)
050 - Sıcaklık Kontrolü (Süre: 0.25 saat)

İş Merkezi: MC-MAINT-01
Gerekli Beceriler: Mekanik Bakım Seviye 2
```

---

## Bölüm 3: İş Emri Yönetimi Ustalığı

### Bakım Yürütmenin Senfonisi

Merve bakımı bir şef gibi yönetiyor ve bir orkestrayı yönlendiriyor. Her iş emri, ekipman güvenilirliğinin daha büyük kompozisyonunda dikkatli bir şekilde planlanmış bir harekettir.

```
İŞ EMRİ YAŞAM DÖNGÜSÜ
══════════════════════════════════════

Oluşturma Aşaması
├── Otomatik (Bakım Planından)
├── Manuel (Acil/Düzeltici)
└── Bildirim Tabanlı

Planlama Aşaması
├── Operasyon Detayları
├── Bileşen Gereksinimleri
├── Kapasite Gereksinimleri
└── Zamanlama

Yürütme Aşaması
├── Malzeme Rezervasyonu
├── Zaman Onayı
├── İlerleme Güncellemeleri
└── Tamamlanma Onayı

Kapanış Aşaması
├── Teknik Tamamlanma
├── Maliyet Kapatması
└── Geçmiş Belgelendirmesi
```

**İş Emri Yönetimi T-Kodları:**
- **IW01**: Bakım Emri Oluştur
- **IW02**: Bakım Emri Değiştir
- **IW03**: Bakım Emri Görüntüle
- **IW28**: Bakım Emirlerini Liste Düzenle
- **IW37N**: Emir Yöneticisi (Yeni)
- **IW38**: Emri Değiştir: Gerçek Veriler
- **IW39**: Emri Tamamla

Merve ana konveyör motordaki olağandışı titreşimlerle ilgili acil bir bildirim alıyor:

**İşlem: IW01 (Bakım Emri Oluştur)**
```
Emir Türü: PM01 (Önleyici Bakım)
Ekipman: EQ-CONV-001
Öncelik: 2 (Yüksek)
Açıklama: Anormal Titreşimleri Araştır - Ana Konveyör

Operasyonlar:
010 - Titreşim Analizi (2.0 saat)
    İş Merkezi: MC-MAINT-01
    Personel: 2 Bakım Teknisyeni
    
020 - Motor Kontrolü (1.5 saat)
    İş Merkezi: MC-MAINT-01
    Personel: 1 Kıdemli Teknisyen
    
030 - Düzeltici Eylemler (TBD saat)
    İş Merkezi: MC-MAINT-01
    Bileşenler: Bulgulara göre TBD

Planlanan Başlangıç: Bugün 14:00
Planlanan Bitiş: Bugün 18:00
```

---

## Bölüm 4: Önleyici Bakım Stratejisi

### Ekipman Arızalarının Önünde Kalmak

Merve'nin önleyici bakım programı, uygulama başlangıcından bu yana acil arızaları %75 azalttı. Rutin bakımın oluşturulmasını ve zamanlanmasını otomatikleştirmek için SAP PM kullanıyor.

```
ÖNLEYİCİ BAKIM DÖNGÜLERİ
═══════════════════════════════════════

Günlük Kontroller (Otomatik Oluşturulan)
├── Görsel Kontroller
├── Temel Ölçümler
└── İşletme Parametresi Kayıtları

Haftalık Bakım (Zamanlanmış)
├── Yağlama Hizmetleri
├── Kayış Ayarları
└── Filtre Değişimleri

Aylık Hizmetler (Planlı)
├── Kapsamlı Kontroller
├── Öngörülü Ölçümler
└── Kalibrasyon Aktiviteleri

Üç Aylık Revizyonlar (Koordineli)
├── Ana Bileşen Kontrolleri
├── Performans Optimizasyonu
└── Yükseltme Değerlendirmeleri
```

**Önleyici Bakım T-Kodları:**
- **IP02**: Bakım Planı Değiştir
- **IP15**: Bakım Planı Görüntüle
- **IP17**: Bakım Kalemi Değiştir
- **IP19**: Bakım Kalemi Görüntüle
- **IP24**: Bakım Planı Programı
- **IP30**: Bakım Planları Arşivleme

Merve karşılama ekipmanları için kapsamlı bir bakım planı kuruyor:

**İşlem: IP01 (Bakım Planı Oluştur)**
```
Bakım Planı: MP-CONV-001
Plan Kategorisi: Ekipman Bakım Planı
Bakım Stratejisi: ST-MOTOR-CONV

Bakım Kalemleri:
Kalem 010: Günlük Görsel Kontrol
    Çağrı Nesnesi: EQ-CONV-001
    Görev Listesi: TL-MOTOR-001
    Döngü: 1 Gün
    
Kalem 020: Haftalık Yağlama Hizmeti  
    Çağrı Nesnesi: EQ-CONV-001
    Görev Listesi: TL-MOTOR-002
    Döngü: 7 Gün
    
Kalem 030: Aylık Performans Analizi
    Çağrı Nesnesi: EQ-CONV-001  
    Görev Listesi: TL-MOTOR-003
    Döngü: 30 Gün

Zamanlama Parametreleri:
Erken Başlangıç: Vade tarihinden 2 gün önce
Geç Bitiş: Vade tarihinden 1 gün sonra
```

---

## Bölüm 5: Arıza Bakımı Mükemmelliği

### Önleme Gerçeklikle Buluştuğunda

Merve'nin en iyi önleyici çabalarına rağmen, ekipman bazen beklenmedik şekilde arızalanır. SAP PM ona duruş süresini minimuma indirmek için hızlı ve verimli yanıt vermesine yardımcı oluyor.

```
ARIZA YÖNETİMİ SÜRECİ
══════════════════════════════════════

Bildirim Aşaması
├── Ekipman Arızası Uyarısı
├── Etki Değerlendirmesi
└── Öncelik Ataması

Acil Müdahale
├── Anında Güvenlik Kontrolü
├── Geçici Çözümler
└── Kaynak Mobilizasyonu

Kök Neden Analizi
├── Arıza Araştırması
├── Katkıda Bulunan Faktörler
└── Önleme Stratejileri

Düzeltici Eylemler
├── Anında Onarımlar
├── Uzun Vadeli Düzeltmeler
└── Süreç İyileştirmeleri
```

**Arıza Yönetimi T-Kodları:**
- **IW51**: Bakım Bildirimi Oluştur
- **IW52**: Bakım Bildirimi Değiştir
- **IW53**: Bakım Bildirimi Görüntüle
- **IW56**: Bildirim Listesi
- **IW21**: Arıza Emri Oluştur
- **IW22**: Arıza Emri Değiştir

Merve acil bir çağrı alıyor - ana sıralama tarayıcısı yoğun sipariş işleme sırasında arızalandı:

**İşlem: IW51 (Bakım Bildirimi Oluştur)**
```
Bildirim Türü: M1 (Arıza Raporu)
Ekipman: EQ-SCAN-001
Öncelik: 1 (Çok Yüksek)
Açıklama: Tarayıcı Ünitesi Tam Arızası - Barkod Okuma Yok

Arıza Detayları:
Arıza Başlangıcı: Bugün 09:30
Etki: Üretim hattının tamamen durması
Anında Etki: 500 sipariş/saat kapasite kaybı
Tahmini İş Etkisi: ₺425.000/saat

Gerekli Eylem: Acil değiştirme veya onarım
Güvenlik Hususları: Yok - elektrik güvenlik prosedürleri uygulandı
```

**İşlem: IW21 (Arıza Emri Oluştur)**
```
Emir Türü: PM03 (Arıza Bakımı)
Referans Bildirimi: 100001234
Ekipman: EQ-SCAN-001
Öncelik: 1 (Acil)

Operasyonlar:
010 - Acil Tanı (0.5 saat)
020 - Bileşen Değişimi (2.0 saat)  
030 - Sistem Testi (1.0 saat)
040 - Üretim Hattı Yeniden Başlatma (0.5 saat)

Gerekli Yedek Parçalar:
- Tarayıcı Kafa Montajı (Mat: 123456789)
- Optik Kablo Seti (Mat: 987654321)
- Kalibrasyon Yazılımı Lisansı (Hizmet)
```

---

## Bölüm 6: Bakım Geçmişi ve Analitik

### Her Bakım Olayından Öğrenmek

Merve bakım verilerini eyleme geçirilebilir içgörülere dönüştürdü. Her iş emri, her arıza, her başarılı önleme organizasyonel öğrenmenin parçası oluyor.

```
BAKIM ANALİTİĞİ ÇERÇEVESİ
═════════════════════════════════════

Ekipman Performansı
├── Kullanılabilirlik Metrikleri
├── Güvenilirlik Göstergeleri
└── Bakılabilirlik Ölçüleri

Maliyet Analizi
├── Ekipman Başına Bakım Maliyetleri
├── İşçilik vs. Malzeme Maliyetleri
└── Planlı vs. Plansız Oranları

Trend Analizi
├── Arıza Modeli Tanıma
├── Mevsimsel Bakım İhtiyaçları
└── Yaşlanan Ekipman Stratejileri

Öngörülü İçgörüler
├── Arızalar Arası Ortalama Süre
├── Optimal Bakım Aralıkları
└── Değiştirme Önerileri
```

**Analitik ve Raporlama T-Kodları:**
- **IW37N**: Analitik ile Emir Yöneticisi
- **MC.1**: Ekipman için ABC Analizi
- **MC.2**: Ekipman Kullanılabilirlik Raporu
- **MC.3**: Ekipman Maliyet Raporu
- **MC.5**: Bakım Maliyet Analizi
- **MCEC**: Ekipman Maliyet Merkezleri

Merve aylık bakım performans analizini çalıştırıyor:

**İşlem: MC.2 (Ekipman Kullanılabilirlik Raporu)**
```
Analiz Dönemi: Bu Ay
Ekipman Seçimi: Tüm Karşılama Hattı Ekipmanları

Ana Metrikler:
┌─────────────────────────────────────────────────────────┐
│ EKİPMAN KULLANILABİLİRLİK PANOSU                       │
├─────────────────────────────────────────────────────────┤
│ Ekipman ID     │ Kullanılabilirlik │ MTBF   │ MTTR   │ Maliyet │
├─────────────────────────────────────────────────────────┤
│ EQ-CONV-001    │    %98.5         │ 720saat│ 2.5saat│ ₺35.000│
│ EQ-SCAN-001    │    %95.2         │ 480saat│ 4.0saat│ ₺58.800│
│ EQ-SORT-001    │    %99.1         │ 960saat│ 1.5saat│ ₺23.800│
│ EQ-PACK-001    │    %97.8         │ 600saat│ 3.0saat│ ₺40.600│
├─────────────────────────────────────────────────────────┤
│ Hat Ortalaması │    %97.7         │ 690saat│ 2.8saat│ ₺39.550│
└─────────────────────────────────────────────────────────┘

Trend: Geçen aya göre +%2.3 iyileşme
Hedef: Yıl sonuna kadar %98.5 kullanılabilirlik
```

---

## Bölüm 7: Yedek Parça ve Malzeme Yönetimi

### Stratejik Envanter Dengesi

Merve doğru zamanda doğru parçalara sahip olmanın etkili bakım gerektirdiğini öğrendi. Çok fazla envanter sermayeyi bağlar; çok az ise uzun duruş süresine neden olur.

```
YEDEK PARÇA STRATEJİSİ
══════════════════════════════════════

Sınıflandırma Sistemi
├── Kritik Parçalar (Yüksek Etki, Düşük Frekans)
├── Temel Parçalar (Orta Etki, Orta Frekans)
└── Standart Parçalar (Düşük Etki, Yüksek Frekans)

Envanter Yönetimi
├── Min/Max Stok Seviyeleri
├── Teslimat Süresi Değerlendirmeleri
└── Tedarikçi Güvenilirlik Faktörleri

Entegrasyon Temas Noktaları
├── MM Modülü (Satın Alma)
├── WM Modülü (Depo)
└── FI Modülü (Maliyet Kontrolü)
```

**Yedek Parça Yönetimi T-Kodları:**
- **MM01**: Malzeme Ana Kartı Oluştur (Yedek Parçalar)
- **MM02**: Malzeme Ana Kartı Değiştir
- **MB51**: Malzeme Belgesi Listesi
- **MD04**: Stok/Gereksinim Listesi
- **ME51N**: Satın Alma Talebi Oluştur
- **IW34**: Rezervasyonları Toplu Serbest Bırak

Merve kritik ekipman için yedek parça durumunu gözden geçiriyor:

**İşlem: MD04 (Stok/Gereksinim Listesi)**
```
Malzeme: MOTOR-BEARING-6208 (Kritik Yedek)
Tesis: 1000

Stok Durumu:
Mevcut Stok: 5 AD
Rezerve: 2 AD (planlı bakım için)
Kullanılabilir: 3 AD
Güvenlik Stoku: 4 AD
Yeniden Sipariş Noktası: 6 AD

Gereksinimler:
42. Hafta: 2 AD (Zamanlanmış PM)
44. Hafta: 1 AD (Zamanlanmış PM)
46. Hafta: 3 AD (Ana revizyon)

Gerekli Eylem: 8 AD için Satın Alma Talebi
Teslimat Süresi: 14 Gün
Tedarikçi: SKF Endüstriyel
```

---

## Bölüm 8: Entegrasyon Mükemmelliği

### Merve'nin Bağlantılı Bakım Ekosistemi

Merve SAP PM'in diğer modüllerle nasıl sorunsuz entegre olduğunu ve birleşik bir iş süreci oluşturduğunu gördü.

```
PM ENTEGRASYON ORTAMI
══════════════════════════════════════

Finansal Entegrasyon (FI/CO)
├── Bakım Maliyet Yakalama
├── Bütçe vs. Gerçek Analizi
└── Varlık Amortisman Etkisi

Malzeme Yönetimi (MM)
├── Yedek Parça Satın Alma
├── Envanter Optimizasyonu
└── Satıcı Yönetimi

Üretim Planlama (PP)
├── Kapasite Planlama
├── Duruş Zamanlaması
└── Üretim Etki Analizi

Kalite Yönetimi (QM)
├── Kontrol Gereksinimleri
├── Kalite Bildirimleri
└── Sürekli İyileştirme

İnsan Kaynakları (HR)
├── Teknisyen Yeterlilikleri
├── Eğitim Gereksinimleri
└── Sertifika Yönetimi
```

**Entegrasyon Örneği: Tam Bakım-Satın Alma Akışı**

Merve konveyör rulmanlarında tekrarlayan arıza modeli keşfeder ve tam bir iş süreci başlatır:

1. **PM Analizi** (MC.5): Yüksek rulman değişim sıklığını belirler
2. **Kalite Araştırması** (QM02): Rulman tedarikçisi için kalite bildirimi oluşturur
3. **Satın Alma Eylemi** (ME51N): Alternatif rulman tedarikçileri için fiyat teklifi ister
4. **Maliyet Analizi** (KSB1): Tedarikçiler arasında bakım maliyetlerini karşılaştırır
5. **Satıcı Değerlendirmesi** (ME61): Tedarikçi performansını değerlendirir
6. **Stratejik Karar**: Daha yüksek kaliteli rulman tedarikçisine geçer

```
ENTEGRE İŞ ETKİSİ
═══════════════════════════════

Finansal Sonuçlar:
├── Rulman Maliyeti: Birim başına +%15
├── Bakım Saatleri: Yıllık -%40
├── Duruş Maliyetleri: Yıllık -%60
└── Net Tasarruf: Yıllık ₺1.260.000

Operasyonel İyileştirmeler:
├── Ekipman Kullanılabilirliği: +%2.5
├── Bakım Verimliliği: +%35
└── Teknisyen Memnuniyeti: +%25
```

---

## Bölüm 9: Gelişmiş Bakım Stratejileri

### Öngörülü Bakım ve Endüstri 4.0

Merve geleneksel bakımı aşarak öngörülü analitik ve IoT entegrasyonunu benimsedi.

```
ÖNGÖRÜLÜ BAKIM EKOSİSTEMİ
════════════════════════════════════

Veri Toplama Katmanı
├── Titreşim Sensörleri
├── Sıcaklık Monitörleri
├── Basınç Ölçerleri
└── Performans Sayaçları

Analitik Motoru
├── Trend Analizi
├── Model Tanıma
├── Anomali Tespiti
└── Arıza Tahmini

Eylem Tetikleyicileri
├── Otomatik Bildirimler
├── İş Emri Oluşturma
├── Yedek Parça Siparişi
└── Zamanlama Optimizasyonu
```

**Gelişmiş T-Kodları ve Araçlar:**
- **IW37N**: Emir Yöneticisi (KPI panoları ile)
- **MC.1**: ABC Analizi
- **MCCS**: Bakım Maliyet Kontrolü
- **Tesis Bakımı Fiori Uygulamaları**: Bakım için modern arayüz

Merve öngörülü bakım senaryosu uygular:

**Öngörülü Bakım Uyarı Örneği:**
```
Ekipman: EQ-CONV-001 (Ana Konveyör Motor)
Sensör Veri Analizi:

Titreşim Trendi: ↗ 30 gün içinde %15 artış
Sıcaklık: ↗ Temel seviyeden +5°C
Akım Çekimi: ↗ Normalden +%8
Çalışma Saatleri: 8.450 saat (Rulman ömrü: 10.000 saat)

Öngörülü Model Çıktısı:
Arıza Olasılığı: 14 gün içinde %35
Önerilen Eylem: Rulman değişimi planla
Optimal Pencere: Sonraki planlı duruş (Hafta sonu)
Maliyet Kaçınma: ₺700.000 (acil arıza maliyeti)
```

**İşlem: IW01 (Öngörülü Bakım Emri)**
```
Emir Türü: PM02 (Durum Tabanlı Bakım)  
Ekipman: EQ-CONV-001
Öncelik: 3 (Orta - Planlı)
Açıklama: Öngörülü Rulman Değişimi - Arızayı Önle

Gerekçe: Sensör verileri %35 arıza olasılığı gösteriyor
Zamanlama: Sonraki müsait bakım penceresi
Bileşenler: Motor rulman seti, yağlayıcılar, conta
Tahmini Süre: 4 saat
Maliyet Kaçınma: ₺700.000 acil arıza maliyeti
```

---

## Bölüm 10: Mükemmelliğin İş Etkisi

### Merve'nin Dönüşüm Yolculuğu - Rakamlar Hikayeyi Anlatıyor

SAP PM yolculuğunun ikinci yılında, Merve dönüşüm sonuçlarını liderliğe sunuyor:

```
BAKIM DÖNÜŞÜMÜ METRİKLERİ
════════════════════════════════════════

Ekipman Güvenilirliği
┌─────────────────────────────────────────┐
│ SAP PM Öncesi  │ SAP PM Sonrası │ Değişim │
├─────────────────────────────────────────┤
│ Kullanılabilirlik │               │        │
│    %85.2       │    %97.8      │ +%12.6 │
├─────────────────────────────────────────┤
│ MTBF           │               │        │
│    240 saat    │    720 saat   │ +%200  │
├─────────────────────────────────────────┤
│ MTTR           │               │        │
│    8.5 saat    │    2.8 saat   │ -%67   │
└─────────────────────────────────────────┘

Maliyet Performansı
┌─────────────────────────────────────────┐
│ Acil Onarımlar         │ -%75            │
│ Fazla Mesai Maliyetleri│ -%60            │
│ Yedek Parça Envanteri  │ -%25            │
│ Toplam Bakım Maliyeti  │ -%35            │
└─────────────────────────────────────────┘

İş Etkisi
┌─────────────────────────────────────────┐
│ Üretim Çalışma Süresi  │ +%12.6          │
│ Sipariş Karşılama Oranı│ +%8.3           │
│ Müşteri Memnuniyeti    │ +%15            │
│ Yıllık Maliyet Tasarrufu│ ₺13.580.000    │
└─────────────────────────────────────────┘
```

---

## Merve'nin SAP PM Ustalığı - Temel T-Kodları Referansı

### Ekipman Yönetimi
- **IE01/02/03**: Ekipman Ana Kartı oluştur/değiştir/görüntüle
- **IH01/02/03**: Fonksiyonel Konum oluştur/değiştir/görüntüle
- **IB01/02/03**: Montaj oluştur/değiştir/görüntüle

### Bakım Planlama
- **IA01/02/03**: Görev Listesi oluştur/değiştir/görüntüle
- **IP01/02/03**: Bakım Planı oluştur/değiştir/görüntüle
- **IP15/17/19**: Bakım Planı ve Kalemi görüntüle
- **IP24**: Bakım Planı Zamanlaması

### İş Emri Yönetimi
- **IW01/02/03**: Bakım Emri oluştur/değiştir/görüntüle
- **IW21/22/23**: Arıza Emri oluştur/değiştir/görüntüle
- **IW28**: Bakım Emirlerini Liste Düzenle
- **IW37N**: Emir Yöneticisi (Yeni)
- **IW38/39**: Emir Gerçek Verileri ve Tamamlama

### Bildirim Yönetimi
- **IW51/52/53**: Bakım Bildirimi oluştur/değiştir/görüntüle
- **IW56**: Bildirim Listesi
- **QM01/02/03**: Kalite Bildirimi oluştur/değiştir/görüntüle

### Analitik ve Raporlama
- **MC.1**: Ekipman için ABC Analizi
- **MC.2**: Ekipman Kullanılabilirlik Raporu
- **MC.3**: Ekipman Maliyet Raporu
- **MC.5**: Bakım Maliyet Analizi
- **MCEC**: Ekipman Maliyet Merkezleri

### Malzemeler ve Yedek Parçalar
- **MM01/02/03**: Malzeme Ana Kartı oluştur/değiştir/görüntüle
- **MD04**: Stok/Gereksinim Listesi
- **MB51**: Malzeme Belgesi Listesi
- **IW34**: Rezervasyonları Toplu Serbest Bırak

---

## Trendyol Sipariş Hikayesi - PM'in Kritik Rolü

Merve, Trendyol sipariş karşılama merkezinin Kara Cuma hacimlerini işlemesini izlerken, PM'in bu başarı hikayesindeki hayati katkısını düşünüyor:

**Yoğun Talep Sırasında Ekipman Güvenilirliği:**
```
KARA CUMA HAZIRLIK KONTROLÜ
═══════════════════════════════════════

Etkinlik Öncesi Bakım (46. Hafta)
├── Tüm Kritik Ekipman: %100 Sağlık Kontrolü
├── Yedek Parçalar: Acil stok %200 artırıldı
├── Yedek Ekipman: Aktive edildi ve test edildi
└── Bakım Ekibi: 7/24 kapsama zamanlandı

Etkinlik Sırasında İzleme
├── Gerçek Zamanlı Ekipman Durum Panosu
├── Öngörülü Analitik: Aktif izleme
├── Acil Müdahale: <15 dakika hedef
└── Yedek Sistemler: Anında aktive etmeye hazır

Elde Edilen Sonuçlar:
├── %99.8 Ekipman Kullanılabilirliği
├── Sıfır Kritik Arıza
├── 2.8M Sipariş Başarıyla İşlendi
└── Ekipman Sorunlarından ₺0 Gelir Kaybı
```

*"Tüm şirket o sipariş rakamlarının tırmanışını izlerken,"* diyor Merve, *"PM her konveyörün, her tarayıcının, her ekipman parçasının kusursuz performans göstermesini sağlayan sessiz gardiyandır. Sadece makineleri bakımını yapmıyoruz - iş başarısını mümkün kılıyoruz."*

---

## Merve'nin Bilgeliği - Tesis Bakımı Felsefesi

*"Tesis Bakımı endüstriyel operasyonların kalp atışıdır. Her makine titreşimleri, sıcaklıkları ve performans verileri aracılığıyla bir hikaye anlatır. SAP PM bu hikayeleri dinlememize, ekipmanlarımızın neye ihtiyacı olduğunu anlamamıza ve fısıltılar çığlığa dönüşmeden önce yanıt vermemize yardımcı olur."*

*"Reaktif itfaiyecilerden proaktif ekipman güvenilirliği orkestra şeflerine evrimleştik. Her iş emri, her bakım planı, her yedek parça kararı operasyonel mükemmellik senfonisine katkıda bulunur."*

*"Bakım başarısının gerçek ölçüsü sadece ekipman çalışma süresi değildir - işletmenin ekipman arızaları konusunda endişelenmeden hedeflerine ulaşmasını sağlamaktır. Üretim üretime, kalite kaliteye ve satış müşterilere odaklandığında, işimizi doğru yapmış oluruz."*

---

**Bakım Mühendisinin İnancı:**
- *Ekipmanlarınızı dinleyin - neye ihtiyaçları olduğunu size söyleyecekler*
- *Onarım yapmak yerine önleyin - öngörü reaksiyonu yener*
- *Her eylemi planlayın - rastgele bakım rastgele sonuçlar yaratır*
- *Her şeyi ölçün - ölçülen şey iyileştirilir*
- *İşletme ile entegre olun - bakım operasyonel hedeflere hizmet eder*
- *Öğrenmeyi asla bırakmayın - her arıza değerli dersler öğretir*

---

**[Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali/) - SAP Hikaye Anlatıcısı ve Dijital Dönüşüm Savunucusu tarafından oluşturuldu**

*"Tesis Bakımında sadece şeyleri tamir etmiyoruz - onların hiç bozulmamasını sağlıyoruz."*

---

## Navigasyon
← [Önceki: Kalite Yönetimi](../09-quality-management/README.md) | [Ana Yolculuğa Dön](../README.md) | [Sonraki: Entegrasyon Senaryoları](../11-integration-scenarios/README.md) →

## İlgili Hikayeler
- 🌟 [Tam Entegrasyon Örneği](../01-overview/Complete-SAP-Integration-Example.md)
- 🎼 [SAP Orkestrası](../01-overview/README-Orchestra.md)
- 🏭 [Peter ile Üretim Planlama](../04-production-planning/README.md)
- 📊 [Marcus ile Malzeme Yönetimi](../03-materials-management/README.md)
- 💰 [Franz ile Mali Muhasebe](../05-financial-accounting/README.md)