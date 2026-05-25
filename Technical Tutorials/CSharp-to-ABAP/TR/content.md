# 📖 İçindekiler

## C# & Python'dan SAP ABAP'a — Kariyer Değiştirenin Saha Rehberi

---

### [Önsöz: Bu Kitap Nasıl Okunur](chapters/00-preface.md)
- Bu kitap kimin için (zaten kod yazıyorsunuz — biz SAP deltasını öğretiyoruz)
- Üç geçiş yöntemi: benzetme → "bunu zaten biliyorsun" → ABAP'taki karşılığı
- Ücretsiz pratik sistem nasıl kurulur
- İşe hazırlık zihniyeti

---

## Bölüm I — Oryantasyon

### [Kısım 1: Bir Geliştirici İçin SAP & ERP](chapters/01-sap-erp-introduction.md)
- 1.1 ERP aslında nedir ("tek ortak veritabanı" fikri)
- 1.2 SAP ECC vs S/4HANA vs SAP BTP — genel tablo
- 1.3 Client (mandant) kavramı: multi-tenant gibi ama sisteme gömülü
- 1.4 ABAP'ın çalıştığı yer: NetWeaver Application Server
- 1.5 Bir ABAP ekibinde günün nasıl geçtiği

### [Kısım 2: Jargon Olmadan SAP Modülleri](chapters/02-sap-modules.md)
- 2.1 MM, SD, FI, CO, PP, HCM, PM, QM — her biri bir satırda
- 2.2 Modüller veriyi nasıl paylaşır (belge akışı fikri)
- 2.3 Hangi tablolar hangi modüle aittir (MARA, VBAK, BKPF…)
- 2.4 Geliştirici olarak "modül dilini" neden konuşmalısınız

### [Kısım 3: SAP Proje Türleri (ve Nereye Uyacağınız)](chapters/03-sap-project-types.md)
- 3.1 Uygulama vs Destek vs Rollout vs Göç
- 3.2 On-premise vs RISE/bulut — sizin için ne değişir
- 3.3 Roller: geliştirici, fonksiyonel, basis, mimar
- 3.4 Junior geliştiricinin ilk 90 günü

---

## Bölüm II — ABAP Temelleri

### [Kısım 4: Eclipse'te ABAP (ADT) — Yeni Visual Studio'nuz](chapters/04-abap-on-eclipse-adt.md)
- 4.1 ADT vs SE80: modern ve klasik IDE
- 4.2 Sisteme bağlanma, projeler, favoriler
- 4.3 Package & Transport = solution/proje + kaynak kontrolü (bir bakıma)
- 4.4 Aktivasyon, söz dizimi kontrolü, ABAP hata ayıklayıcı
- 4.5 Visual Studio geliştiricisinin özleyeceği klavye kısayolları

