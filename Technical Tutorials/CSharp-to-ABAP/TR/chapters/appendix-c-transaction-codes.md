# Ek C: Transaction Kodu Referansı

*Her gün yazacağınız kodlar — gruplandırılmış, açıklanmış ve gizeminden arındırılmış.*

---

## 📌 Transaction kodu nedir?

Transaction kodu (T-code), SAP'ta gezinmenin en hızlı yoludur. SAPGUI'nin sol üst köşesindeki komut alanına yazın ve Enter'a basın — doğrudan oraya gidersiniz. Menü yok, gezinti ağacı yok.

> 💡 **Başlamadan önce gezinti ipuçları:**
> - Mevcut ekranı kapatıp yenisini açmak için kodun önüne `/n` yazın: `/nSE11`
> - **Yeni bir SAPGUI penceresinde** (yeni oturum) açmak için önüne `/o` yazın: `/oSE37`
> - Oturumu kapatmak için `/nEX` yazın. Mevcut oturumu silmek için `/i` yazın.
> - SAP Easy Access menüsündeyseniz kodu doğrudan da girebilirsiniz — eğik çizgi gerekmez.
> - ADT'de (Eclipse) çoğu işleve repository tarayıcısı üzerinden erişirsiniz; ancak ADT'nin karşılamadığı işlemler için SAPGUI penceresine T-code yazmaya devam edersiniz.

---

## 🛠️ Geliştirme

| T-Code | Ne için kullanılır |
|---|---|
| `SE11` | **Data Dictionary** — tablo, yapı, veri elemanı, domain, tip grubu, arama yardımı oluşturma/düzenleme. Veri modelle ilgili her şey için ilk durağınız. |
| `SE16N` | **Tablo veri tarayıcısı (yeni)** — herhangi bir tablodaki gerçek satırları görüntüleme. SSMS'te SELECT * çalıştırmak gibi. Veri yazımlarınızı doğrulamak için geliştirme sırasında kullanın. |
| `SE16` | **Tablo veri tarayıcısı (klasik)** — SE16N'nin eski sürümü; hâlâ çalışıyor, hâlâ kullanılıyor. |
| `SE24` | **Class Builder** — eski workbench'te ABAP sınıfları ve interface'leri oluşturma ve yönetme. Yeni projelerde ADT kullanırsınız, ancak SE24 hâlâ eski sınıfları görüntülemek/değiştirmek için kullanılır. |
| `SE37` | **Function Builder** — Function Module'leri oluşturma, test etme ve yönetme. BAPI'leri çağrı kodu yazmadan önce etkileşimli olarak test ettiğiniz yer de burası. |
| `SE38` | **ABAP Editor** — klasik program (rapor) düzenleyici. ABAP programları için Notepad++ gibi. Yeni geliştirme için ADT daha iyidir; SE38 hızlı kontroller için hâlâ kullanılır. |
| `SE80` | **Object Navigator / ABAP Workbench** — eski hepsi bir arada IDE. Paketlere, programlara, sınıflara, fonksiyon gruplarına ve her şeye göz atın. ADT'yi daha sık kullanırsınız; ancak SE80 tanımadığınız kod tabanlarını gezmek için kullanışlıdır. |
| `SE91` | **Message Maintenance** — message class'ları (ABAP programlarında kullanılan numaralı hata/uyarı/başarı mesajları) oluşturma ve düzenleme. |
| `SE93` | **Transaction Maintenance** — programlarınızı veya ekranlarınızı çağıran yeni transaction kodları oluşturma. |
| `ADT` | T-code değil — **ABAP Development Tools**, Eclipse eklentisidir. Eclipse'i açın, sisteminize bağlanın ve orada geliştirin. Sözdizimi vurgulama, kod tamamlama, yeniden düzenleme, birim testleri ve ABAP hata ayıklayıcısını destekler. Tüm modern ABAP geliştirmesi için önerilen IDE. |

---

## 🔍 Veri Tarama

