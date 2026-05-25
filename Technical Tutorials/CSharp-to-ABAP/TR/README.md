# 🧭 C# & Python'dan SAP ABAP'a — Kariyer Değiştirenin Saha Rehberi

> *"Bir şeyi basitçe açıklayamıyorsan, onu yeterince anlamamışsın demektir."* — Richard Feynman

> 🇬🇧 **English version:** [../README.md](../README.md)

## Hoş Geldiniz

Zaten iyi kod yazıyorsunuz. C# biliyorsunuz, belki Python da. Sınıfları, generics'i, LINQ'yu, async'i, REST'i, ORM'leri, dependency injection'ı anlıyorsunuz. Şimdi ise **SAP ABAP geliştiricisi olmak — ve işe hızla girmek** istiyorsunuz.

Bu kitap tam o köprü. Burada size "programlamayı" öğretmiyoruz (onu zaten biliyorsunuz). Size **bildiğiniz her şeyin SAP dünyasına nasıl eşlendiğini** öğretiyoruz: adların farklı olduğu, IDE'nin daha eski hissettirdiği, veritabanının kutsal sayıldığı ve 40 yıllık bir dilin küresel ekonominin büyük bir dilimini sessiz sedasız çalıştırdığı o dünyaya.

Kitap boyunca **Feynman tekniğini** kullanıyoruz: sade dil, gerçek benzetmeler, pazarlama jargonu yok — sanki kahve (ya da çay ☕) eşliğinde pair programming yapıyormuşuz gibi.

## Bu Kitap Kimin İçin?

- SAP ekibine geçen **C# / .NET geliştiricileri**
- ABAP rolü alan (ya da almak isteyen) **Python geliştiricileri**
- **İlk SAP işine** hedef koyan bootcamp mezunları ve kendi kendine öğrenenler
- Kod yazabilen ama `SE80`, `DDIC`, `BAPI`, `SPROXY`, `CDS`, `RAP` gibi kavramlarla karşılaşınca *"bunlar da ne?"* diye düşünen herkes

> **Ön koşul:** Döngü yazabilmeli, sınıf tasarlayabilmeli ve bir REST API çağrısı yapabilmelisiniz. Bunları henüz bilmiyorsanız önce öğrenin — bu kitap enerjisini *SAP'a özgü* boşluğa harcar, programlamayı sıfırdan öğretmeye değil.

## Bu Kitap Diğerlerinden Nasıl Farklı?

Her kavram **üç kez** öğretilir:

1. **Benzetme** — kavramın sade dilde *ne olduğu*.
2. **Bunu zaten biliyorsun** — karşılaştırmalı C#/Python eşdeğeri, kod olarak.
3. **ABAP'taki karşılığı** — gerçek ABAP kodu, onu nerede bulacağınız ve yeni gelenleri tuzağa düşüren gotcha'lar.

```text
   Bildiğiniz (C# / Python)               SAP'ın adlandırması
   ──────────────────────────────       ─────────────────────────────
   Class library / NuGet package    →   Package + Transport (SE80 / ADT)
   Entity Framework model / table   →   Data Dictionary (DDIC, SE11)
   SQL Server / PostgreSQL          →   SAP HANA (ve Open SQL yazarsınız)
   ASP.NET Web API controller       →   OData service (SEGW / RAP)
   DTO / record                     →   Structure + Work Area
   List<T> / pandas DataFrame       →   Internal Table
   try/catch                        →   TRY ... CATCH cx_root
   interface + DI                   →   Interface + factory / BAdI
   Razor / Blazor page              →   Fiori / UI5 app
```

## Öğrenme Yolu (ve Bu Sıranın Nedeni)

