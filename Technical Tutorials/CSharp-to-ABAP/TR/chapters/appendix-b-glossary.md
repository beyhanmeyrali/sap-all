# Ek B: Yeni Başlayanlar İçin Sözlük

*Konuşma ortasında sizi durduran her SAP terimi — sade bir dille tanımlanmış, mümkün olan yerlerde C#/Python paraleli verilmiş.*

---

## 📌 Bu sözlük nasıl kullanılır?

Alfabetik sıra. Bir standup toplantısında ya da bilet üzerinde tanımadığınız bir terime rastlarsanız önce buraya bakın. Her madde şunları söyler: şeyin ne *olduğu*, C#/Python'da nasıl adlandırılacağı ve — kullanışlı olduğunda — açan transaction kodu.

---

## A

**ABAP**
SAP sistemlerinin içinde çalışan programlama dili. *Advanced Business Application Programming* ifadesinin kısaltmasıdır. SAP tarafından 1980'lerde oluşturulmuş olup tamamen SAP çalışma zamanı içinde yaşar (bağımsız bir derleyici yoktur). "SAP çekirdeğinin anladığı tek dil" olarak düşünülebilir — CLR'ınızın aynı zamanda veritabanı sunucunuz, web sunucunuz ve iş zamanlayıcınız da olduğu bir ortamda yalnızca .NET kodu yazabilmek gibi. Güncel sürüm (ABAP 7.5+) tam OOP, lambda'lar, şablon string'ler ve daha fazlasını destekler.

**ABAP Dictionary** — bkz. *DDIC*.

**ABAP Unit**
Yerleşik birim test çerçevesi. C#'ta `xUnit` veya Python'da `pytest` gibi, ancak ABAP Workbench ve ADT'ye entegre olarak gelir. Test sınıfları `FOR TESTING` ile etiketlenir. ADT'de Ctrl+Shift+F10 ile çalıştırılır.

**ADT (ABAP Development Tools)**
ABAP için Eclipse tabanlı IDE. SAP dünyasında Visual Studio / VS Code karşılığınızdır. ABAP kodunu SAPGUI'nin SE80'i yerine burada yazar ve hata ayıklarsınız. ADT, canlı bir SAP sistemine HTTP üzerinden bağlanır — yerel derleme yoktur. Bkz. Bölüm 4.