| T-Code | Ne için kullanılır |
|---|---|
| `SE16N` | Herhangi bir tabloyu filtre, sütun seçimi ve dışa aktarma ile tarama. Verileri hızlıca kontrol etmek için geliştiricinin en iyi arkadaşı. |
| `SE16` | Klasik tablo tarayıcısı — SE16N ile aynı amaç, biraz daha az özellikli. Yine de hızlı kontrol için SELECT yazmaktan daha hızlı. |
| `SE17` | Genel Tablo Görüntüleme — daha eski bir salt okunur görüntüleyici. Günümüzde daha az kullanılıyor ama hâlâ mevcut. |

> 💡 SE16N'de sorguyu çalıştırmak için **F8** (Execute) veya saat simgesine basın. Sorgunuz "maksimum satıra ulaşıldı" dönüyorsa satır limitini artırmak için **Settings** menüsünü kullanın.

---

## 🐛 Hata Ayıklama ve Çalışma Zamanı Analizi

| T-Code | Ne için kullanılır |
|---|---|
| `/h` | **ABAP Hata Ayıklayıcısını etkinleştir** — hata ayıklamak istediğiniz eylemi tetiklemeden önce komut alanına `/h` yazın ve Enter'a basın. SAP bir sonraki ABAP ifadesinde duracak. Bu `F9 / Hata ayıklayıcı ekle` karşılığıdır. SAPGUI'de çalışır; ADT'nin kendi entegre hata ayıklayıcısı vardır. |
| `ST22` | **ABAP Çalışma Zamanı Hata dökümü günlüğü** — kısa dökümleri gösterir (programı çökerteniakalanmamış exception'lar). İşlenmeyen exception'lar için Windows Olay Günlüğü / Application Insights gibi. "Program kısa döküm ile öldü" dendiğinde bakacağınız ilk yer. |
| `SM50` | **Work Process Genel Bakış** — şu anda çalışan iş süreçlerini gösterir. SAP sunucu süreçleri için Görev Yöneticisi gibi. Uzun süre çalışan bir programın takılıp takılmadığını veya bir dialog sürecinin tüm CPU'yu tüketip tüketmediğini görmek için kullanın. |
| `SM37` | **Background Job Monitor** — background job'ların durumunu, günlüklerini ve spool çıktısını görüntüleme. Windows Görev Zamanlayıcısı işinin son çalıştırma sonucunu kontrol etmek gibi. |
| `SAT` | **ABAP Çalışma Zamanı Analizi (Trace)** — performansı profiler: hangi ifadelerin en uzun sürdüğü, kaç DB çağrısı yapıldığı vb. Hafif profiler / SQL Server Execution Plan görüntüleyici gibi. |
| `ST05` | **SQL Trace / Performans Trace** — bir transaction sırasında yürütülen her veritabanı ifadesini yürütme süreleriyle kaydeder. Eksik indeksleri, tam tablo taramalarını ve N+1 SELECT sorunlarını bulmak için son derece değerli. SQL Server Profiler veya EF Core'un sorgu günlükleme özelliği gibi. |

---

## 🌐 OData / Gateway

| T-Code | Ne için kullanılır |
|---|---|
| `SEGW` | **SAP Gateway Service Builder** — OData servisleri oluşturmak için klasik araç. Entity türlerini tasarlayın, DPC/MPC sınıfları oluşturun, CRUD mantığını uygulayın. RAP devreye girmeden önceki sistemlerde OData geliştirmesinin vazgeçilmezi. Bkz. Bölümler 21–31. |
| `/IWFND/MAINT_SERVICE` | **OData Servislerini Yönet** — bir OData servisini kaydedin ve etkinleştirin, böylece gateway üzerinden erişilebilir olsun. Bir endpoint'i deploy etmek gibi. Servisiniz SEGW'de mevcut ama çağrılamıyorsa burada etkinleştirilmemiş olabilir. |
| `/IWFND/GW_CLIENT` | **Gateway İstemcisi** — SAP içinden doğrudan OData isteklerini test etmek için yerleşik HTTP istemcisi. Postman gibi, ancak sistemin içinde. SAP'tan ayrılmadan OData servislerinizi test etmek için vazgeçilmez. |
| `/IWFND/ERROR_LOG` | **Gateway Hata Günlüğü** — OData servisi çağrılarıyla oluşan hataları gösterir. Fiori uygulamanız veya harici istemciniz "500 İç Sunucu Hatası" dediğinde bakacağınız ilk yer. |

---

## 📨 IDoc / ALE

| T-Code | Ne için kullanılır |
|---|---|
| `WE02` | **IDoc Listesi** — IDoc'ları türe, duruma, tarihe ve partnera göre tarama. Bir IDoc'un oluşturulup oluşturulmadığını ve doğru şekilde işlenip işlenmediğini kontrol etmek için ana görünümünüz. |
| `WE05` | **IDoc Listeleri (genişletilmiş)** — daha fazla filtre seçeneğine sahip alternatif IDoc tarayıcısı. |
| `WE19` | **IDoc İşleme Test Aracı** — bir IDoc'u yeniden işleme veya manuel olarak tetikleme. Başarısız bir mesajı mesaj kuyruğundan yeniden çalıştırmak gibi. |
| `WE20` | **Partner Profil Bakımı** — hangi IDoc'ların hangi iş ortaklarına gidip geleceğini ve nasıl (port, mesaj türü vb.) yapılandırma. Bir mesaj aracısı konu aboneliği yapılandırmak gibi. |
| `BD87` | **ALE Mesajları için Durum Monitörü** — başarısız gelen IDoc'ları toplu olarak bulma ve yeniden işleme. Başarısız mesajların temel nedenini düzelttikten sonra kullanışlıdır. |
| `WE60` | **IDoc Türü Belgelendirmesi** — bir IDoc türünün yapısını ve alan belgelendirmesini görüntüleme. IDoc için Swagger şeması okumak gibi. |

---

## 🔌 Web Servisleri (SOAP)

| T-Code | Ne için kullanılır |
|---|---|
| `SOAMANAGER` | **SOA Manager** — SOAP web servisi yapılandırmalarını, mantıksal portları ve bağlamaları yönetme. Tüketilen veya yayımlanan bir SOAP servisi endpoint URL'ini ve kimlik doğrulamasını yapılandırmak için bunu kullanın. |
| `SPROXY` | **Enterprise Services Tarayıcısı / Proxy Oluşturma** — harici bir SOAP servisini ABAP'tan çağırabilmek için WSDL'den ABAP proxy sınıfları oluşturma. Visual Studio'da `dotnet-svcutil` çalıştırmak veya `Add Service Reference` eklemek gibi. Bkz. Bölüm 19. |

---

## 🧩 Geliştirmeler (Enhancements)

| T-Code | Ne için kullanılır |
|---|---|
| `SMOD` | **Enhancement Görüntüleme** — kullanılabilir SAP user exit'lerini (eski geliştirme çerçevesi) görüntüleme. SAP'ın standart kodda nerede kanca noktası sağladığını bulmak için kullanışlıdır. |
| `CMOD` | **Enhancement Yönetimi** — user exit'leri bir araya getiren ve etkinleştiren geliştirme *projelerini* oluşturma ve yönetme. Klasik geliştirmeler için etkinleştirme noktası. |
| `SE18` | **BAdI Builder** — BAdI tanımlarını görüntüleme: SAP'ın kanca noktasını nereye koyduğu, interface'in nasıl göründüğü. Uygulamadan önce bir interface tanımını okumak gibi. |
| `SE19` | **BAdI Uygulaması** — bir BAdI uygulaması oluşturma. Bir interface'i uygulayan bir sınıf eklemek gibi. Bkz. Bölüm 14. |

---

## 📄 Formlar

| T-Code | Ne için kullanılır |
|---|---|
| `SMARTFORMS` | **Smartforms Form Builder** — Smartforms baskı formlarını (Adobe Forms'ın öncülü) tasarlama ve yönetme. Pek çok canlı sistemde hâlâ yaygın. Bkz. Bölüm 10. |
| `SFP` | **Adobe Forms / Form Builder** — Adobe LiveCycle tabanlı baskı ve etkileşimli formları oluşturma ve yönetme. Yeni form geliştirmesi için mevcut standart. Bkz. Bölüm 10. |
| `SE71` | **SAPscript Form Painter** — en eski SAP form teknolojisi (Smartforms öncesi). Çok eski sistemlerde karşılaşırsınız. Büyük ölçüde yalnızca eski sistem kodlarında. |

---

## 📦 Veri Taşıma (Data Migration)

| T-Code | Ne için kullanılır |
|---|---|
| `SHDB` | **BDC Kaydı** — daha sonra BDC batch input olarak oynatmak üzere SAPGUI ekran oturumu kaydetme. SAP ekranları için Selenium kaydet-ve-yeniden-oynat gibi. Bkz. Bölüm 15. |
| `LSMW` | **Legacy System Migration Workbench** — BDC kaydı, BAPI çağrıları veya IDoc'lar aracılığıyla veri taşımak için yapılandırılmış araç. Nokta ve tıklama veri yükleme çerçevesi. Bkz. Bölüm 15. |
| `LTMC` | **SAP S/4HANA Migration Cockpit** — modern S/4HANA veri taşıma aracı. BDC yerine taşıma şablonları (Excel/XML) ve hazırlık tabloları kullanır. S/4HANA greenfield projelerinde LSMW'nin yerini alır. |

---

## 🚂 Transport Yönetimi

| T-Code | Ne için kullanılır |
|---|---|
| `SE01` | **Transport Organizer (Genişletilmiş)** — sistemdeki tüm transport request'lerinin tam görünümü. Transport oluşturma, görüntüleme ve yönetme. |
| `SE09` | **Workbench Transport Organizer** — ABAP geliştirme nesneleri (programlar, sınıflar, tablolar) için transport'ları görüntüleme ve yönetme. Geliştirici olarak ana transport yönetim ekranınız. |
| `SE10` | **Customizing Transport Organizer** — yapılandırma / özelleştirme değişiklikleri (kod değil) için transport'ları görüntüleme ve yönetme. |
| `STMS` | **Transport Management System** — sistemden sisteme transport kontrol merkezi. Basis, DEV'den QA'ya ve PRD'ye transport almak için bunu kullanır. Geliştirici olarak import kuyruklarını ve durumları kontrol etmek için kullanırsınız. |

---

## 🧭 Erken öğrenmeye değer diğer yararlı T-code'lar

| T-Code | Ne için kullanılır |
|---|---|
| `SU01` | Kullanıcı bakımı — kullanıcı hesapları oluşturma/değiştirme (ağırlıklı olarak Basis, ancak bilmek faydalı). |
| `SU53` | Son yetkilendirme denetimi başarısızlığını göster — "yetkisiz" hatasına hangi yetkilendirme nesnesinin neden olduğunu tam olarak söyler. Auth sorunlarını bildirirken bu ekran görüntüsünü Basis ekibinizle paylaşın. |
| `SPRO` | SAP Reference IMG — özelleştirme (yapılandırma) giriş noktası. Çoğu ABAP geliştiricisi buraya nadiren girer, ancak her fonksiyonel danışman burada yaşar. |
| `SM30` / `SM31` | Tablo Görünümü Bakımı — `V_` ön ekli görünümlerdeki özelleştirme girdilerini yapılandırma. Oluşturulan bir CRUD ekranı aracılığıyla referans veri tablosunu düzenlemek gibi. |
| `SLG1` | Uygulama Günlüğü Görüntüleyicisi — `BAL_*` function module'leriyle yazılan günlükleri okuma. |
| `SICK` | Kurulum Denetimi — sistemin sağlıklı olduğunu doğrular; nadiren gereklidir ancak bilmekte fayda vardır. |

---

*[← İçindekiler](../content.md) | [← Önceki: Ek B: Yeni Başlayanlar İçin Sözlük](appendix-b-glossary.md) | [Sonraki: Ek D: Kaynaklar & İş Hazırlığı →](appendix-d-resources.md)*