### [Kısım 5: Data Dictionary (DDIC) — SAP'ın Şema Katmanı](chapters/05-data-dictionary-ddic.md)
- 5.1 DDIC neden vardır (tüm şirketin verisini tiplendirmek)
- 5.2 Domain, Data Element, Structure, Table (C# tipleri ve EF'ye karşı)
- 5.3 Transparent table'lar, anahtarlar, indeksler
- 5.4 Table type'lar, arama yardımları, yabancı anahtarlar
- 5.5 SE11 uygulaması: ilk tablonuzu oluşturun

### [Kısım 6: C#/Python Geliştiricileri İçin ABAP Söz Dizimi Temelleri](chapters/06-abap-syntax-basics.md)
- 6.1 Değişkenler, tipler, `DATA`, `TYPES`
- 6.2 Kontrol akışı: IF/CASE/LOOP/WHILE/DO
- 6.3 Structure'lar ve work area'lar vs DTO'lar/record'lar
- 6.4 Internal table = sizin List<T>'niz / DataFrame'iniz
- 6.5 Open SQL: tablolara SELECT (vs LINQ/SQL)
- 6.6 Alt rutinler, çağrı yığını ve mesaj yönetimi

### [Kısım 7: Modern ABAP Söz Dizimi (7.4+) — Güzel Kısımlar](chapters/07-modern-abap-syntax.md)
- 7.1 Satır içi bildirimler (`DATA(...)`), `VALUE #( )`, `CORRESPONDING`
- 7.2 Tablo ifadeleri, `FILTER`, `REDUCE`, `FOR` (ABAP'ın LINQ'su)
- 7.3 String şablonları `|...|`
- 7.4 Yeni `SELECT` özellikleri, `@` host değişkenleri
- 7.5 Yan yana: eski ABAP'ı modern yolla yeniden yazmak

---

## Bölüm III — Klasik Çıktı

### [Kısım 8: Raporlar — Klasik Listeler ve ALV Grid'ler](chapters/08-reports-classical-and-alv.md)
- 8.1 Rapor olay modeli (INITIALIZATION, START-OF-SELECTION…)
- 8.2 Seçim ekranları = giriş formunuz
- 8.3 Klasik WRITE listeleri (ve hâlâ neden görürsünüz)
- 8.4 ALV: her SAP kullanıcısının beklediği grid (CL_SALV_TABLE)
- 8.5 Yapıştırıp çalıştırabileceğiniz tam bir ALV raporu

### [Kısım 9: Module Pool (Diyalog) Programlama](chapters/09-module-pool-programming.md)
- 9.1 Diyalog programı nedir (SAP GUI'daki WinForms)
- 9.2 Ekranlar, PBO/PAI, akış mantığı
- 9.3 Ekran öğeleri, tablo kontrolleri, GUI durumu
- 9.4 Hâlâ ne zaman kullanılır vs Fiori

### [Kısım 10: Smartforms & Adobe Forms](chapters/10-smartforms-and-adobe-forms.md)
- 10.1 SAP'ın yazdırma yöntemi neden böyle
- 10.2 SAPscript → Smartforms → Adobe Forms zaman çizelgesi
- 10.3 Sürücü program + form, veri geçirme
- 10.4 ABAP'tan PDF üretme

---

## Bölüm IV — OOP & Entegrasyon

### [Kısım 11: ABAP Objects — OOP vs C#/Python](chapters/11-abap-oop.md)
- 11.1 Sınıflar, metotlar, attribute'lar, görünürlük
- 11.2 Interface'ler, kalıtım, polimorfizm, abstract/final
- 11.3 Static vs instance, constructor'lar, `ME->`
- 11.4 Exception'lar: sınıf tabanlı `cx_root` vs try/catch
- 11.5 Event'ler, friend'ler ve factory deseni
- 11.6 Clean ABAP: sizi işe alan stil rehberi

### [Kısım 12: Function Module, RFC & BAPI](chapters/12-function-modules-rfc-bapi.md)
- 12.1 Function module'ler = çağrılabilir kütüphane fonksiyonlarınız
- 12.2 Import/Export/Changing/Tables parametreleri
- 12.3 RFC = remote procedure call (.NET/Python'dan çağrılabilir!)
- 12.4 BAPI'ler: SAP'ın resmi, kararlı iş API'leri
- 12.5 SAP'ı C#'tan (NCo) ve Python'dan (PyRFC) çağırmak

### [Kısım 13: Uygulamalı — BAPI_ACC_DOCUMENT_POST ile Kayıt (SE37)](chapters/13-handson-bapi-acc-document-post.md)
- 13.1 Finansmanda "belge kaydetmek" ne anlama gelir
- 13.2 SE37'de BAPI arayüzünü okumak
- 13.3 Başlık, GL ve döviz tablolarını doldurmak
- 13.4 BAPI_TRANSACTION_COMMIT ve hata yönetimi
- 13.5 Eksiksiz, çalıştırılabilir bir örnek

### [Kısım 14: Standart SAP'ı Geliştirmek (Güvenli Yol)](chapters/14-abap-enhancements.md)
- 14.1 SAP kodunu neden doğrudan değiştiremezsiniz
- 14.2 Geliştirme spektrumu: User Exit → BAdI → Enhancement Point
- 14.3 Doğru exit/BAdI'yi bulmak
- 14.4 Adım adım BAdI uygulamak
- 14.5 Clean Core: S/4HANA'da geliştirmeler

### [Kısım 15: Veri Göçü — BDC & LSMW](chapters/15-data-migration-lsmw-bdc.md)
- 15.1 Göç problemi (eski veriyi SAP'a yüklemek)
- 15.2 BDC: batch input & call transaction
- 15.3 SHDB ile kayıt alma
- 15.4 LSMW: az/sıfır kodlu göç tezgahı
- 15.5 Modern alternatif: migration cockpit

---

## Bölüm V — Modern Veri & Servisler

### [Kısım 16: CDS View'ları (Core Data Services)](chapters/16-cds-views.md)
- 16.1 CDS neden vardır (mantığı HANA'ya itmek)
- 16.2 İlk CDS view'ınız (SQL view / EF projeksiyonuyla karşılaştırma)
- 16.3 Association'lar, annotation'lar, parametreler
- 16.4 Analitik ve tüketim view'ları
- 16.5 OData & RAP'ın temeli olarak CDS

### [Kısım 17: AMDP (ABAP Managed Database Procedures)](chapters/17-amdp.md)
- 17.1 CDS yetmediğinde: SQLScript yazmak
- 17.2 AMDP sınıfı ve metot anatomisi
- 17.3 AMDP çağırma, hata ayıklama, performans
- 17.4 AMDP vs CDS vs Open SQL: araç seçimi

### [Kısım 18: OData & REST — Büyük Resim](chapters/18-odata-and-rest-intro.md)
- 18.1 REST özeti (bunu biliyorsunuz) ve OData'nın katkısı
- 18.2 OData URL kuralları ($filter, $expand, $top…)
- 18.3 SAP Gateway mimarisi
- 18.4 OData oluşturmanın iki yolu: SEGW (klasik) vs RAP (modern)
- 18.5 Metadata belgesi ve servis kataloğu

### [Kısım 19: RFC Web Servisleri — Consumer Proxy (SPROXY)](chapters/19-consumer-proxy-sproxy.md)
- 19.1 ABAP'tan dış SOAP/REST servisi tüketmek
- 19.2 SPROXY: WSDL'den proxy üretmek (`Add Service Reference` gibi)
- 19.3 Mantıksal portlar (SOAMANAGER)
- 19.4 Proxy çağırma ve hata yönetimi

### [Kısım 20: ALE & IDoc'lar — SAP'ın Klasik Mesajlaşması](chapters/20-ale-idocs.md)
- 20.1 IDoc = güçlü tiplenmiş mesaj/zarf
- 20.2 Temel tipler, segmentler, partner profilleri
- 20.3 Gelen ve giden akış
- 20.4 İzleme (WE02/WE19) ve yeniden işleme

---

## Bölüm VI — OData Derinlemesine

### [Kısım 21: Entity Type, Entity Set, XML & JSON](chapters/21-odata-entity-types-xml-json.md)
- 21.1 Entity Type vs Entity Set (sınıf vs koleksiyon)
- 21.2 Property'ler, anahtarlar ve SEGW'deki veri modeli
- 21.3 Metadata XML'i ($metadata) açıklaması
- 21.4 XML vs JSON yanıtları ($format)

### [Kısım 22: Servis Metodları & Import Parametreleri](chapters/22-odata-methods-and-parameters.md)
- 22.1 DPC/DPC_EXT sınıfı ve yeniden tanımlanabilir metotlar
- 22.2 URL işlemlerini metotlara eşlemek
- 22.3 io_tech_request_context: anahtarları, filtreleri, parametreleri okumak
- 22.4 Veri ve durum döndürmek

### [Kısım 23: Veri Okuma — GET_ENTITY & GET_ENTITYSET](chapters/23-odata-read-crud-get-entity.md)
- 23.1 GET_ENTITYSET (koleksiyon / liste)
- 23.2 GET_ENTITY (anahtarla tek kayıt)
- 23.3 İstekten anahtarları okumak
- 23.4 Uçtan uca salt okunur servis

### [Kısım 24: Arama Dizileri & Sorgu Seçenekleri](chapters/24-odata-search-and-query-options.md)
- 24.1 $filter → WHERE koşulu eşleşmesi
- 24.2 Serbest metin araması (search string)
- 24.3 $top, $skip, $orderby, $inlinecount (sayfalama)
- 24.4 GET_ENTITYSET'te uygulamak

### [Kısım 25: Oluşturma, Güncelleme & Silme](chapters/25-odata-create-update-delete.md)
- 25.1 CREATE_ENTITY (POST)
- 25.2 UPDATE_ENTITY (PUT/PATCH/MERGE)
- 25.3 DELETE_ENTITY (DELETE)
- 25.4 Doğrulama, mesajlar ve geri alma

### [Kısım 26: Association'lar — Asıl ve Bağımlı Entity Set'ler](chapters/26-odata-associations.md)
- 26.1 Association'lar ve navigation property'ler
- 26.2 SEGW'de tanımlamak
- 26.3 Navigation aracılığıyla bağımlı set için GET_ENTITYSET
- 26.4 Örnek: Müşteri → Siparişler

### [Kısım 27: Tek Serviste Başlık + Kalem Verisi](chapters/27-odata-header-and-item.md)
- 27.1 Klasik başlık/kalem modeli (örn. sipariş + satır kalemleri)
- 27.2 İki entity set + association
- 27.3 Başlığı kalemleriyle okumak
- 27.4 Gerçek örnek uygulaması

### [Kısım 28: Function Import'lar](chapters/28-odata-function-import.md)
- 28.1 CRUD yetmediğinde (eylemler/işlemler)
- 28.2 SEGW'de function import tanımlamak
- 28.3 EXECUTE_ACTION'ı yeniden tanımlamak
- 28.4 Örnek: "siparişi onayla" function import

### [Kısım 29: CREATE_DEEP_ENTITY](chapters/29-odata-create-deep-entity.md)
- 29.1 Tek istekte başlık + kalem kaydetmek
- 29.2 Derin yapı ve io_data_provider
- 29.3 Uygulama ve test
- 29.4 Altta yatan BAPI ile transactional kayıt

### [Kısım 30: GET_EXPANDED_ENTITYSET](chapters/30-odata-get-expanded-entityset.md)
- 30.1 $expand ve expand metodunu neden geçersiz kılarsınız
- 30.2 Genişletilmiş yanıt yapısını oluşturmak
- 30.3 Uygulama ve test
- 30.4 Performans notları

### [Kısım 31: OData'da Dosya Yükleme ve İndirme](chapters/31-odata-file-upload-download.md)
- 31.1 Media entity'ler ve stream'ler
- 31.2 GET_STREAM (indirme) & CREATE_STREAM (yükleme)
- 31.3 MIME türleri ve content disposition
- 31.4 Örnek: belge ekleme ve getirme

---

## Bölüm VII — Gerçek Dünya Entegrasyonları

### [Kısım 32: Google Form → SAP Entegrasyonu](chapters/32-google-form-integration.md)
- 32.1 Senaryo (form gönderimi SAP'a ulaşır)
- 32.2 Apps Script webhook → SAP OData/REST
- 32.3 Kimlik doğrulama ve gelen servis
- 32.4 Uçtan uca yapım (harika bir portföy parçası)

### [Kısım 33: ABAP'tan WhatsApp Entegrasyonu](chapters/33-whatsapp-integration.md)
- 33.1 Senaryo (WhatsApp mesajları gönderme/alma)
- 33.2 Cloud API temelleri ve webhook'lar
- 33.3 CL_HTTP_CLIENT / RFC ile mesaj göndermek
- 33.4 SAP'a mesaj almak

---

## Bölüm VIII — Modern UI & RAP

### [Kısım 34: ABAP Geliştiricileri İçin Fiori & UI5](chapters/34-fiori.md)
- 34.1 Fiori nedir (SAP'ın React/Angular dönemine ait UI'sı)
- 34.2 UI5 vs bildiğiniz framework'ler
- 34.3 Fiori Elements (metadata güdümlü UI'lar) vs serbest tasarım
- 34.4 Fiori uygulaması OData servisinizle nasıl konuşur
- 34.5 Launchpad ve dağıtım

### [Kısım 35: RAP — RESTful Application Programming (Final)](chapters/35-rap-restful-application-programming.md)
- 35.1 RAP nedir ve SEGW'nin yerini neden alır
- 35.2 Yığın: DB table → CDS → Behavior Definition → Servis
- 35.3 Managed vs unmanaged senaryolar
- 35.4 Adım adım tam bir RAP CRUD uygulaması
- 35.5 OData V4 + Fiori Elements uygulaması olarak yayımlamak
- 35.6 Bundan sonra nereye (kıdemli seviyeye yol haritanız)

---

## Ekler

### [Ek A: C# / Python ↔ ABAP Hızlı Başvuru](chapters/appendix-a-csharp-abap-cheatsheet.md)
### [Ek B: Yeni Gelenler İçin Sözlük](chapters/appendix-b-glossary.md)
### [Ek C: Transaction Kodu Referansı](chapters/appendix-c-transaction-codes.md)
### [Ek D: Kaynaklar, Pratik Sistemler & İş Hazırlığı](chapters/appendix-d-resources.md)

---

*[README'ye Geri Dön](README.md)*