**ALE (Application Link Enabling)**
IDocs kullanarak SAP sistemleri arasında veri dağıtmaya yarayan SAP çerçevesi. Dahili bir pub/sub veya ESB gibi düşünülebilir; ancak daha eskidir ve XML içermez (IDoc'lar kendi düz dosya formatlarını kullanır). Yapılandırma `BD64` transaction'ında yer alır.

**ALV (ABAP List Viewer)**
SAP'taki standart ızgara/rapor görüntüleme bileşeni. WinForms'taki DataGrid veya WPF'teki `<DataGrid>` gibi — ancak klasik ABAP raporlarında kullandığınız *tek* ızgaradır ve yerleşik sıralama, filtreleme, dışa aktarma ve düzen kaydetme özellikleriyle birlikte gelir. Ana sınıf: `CL_SALV_TABLE` (OOP ALV) veya `REUSE_ALV_GRID_DISPLAY` function module'ü (klasik). Bkz. Bölüm 8.

**AMDP (ABAP Managed Database Procedures)**
ABAP sınıf metotlarını kullanarak SAP HANA saklı yordamları yazmanın bir yolu. Bir C# sınıfı içinde özel bir öznitelikle bildirilmiş bir SQL Server saklı yordamı yazmak gibi. Karmaşık hesaplamaları veritabanı katmanına taşımak için kullanışlıdır. Bkz. Bölüm 17.

---

## B

**BAdI (Business Add-In)**
Standart SAP kodunu değiştirmeden özel mantık eklemenizi sağlayan SAP geliştirme çerçevesi — SAP'ın tanımlanmış kanca noktalarında çağırdığı bir interface'i uygulamak gibi. En yakın C# analojisi bir eklenti desenidir: SAP interface'i tanımlar, siz uygulamayı sağlarsınız, çerçeve onu çağırır. `SE18`'de (tanım) ve `SE19`'da (uygulama) yönetilir. Bkz. Bölüm 14.

**BAPI (Business Application Programming Interface)**
Bir SAP iş nesnesi metodunu temsil eden kararlı, belgelenmiş RFC etkin function module. REST endpoint'i gibi düşünün ama fonksiyon çağrıları için: `BAPI_CUSTOMER_CREATEFROMDATA1`, `BAPI_ACC_DOCUMENT_POST` vb. BAPI'ler, dışarıdan SAP'a veri göndermek için güvenli ve desteklenen yoldur. Başarılı bir BAPI çağrısından sonra her zaman `BAPI_TRANSACTION_COMMIT` çağırın, aksi takdirde verileriniz kaydedilmez. Bkz. Bölüm 12.

**BDC (Batch Data Communication)**
SAP GUI ekran etkileşimlerini toplu olarak otomatikleştirme tekniği — veri girmek için SAPGUI ekranlarını yönlendiren kayıtlı bir makro veya Selenium betiği gibi. BAPI olmadığında veri taşıma amacıyla kullanılır. Bkz. Bölüm 15.

**BTP (SAP Business Technology Platform)**
SAP'ın bulut platformu. Azure veya AWS gibi, ancak SAP boyutlu: ABAP ortamlarını, entegrasyon ara yazılımlarını (Integration Suite), düşük kodlu araçları ve daha fazlasını barındırır. *SAP BTP ABAP Environment*, modern ABAP'ın (RAP, CDS, Tier 1 API'ler) çalıştığı bulut tabanlı bir ABAP sistemidir.

---

## C

**CDS (Core Data Services)**
SQL benzeri, ek açıklama destekli bir sözdizimi kullanarak veri modelleri ve görünümler tanımlamayı sağlayan, veritabanına "itilen" bir katman. Yerleşik OData, yetkilendirme ve UI ek açıklama desteğine sahip son derece güçlü bir SQL görünümü olarak düşünülebilir. C# açısından: LINQ ifadenizin veritabanı görünümü olarak derlendiğini ve aynı zamanda otomatik olarak bir REST endpoint oluşturduğunu hayal edin. S/4HANA geliştirmenin modern omurgası. Bkz. Bölüm 16.

**Client / Mandant**
Tek bir SAP sistemi içindeki mantıksal kiracı. Tek bir fiziksel SAP kurulumu birden fazla client barındırabilir (örn. client 100 = Geliştirme, 200 = QA, 300 = Üretim). Client'a bağımlı her tabloda `MANDT` anahtar alanı vardır. ABAP'ın Open SQL'i geçerli client'a göre otomatik filtreler — `WHERE mandt = sy-mandt` yazmazsınız, otomatik eklenir. Veritabanına bakıp `MANDT`'nin her yerde olduğunu gören her yeni gelene şaşırtıcı gelir.

**CMOD**
Geliştirme projelerini (SAP user exit / BAdI koleksiyonları) etkinleştiren ve yöneten transaction. Genellikle `SMOD` ile birlikte kullanılır. Bkz. *Enhancements*.

**Commit Work**
`COMMIT WORK` — ABAP'taki açık veritabanı commit ifadesi. EF Core'un `SaveChanges()` veya SQLAlchemy'nin `session.commit()` aksine, bunu çağırana kadar hiçbir şey kalıcı olarak veritabanına yazılmaz. Unutmak, ABAP veri taşıma kodundaki en yaygın hatalardan biridir. Ayrıca: geri almak için `ROLLBACK WORK`.

---

## D

**Data Element**
Anlam içeren adlandırılmış, yeniden kullanılabilir bir ABAP tipi: alan etiketi, arama yardımı, belgeleme ve domain. C#'taki adlandırılmış tip takma adı (`typedef`) artı meta veri gibi. Bir DDIC tablo sütununu `zfield TYPE ze_customer_id` olarak bildirdiğinizde bir Data Element'e referans veriyorsunuzdur. `SE11`'de yönetilir.

**DDIC (Data Dictionary)**
SAP'ın merkezi şema katmanı — tüm tablo tanımları, yapı tipleri, veri elemanları, domain'ler ve tip gruplarının bulunduğu yer. Entity Framework'ün model tanımlarının ve SQL Server'ın information schema'sının bir kombinasyonu gibi, ancak SAP sisteminin içinde saklanır. Transaction: `SE11`. Bkz. Bölüm 5.

**DPC / MPC (Data Provider Class / Model Provider Class)**
SEGW ile oluşturulan klasik bir OData servisini destekleyen iki üretilmiş ABAP sınıfı. MPC entity modelini tanımlar; DPC CRUD mantığını uygular. Gerçek kodunuzu yazmak için `*_EXT` varyantlarını genişletirsiniz. Bkz. Bölümler 21–31.

**Domain**
Bir Data Element için değer aralığını ve veri tipini tanımlayan DDIC nesnesi. Bir enum tanımı veya kısıtlı skaler tip gibi düşünün: `ZDOMAIN_STATUS` yalnızca `'A'`, `'I'`, `'D'` değerlerine izin verebilir. `SE11`'de yönetilir.

---

## E

**Enhancement**
Standart SAP kodunu değiştirmeden genişletme yöntemi (böylece SAP Support Package'ları değişikliklerinizin üzerine yazmaz). Türleri: User Exit'ler (eski), BAdI'ler (modern), Enhancement Spot'lar, Örtük/Açık Geliştirmeler. Bkz. Bölüm 14.

---

## F

**Fiori**
SAP'ın UI5 üzerine inşa edilmiş modern UI tasarım sistemi ve uygulama çerçevesi. SAP'ın Material Design + bileşen kütüphanesi karşılığı gibi. Fiori uygulamaları OData servislerini tüketen web uygulamaları (HTML5/JavaScript). Bkz. Bölüm 34.

**Function Module**
Biçimsel bir interface'e (Import, Export, Changing, Tables, Exceptions parametreleri) sahip yeniden kullanılabilir, çağrılabilir bir ABAP kod birimi. Sistem tarafından zorunlu kılınan biçimsel bir sözleşmeyle birlikte bir yardımcı sınıftaki statik metot gibi. Bazı Function Module'ler RFC etkindir ve SAP dışından çağrılabilir. `SE37`'de yönetilir. Bkz. Bölüm 12.

---

## G

**Gateway (SAP Gateway)**
OData servislerini barındıran ve sunan SAP bileşeni. OData endpoint'leriniz için API ağ geçidi / IIS barındırma katmanı gibi düşünülebilir. `/IWFND/MAINT_SERVICE` transaction'ı servisleri kaydeder; `/IWFND/GW_CLIENT` bunları Postman gibi test etmenizi sağlar. Bkz. Bölüm 18.

---

## H

**HANA (SAP HANA)**
SAP'ın bellek içi ilişkisel veritabanı — modern SAP sistemlerinin üzerinde çalıştığı veritabanı. Sütun tabanlı depolama, bellek içi işleme, toplamalar için son derece hızlı. AMDP yazmıyorsanız buna karşı **Open SQL** yazarsınız (yerel HANA SQL değil). Bir ABAP geliştiricisi olarak bellek içi olması büyük ölçüde görünmezdir.

---

## I

**IDoc (Intermediate Document)**
Sistemden sisteme veri alışverişi için SAP'ın özel mesaj formatı. Bir IDoc; kontrol kaydı, veri kayıtları ve durum kayıtlarından oluşan düz, yapılandırılmış bir belgedir — bir EDI mesajı veya SAP'ın kendi formatındaki güçlü tipli XML belgesi gibi. ALE entegrasyonunda ve harici EDI ortaklarıyla yoğun şekilde kullanılır. Bkz. Bölüm 20. Transaction'lar: `WE02`, `WE05`, `WE19`.

**Internal Table**
Aynı yapıyı (tipi) paylaşan satırlardan oluşan bellek içi koleksiyon. ABAP'taki en önemli veri yapısı. C#'ta `List<T>` veya pandas'ta `DataFrame` gibi, ancak üç çeşidi vardır: `STANDARD` (dizi benzeri), `SORTED` (otomatik sıralanmış, ikili arama), `HASHED` (hash haritası, O(1) anahtar arama). Bkz. Ek A ve Bölüm 6.

---

## L

**LSMW (Legacy System Migration Workbench)**
BDC oturumları kaydedebilen, BAPI çağırabilen veya düz dosyalardan veri yüklemek için IDoc kullanan nokta ve tıklama veri taşıma aracı. SAP için yerleşik ETL aracı gibi. Transaction: `LSMW`. Bkz. Bölüm 15.

**LUW (Logical Unit of Work)**
Veritabanı değişikliklerinin tamamının ya commit edildiği ya da geri alındığı sınır — veritabanı işlemiyle eşdeğer. ABAP dialog programlarında LUW, `COMMIT WORK` / `ROLLBACK WORK` ile kontrol edilir. Önemli: Uzak sistemlere yapılan RFC çağrılarının kendi LUW'ları vardır; bu nedenle BAPI commit'leri özel işlem gerektirir.

---

## M

**Module Pool**
Ekran tabanlı dialog (form) uygulamaları uygulayan bir ABAP programı türü — SAPGUI içinde çalışan bir WinForms veya WPF uygulaması gibi. Screen Painter'da tasarlanmış ekranlardan, akış mantığıyla (`PROCESS BEFORE OUTPUT` / `PROCESS AFTER INPUT`) bağlı olarak oluşturulur. Bkz. Bölüm 9.

**MANDT** — bkz. *Client / Mandant*.

---

## N

**NetWeaver (SAP NetWeaver)**
ABAP (ve Java) programları çalıştıran uygulama sunucusu platformu. Her şeyin altındaki çalışma zamanı katmanı: iş süreçlerini, belleği, veritabanı bağlantılarını ve HTTP için ICM'yi (Internet Communication Manager) yönetir. IIS + CLR + SQL Server bağlantı havuzu, tek bir üründe birleştirilmiş gibi düşünülebilir. Modern S/4HANA hâlâ NetWeaver üzerinde çalışır.

---

## O

**OData**
Veri API'leri oluşturmak için kullanılan açık protokol (HTTP + REST + Atom/JSON üzerine inşa edilmiş), SAP'ta yoğun şekilde kullanılır. Şemalı REST gibi: entity türleri, entity setleri, gezinti özellikleri ve standart sorgu seçenekleri (`$filter`, `$expand`, `$top` vb.) protokolün bir parçasıdır. SAP Gateway OData sunar. Bkz. Bölümler 18–31.

**Open SQL**
ABAP'ın SQL diyalekti — ABAP çalışma zamanının yerel veritabanı diyalektine (HANA SQL, Oracle vb.) çevirdiği taşınabilir bir SQL alt kümesi. ABAP raporlarında ve sınıflarında her zaman Open SQL yazarsınız; HANA'ya özgü SQL yalnızca AMDP'lerde görünür. C# LINQ'den temel farkı: Open SQL, tür güvenli bir sorgu oluşturucu değil, ABAP kaynağına gömülü string tabanlı bir DSL'dir.

---

## P

**Package**
ABAP'ta organizasyonel kapsayıcı — C# namespace'i veya Java paketi gibi, ancak aynı zamanda bir transport birimi. Her ABAP nesnesi (sınıf, tablo, program) bir pakete aittir. Paketler, hangi nesnelerin bir Transport Request üzerinde birlikte taşınacağını kontrol eder. Nesne: `DEVC`. Transaction: `SE80`.

---

## R

**RAP (RESTful Application Programming)**
SAP'ın modern uygulama geliştirme modeli; yeni geliştirmelerde SEGW tabanlı OData'nın yerini alır. CDS view'ları + behavior definition'lar + service definition'lar üzerine inşa edilmiştir. Güçlü tipli, ek açıklama destekli ve ABAP Cloud için tasarlanmıştır. ASP.NET Web Forms'tan ASP.NET Core Minimal API'ye geçiş gibi. Bkz. Bölüm 35.

**RFC (Remote Function Call)**
Function Module'leri sistem sınırları ötesinde çağırmak için kullanılan protokol — SAP sistemleri arasında ya da `pyrfc` veya SAP JCo connector gibi kütüphaneler aracılığıyla .NET / Java / Python'dan SAP'a. gRPC veya adlandırılmış kanal gibi, ancak SAP'a özgü. Bkz. Bölüm 12.

**RICEF**
SAP proje kapsamlandırmada kullanılan kısaltma: **R**eports (Raporlar), **I**nterfaces (Arayüzler), **C**onversions (Dönüşümler), **E**nhancements (Geliştirmeler), **F**orms (Formlar). Bunlar, ABAP geliştirme çalışmasının altı ana kategorisidir. Birisi "kapsamda kaç RICEF var?" diye sorarsa bu nesne türlerinin sayısını istiyor demektir. Kısaltmayı bilmek sizi birinci günden itibaren deneyimli gösterir.

---

## S

**S/4HANA**
SAP'ın yalnızca SAP HANA üzerinde çalışan mevcut nesil flagship ERP'si. SAP ECC'nin halefi. Geliştiriciler için önemli: S/4HANA daha katı kodlama kuralları uygular (belirli tabloların doğrudan okunmaması, SELECT * kullanılmaması, klasik API'lerin kullanımdan kaldırılması) ve CDS + RAP + ABAP Cloud'a yönlendirir.

**SAPGUI**
SAP sistemleriyle etkileşim için kalın istemci masaüstü uygulaması — menüler, transaction kodları, ekranlar. Formlarla bir terminal emülatörü gibi. Geliştiriciler kod yazarken ADT (Eclipse) kullansalar bile pek çok görev için bunu günlük olarak kullanmaya devam eder. Daha yeni sistemlerde SAP Business Client veya Fiori Launchpad da kullanılabilir.

**SEGW (SAP Gateway Service Builder)**
Entity türleri tasarlayarak ve bunları ABAP koduna bağlayarak OData servisleri oluşturmaya yarayan klasik transaction. DPC/MPC sınıfları üreten bir kod üretimi sihirbazı gibi. Yeni geliştirmelerde RAP tarafından geçersiz kılınıyor, ancak mevcut sistemlerde hâlâ son derece yaygın. Bkz. Bölüm 18.

**Selection Screen**
SAP'ın bir rapordaki `SELECT-OPTIONS` ve `PARAMETERS` ifadelerinden otomatik olarak oluşturduğu parametre giriş ekranı. Rapor çalışmadan önce filtre ölçütlerini toplayan bir Windows iletişim kutusu gibi. HTML yok, controller yok — SAP onu tanımlamalardan oluşturur.

**Smartforms / Adobe Forms**
SAP'ın yazdırılabilir form teknolojileri. Smartforms daha eski SAPScript halefidir (`SMARTFORMS` transaction'ı); Adobe Forms (`SFP` transaction'ı) Adobe LiveCycle Designer kullanan modern standarttır. RDLC / Crystal Reports / SSRS gibi, ancak SAP baskı ve arşivlemeyle derin entegrasyona sahip. Bkz. Bölüm 10.

**SMOD / CMOD**
`SMOD` = kullanılabilir SAP user exit'lerini görüntüleme (eski geliştirme yaklaşımı). `CMOD` = bu exit'leri etkinleştiren geliştirme projelerini oluşturma/yönetme. Klasik (BAdI öncesi) geliştirmeler için kullanılır. Bkz. Bölüm 14.

**SOAMANAGER**
SAP'ta SOAP/web servisi yapılandırmalarını ve bağlamalarını yönetmek için kullanılan transaction. Geleneksel SOAP servislerini tüketirken veya yayımlarken kullanılır.

**SPROXY (Service Proxy)**
Harici bir SOAP web servisini ABAP'tan çağırabilmek için WSDL'den ABAP proxy sınıfları oluşturmaya yarayan transaction. Bir WCF istemci proxy'si oluşturmak için `dotnet-svcutil` kullanmak gibi. Bkz. Bölüm 19.

**Structure**
Alanları bir araya getiren bir DDIC tipi (veya yerel olarak tanımlanmış tip) — C#'taki `struct` veya Python'daki `dataclass` gibi. Yapıların **değer semantiği** vardır: birini diğerine atamak tüm alanları kopyalar. Bir internal table'ın tek satırı olarak kullanılan yapıya **work area** denir.

---

## T

**Transaction Code (T-Code)**
SAPGUI ekranlarında doğrudan bir işleve atlamak için SAP komut alanına yazdığınız kısa kod — SAPGUI ekranları için bir klavye kısayolu veya URL yolu gibi. `SE11` DDIC'i açar, `SE38` ABAP düzenleyicisini açar, `SE37` Function Module bakımını açar vb. Tam başvuru için bkz. Ek C.

**Transport Request**
Sistem ortamı üzerinden birlikte hareket eden ABAP nesneleri (sınıflar, tablolar, programlar, özelleştirme girdileri) paketi (DEV → QAS → PRD). Bir pull request + dağıtım paketi gibi: DEV'de geliştirirsiniz, transport'u serbest bırakırsınız ve Basis ekibi onu üretime alır. ABAP'ta oluşturduğunuz her şey bir transport'a atanır. Transaction'lar: `SE01`, `SE09`, `SE10`, `STMS`.

---

## U

**UI5 (SAPUI5 / OpenUI5)**
SAP'ın JavaScript UI çerçevesi — yüzlerce kurumsal düzey bileşen ve derin SAP sistemi entegrasyonuyla birlikte React veya Angular gibi. Fiori uygulamaları UI5 üzerine inşa edilmiştir. OpenUI5, açık kaynak sürümüdür. Yalnızca OData servislerini tüketiyorsanız ABAP'a dokunmadan UI5 uygulamaları yazabilirsiniz.

---

## W

**Work Area**
Bir internal table'dan okunan veya ona yazılan tek bir satırlık veriyi tutan değişken. C#'ta bir listeden doldurulan `Customer customer;` değişkeni gibi. ABAP'ta: `DATA ls_customer TYPE zcustomer.` — `ls_` "yerel yapı" / work area için geleneksel ön ektir.

**Work Process**
SAP uygulama sunucusu içinde aynı anda bir isteği işleyen işletim sistemi düzeyinde süreç. SAP'ın sabit bir iş süreci havuzu vardır (Dialog, Background, Update, Enqueue, Spool). İş parçacığı havuzu çalışanı gibi, ancak daha ağır — `SM50`'de görebilirsiniz. Bunları anlamak, performans ve kilitlenme sorunlarını hata ayıklarken önemlidir.

---

## Bilinmesi gereken ek terimler

**Activation** — Bir ABAP nesnesi kullanılabilir olmadan önce *etkinleştirilmesi* gerekir (derleme gibi). ADT'de: Ctrl+F3. Yeni tablonuzu veya sınıfınızı bulamıyorsanız muhtemelen etkinleştirmeyi unuttunuz.

**BASIS** — SAP sistem yönetimi ekibi. Transport'ları, kullanıcı yetkilendirmelerini, sistem kopyalarını ve çekirdek yükseltmelerini yönetirler. Basis ile iyi bir ilişki kurun; üretim ortamına nelerin gireceğine onlar karar verir.

**Client-Independent vs Client-Dependent** — Bazı DDIC nesneleri (tablo *tanımları* gibi) client bağımsızdır (tüm client'larda mevcuttur). Çoğu iş tablosundaki veri satırları client bağımlıdır (her client'ın kendi satırları vardır). Çapraz-client tablolar (örn. `T001` şirket kodları) herhangi bir client'tan aynı görünür.

**Enhancement Spot** — SAP standart kodunda özel mantık enjekte edebileceğiniz tanımlanmış bir kanca noktası; BAdI veya Örtük Geliştirme aracılığıyla. Ara yazılım boru hattı ekleme noktası gibi.

**ICF (Internet Communication Framework)** — Gelen web isteklerini işleyen SAP'ın HTTP sunucusu çerçevesi. SAP içindeki OData, REST, SOAP ve özel HTTP işleyicileri için kullanılır.

**Message Class** — Numaralı hata/uyarı/bilgi/başarı mesajları için kapsayıcı (`SE91`). Kullanıcıya yönelik mesajlar için kaynak dosyaları gibi. ABAP'ta `MESSAGE E001(zmessage_class)` olarak referans alınır.

**Namespace** — Müşterilere ait SAP nesneleri müşteri namespace'inde (`Z*` veya `Y*` ön eki) veya kayıtlı bir namespace'de (`/COMPANY/`) olmalıdır. Şirketinizin kayıtlı bir namespace'i yoksa nesne adlarını her zaman `Z` ile başlatırsınız.

**Screen Painter** — SAPGUI / SE80 içindeki Module Pool programları için görsel ekran tasarımcısı. WinForms tasarımcısı gibi, ancak daha eski ve biraz tuhaf.

**SE16N / SE16** — Tablo veri tarayıcısı transaction'ları. SE16N daha yeni sürümdür. SQL Server Management Studio'da `SELECT * FROM table` çalıştırmak gibi — geliştirme ve hata ayıklama sırasında tabloda ne olduğunu kontrol etmenin hızlı yolu.

**SY (System fields)** — `SY` yapısı (örn. `sy-subrc`, `sy-datum`, `sy-uname`, `sy-tabix`) ABAP'ın ortam değişkenleri karşılığıdır: çalışma zamanı tarafından otomatik olarak doldurulur. Başarısız olabilecek ifadelerden sonra her zaman `sy-subrc`'yi kontrol edin.

---

*[← İçindekiler](../content.md) | [← Önceki: Ek A: C#/Python ↔ ABAP Hızlı Başvuru](appendix-a-csharp-abap-cheatsheet.md) | [Sonraki: Ek C: Transaction Kodu Referansı →](appendix-c-transaction-codes.md)*