| Bölüm | Ne Öğreneceksiniz | Kariyer İçin Önemi |
|-------|------------------|-------------------|
| **I — Oryantasyon** | SAP/ERP'nin gerçekte ne olduğu, modüller, proje türleri | Stand-up'larda ekibinizin konuştuklarını anlayabilmek için |
| **II — Temeller** | ADT (Eclipse'te ABAP), DDIC, eski ve yeni ABAP söz dizimi | Her gün kullandığınız ekmek-tuz bilgileri |
| **III — Klasik Çıktı** | Raporlar, ALV, Module Pool, Smartforms/Adobe Forms | Gerçek dünya biletlerinin yarısı "şu raporu / formu düzelt" şeklindedir |
| **IV — OOP & Entegrasyon** | ABAP Objects, Function Module'lar, RFC, BAPI, geliştirmeler, BDC/LSMW | Entegrasyon ve temiz kodun kalbi |
| **V — Modern Veri** | CDS View'ları, AMDP, OData giriş, Consumer Proxy, IDoc'lar | Recruiter'ların aradığı S/4HANA becerileri |
| **VI — OData Derinlemesine** | Entity type'lar, CRUD, association'lar, derin/genişletilmiş entity'ler, dosya yükleme/indirme | Juniorları işe alan *asıl* entegrasyon becerisi |
| **VII — Entegrasyonlar** | ABAP'tan Google Forms ve WhatsApp | CV'nizi öne çıkaran portföy projeleri |
| **VIII — Modern UI & RAP** | Fiori, ardından RESTful Application Programming modeli | SAP'ın gidişatı — geleceğe yönelik final |

## Hızlı Gezinti

📖 **[İçindekiler](content.md)** — tam bölüm haritası

### Kısımlar

**Bölüm I — Oryantasyon**
- [Önsöz: Bu kitap nasıl okunur](chapters/00-preface.md)
- [01 — Bir Geliştirici İçin SAP & ERP](chapters/01-sap-erp-introduction.md)
- [02 — Jargon Olmadan SAP Modülleri (MM, SD, FI, …)](chapters/02-sap-modules.md)
- [03 — SAP Proje Türleri (ve nereye uyacağınız)](chapters/03-sap-project-types.md)

**Bölüm II — ABAP Temelleri**
- [04 — Eclipse'te ABAP (ADT): Yeni Visual Studio'nuz](chapters/04-abap-on-eclipse-adt.md)
- [05 — Data Dictionary (DDIC): SAP'ın şema katmanı](chapters/05-data-dictionary-ddic.md)
- [06 — C#/Python geliştiricileri için ABAP söz dizimi temelleri](chapters/06-abap-syntax-basics.md)
- [07 — Modern ABAP Söz Dizimi (7.4+): güzel kısımlar](chapters/07-modern-abap-syntax.md)

**Bölüm III — Klasik Çıktı**
- [08 — Raporlar: Klasik listeler ve ALV grid'ler](chapters/08-reports-classical-and-alv.md)
- [09 — Module Pool (diyalog) programlama](chapters/09-module-pool-programming.md)
- [10 — Smartforms & Adobe Forms](chapters/10-smartforms-and-adobe-forms.md)

**Bölüm IV — OOP & Entegrasyon**
- [11 — ABAP Objects: OOP vs C#/Python](chapters/11-abap-oop.md)
- [12 — Function Module, RFC & BAPI](chapters/12-function-modules-rfc-bapi.md)
- [13 — Uygulamalı: BAPI_ACC_DOCUMENT_POST ile kayıt (SE37)](chapters/13-handson-bapi-acc-document-post.md)
- [14 — Standart SAP'ı Geliştirmek (güvenli yol)](chapters/14-abap-enhancements.md)
- [15 — Veri Göçü: BDC & LSMW](chapters/15-data-migration-lsmw-bdc.md)

**Bölüm V — Modern Veri & Servisler**
- [16 — CDS View'ları (Core Data Services)](chapters/16-cds-views.md)
- [17 — AMDP (ABAP Managed Database Procedures)](chapters/17-amdp.md)
- [18 — OData & REST: Büyük Resim](chapters/18-odata-and-rest-intro.md)
- [19 — RFC web servisleri: Consumer Proxy (SPROXY)](chapters/19-consumer-proxy-sproxy.md)
- [20 — ALE & IDoc'lar: SAP'ın klasik mesajlaşması](chapters/20-ale-idocs.md)

**Bölüm VI — OData Derinlemesine**
- [21 — Entity Type, Entity Set, XML & JSON](chapters/21-odata-entity-types-xml-json.md)
- [22 — Servis metodları ve import parametreleri](chapters/22-odata-methods-and-parameters.md)
- [23 — Veri okuma: GET_ENTITY & GET_ENTITYSET](chapters/23-odata-read-crud-get-entity.md)
- [24 — Arama dizileri ve sorgu seçenekleri](chapters/24-odata-search-and-query-options.md)
- [25 — Oluşturma, Güncelleme & Silme](chapters/25-odata-create-update-delete.md)
- [26 — Association'lar: asıl ve bağımlı entity set'ler](chapters/26-odata-associations.md)
- [27 — Tek serviste başlık + kalem verisi](chapters/27-odata-header-and-item.md)
- [28 — Function Import'lar](chapters/28-odata-function-import.md)
- [29 — CREATE_DEEP_ENTITY](chapters/29-odata-create-deep-entity.md)
- [30 — GET_EXPANDED_ENTITYSET](chapters/30-odata-get-expanded-entityset.md)
- [31 — OData'da dosya yükleme ve indirme](chapters/31-odata-file-upload-download.md)

**Bölüm VII — Gerçek Dünya Entegrasyonları**
- [32 — Google Form → SAP Entegrasyonu](chapters/32-google-form-integration.md)
- [33 — ABAP'tan WhatsApp Entegrasyonu](chapters/33-whatsapp-integration.md)

**Bölüm VIII — Modern UI & RAP**
- [34 — ABAP Geliştiricileri İçin Fiori & UI5](chapters/34-fiori.md)
- [35 — RAP: RESTful Application Programming (Final)](chapters/35-rap-restful-application-programming.md)

### Ekler
- [Ek A — C# / Python ↔ ABAP hızlı başvuru](chapters/appendix-a-csharp-abap-cheatsheet.md)
- [Ek B — Yeni gelenler için sözlük](chapters/appendix-b-glossary.md)
- [Ek C — Transaction kodu referansı](chapters/appendix-c-transaction-codes.md)
- [Ek D — Kaynaklar, pratik sistemler & iş hazırlığı](chapters/appendix-d-resources.md)

## Felsefe

1. **Bildiklerinize saygı duyun.** Programlamada başlangıç değilsiniz — *SAP'ta* başlangıçsınız. Biz deltayı öğretiyoruz.
2. **Her zaman iki dünyayı gösterin.** Her kavram önce C# (ve çoğu zaman Python) paraleli ile verilir, ardından ABAP gelir.
3. **İşe yararlığı ön planda tutun.** Gerçek transaction kodları, gerçek tablo adları, gerçek mülakat soruları, gerçek portföy projeleri.
4. **Dürüst olun.** ABAP'ın hantal veya eski kaldığı yerlerde açıkça söyleriz — ve modern alternatifi gösteririz.
5. **Teoriden çok uygulamaya.** Her bölüm, gerçekten yapabileceğiniz bir şeyle biter.

---

*☕ ve Feynman tekniğiyle inşa edildi — SAP dünyasına geçiş yapan geliştiriciler için.*
