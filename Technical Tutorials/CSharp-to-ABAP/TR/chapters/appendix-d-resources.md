# Ek D: Kaynaklar, Pratik Sistemler & İş Hazırlığı

*Bu kitabı bitirmekten mülakata girmeye — ve işin ilk ayını atlatmaya — ihtiyacınız olan her şey.*

---

## 🛠️ Pratik sistemler: gerçek bir SAP bulun ve kullanmaya başlayın

ABAP'ı yalnızca okuyarak öğrenemezsiniz. Kod yazabileceğiniz, bir şeyleri bozup düzeltebileceğiniz ve transaction çalıştırabileceğiniz canlı bir sisteme ihtiyacınız var. İşte gerçekçi seçenekleriniz.

### Seçenek 1 — ABAP Platform Trial (Docker "Developer Edition") — klasik ABAP için önerilen

SAP, ABAP uygulama sunucusunun resmi bir topluluk sürümünü Docker konteyneri olarak yayımlamaktadır. Size tam bir on-prem tarzı sistem sunar: kendi dizüstü bilgisayarınızda çalışan `SE11`, `SE37`, `SE38`, `SE80`, `SEGW`, `WE02` — eksiksiz klasik araç seti.

**Ne elde edersiniz:** Geliştirme client'ı, gerçek ABAP stack, SAPGUI bağlantısı ve ADT (Eclipse) desteğiyle tam işlevli bir SAP sistemi.

**Nasıl bulunur:**
- SAP Community web sitesinde veya GitHub'da **"ABAP Platform Trial SAP Developer Edition"** arayın.
- Resmi GitHub organizasyonu **`SAP-samples`** — **`abap-platform-trial-docker`** veya benzer adlı depoyu arayın.
- SAP ayrıca trial sürümü VirtualBox görüntüsü olarak da yayımlar — SAP Community veya SAP Support Portal indirme alanında **"SAP NetWeaver AS ABAP Developer Edition"** arayın.

**Minimum sistem gereksinimleri:** ~16 GB RAM önerilir (8 GB mümkün ama yavaş), 100+ GB disk alanı, Docker Desktop veya VirtualBox.

**Kurulum özeti (Docker yolu):**
1. Docker Desktop'ı kurun.
2. SAP ABAP Platform Trial imajını çekin (talimatlar GitHub deposunun README'sinde).
3. Konteyneri başlatın — veritabanı başlarken ilk açılış 5–15 dakika sürer.
4. SAPGUI'yi indirin (SAP tarafından geliştirme kullanımı için ücretsiz — "SAP GUI for Windows download" arayın).
5. `localhost`, sistem numarası `00`, client `001` için bir SAPGUI bağlantısı yapılandırın.
6. Varsayılan geliştirici kimlik bilgileriyle giriş yapın (repo README'sinde listelenmiştir).
7. ADT'yi (Eclipse + ABAP Development Tools eklentisi) kurun ve aynı sistemi ABAP projesi olarak ekleyin.
8. Bitti — dizüstü bilgisayarınızda gerçek bir ABAP sisteminiz var.

> ⚠️ SAP bu imajları zaman zaman günceller ve imaj adı değişir. Bu özet yerine her zaman resmi SAP deposundaki güncel README'yi takip edin.

---

### Seçenek 2 — SAP BTP ABAP Environment Trial

SAP, **BTP ABAP Environment'ın ücretsiz deneme sürümünü** sunar — modern geliştirmeye (ABAP Cloud, RAP, CDS, yalnızca Tier 1 API yüzeyi) odaklanan bulut barındırmalı bir ABAP sistemi. SAPGUI yok — tamamen ADT (Eclipse) veya SAP Business Application Studio'daki ABAP Geliştirme araçlarında çalışırsınız.

**Ne elde edersiniz:** Modern ABAP Cloud yetenekleri: CDS, RAP, ABAP Unit, yayımlanmış API'ler. ABAP Cloud'un izin verilen yüzey alanı olmadığı için klasik transaction'ları kullanamazsınız (SE80, SEGW, özel FM'ler için SE37).

**Nasıl bulunur:**
- **trial.sap.com**'a gidin ve ücretsiz bir BTP deneme hesabı oluşturun.
- BTP Cockpit'te **Boosters**'a gidin ve **"Prepare an Account for ABAP Trial"** arayın — ortamı sizin için otomatik yapılandırır.
- ADT'yi (Eclipse) sistem URL'si ve BTP kimlik bilgilerinizi kullanarak sağlanan sisteme bağlayın.

**En çok şunlar için uygundur:** Bölüm VIII içeriğini (RAP, CDS) pratiğe dökmek ve ABAP Cloud kısıtlamalarını anlamak. Bölüm II–VII için Docker Developer Edition ile birleştirin.

---

### Seçenek 3 — Sistem erişimli openSAP kursları

Birçok **openSAP** kursu (SAP'ın ücretsiz MOOC platformu — **"openSAP"** arayın) kursun bir parçası olarak geçici ABAP sistemi erişimi içerir. openSAP platformunda şu anda çalışan veya kendi hızınızda ilerlenen kursları kontrol edin. Aranacak temel kurs başlıkları:

- **"Writing Testable Code for ABAP"** — ABAP Unit, temiz kod
- **"Building Apps with the ABAP RESTful Application Programming Model"** — sıfırdan RAP
- **"Efficient Development with ABAP"** — modern ABAP sözdizimi
- **"Introduction to Application Development Using ABAP"** — giriş seviyesi, zihinsel modeli oluşturmak için iyi

Bu kurslar ücretsizdir; sistem erişimi (dahil olduğunda) geçicidir (genellikle kursun süresi kadar).

---

## 📚 Öğrenme kaynakları: takıldığınızda nereye gideceğiniz

### Resmi belgeler

| Kaynak | Neleri kapsar | Nasıl bulunur |
|---|---|---|
| **SAP Help Portal** | Her SAP ürünü, transaction ve API için yetkili başvuru. Başlangıçta bunaltıcı ama vazgeçilmez. | **"SAP Help Portal"** arayın — `help.sap.com` adresinde. |
| **ABAP Keyword Documentation** | Eksiksiz ABAP dil başvurusu — her ifade, her madde, örneklerle. C# dil belirtimi gibi ama okunabilir. ADT'de: bir anahtar kelimeye imleci konumlandırın ve **F1**'e basın. SAP Help Portal'da "ABAP — Reference" altında da gezinilebilir. | `help.sap.com` → ABAP |
| **SAP RAP belgelendirmesi** | RESTful Application Programming Model için resmi kılavuz. Behavior definition'lar, projection'lar, service definition'lar, taslak işlemeyi kapsar. | **"SAP RAP ABAP RESTful Application Programming Model help"** arayın |
| **ABAP Platform sürüm notları** | Her ABAP sürümündeki yenilikler. Sistem sürümleri arasında geçiş yaparken gerekli. | SAP Help Portal |

### Topluluk ve açık kaynak

| Kaynak | Neleri kapsar | Nasıl bulunur |
|---|---|---|
| **SAP Community** | Soru-cevap forumları, blog yazıları, uzman yanıtları. SAP soruları için Stack Overflow karşılığı — soru sorun, mevcut thread'leri arayın. | **"SAP Community"** arayın |
| **[Clean ABAP Style Guide](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md)** | Temiz, modern, okunabilir ABAP yazmak için kapsamlı ve özgün bir kılavuz. ABAP için Google Style Guide gibi. SAP tarafından GitHub'da yayımlanmıştır. **Bunu yer imlerinize ekleyin — kitaptaki kod incelemeleri ve mülakatlar için en yararlı tek bağlantı.** | `github.com/SAP/styleguides` → `clean-abap/CleanABAP.md` |
| **abapGit** | ABAP için açık kaynaklı Git istemcisi — ABAP nesnelerini sürüm kontrolüne almanızı ve GitHub üzerinden kod paylaşmanızı sağlar. Portföy projeleri için gerekli. | **"abapGit"** arayın — proje `abapGit.org` adresinde ve GitHub deposu `abapGit/abapGit`. |
| **ABAP Open Checks (abaplint)** | abapGit ile CI pipeline'larında kullanılabilen ABAP kodu için statik analiz aracı. ABAP için Roslyn analizörleri gibi. | GitHub'da **"abaplint"** arayın. |
| **blogs.sap.com** | SAP'ın resmi blog platformu — binlerce uygulayıcı tarafından yazılmış öğretici, derinlemesine analiz ve nasıl yapılır kılavuzu. Yeni yazıları filtrelerseniz yüksek sinyal-gürültü oranı. | `blogs.sap.com` |

---

## 🧭 İş hazırlığı: okuyucudan işe alınmaya

### İşe alanların gerçekten taradığı beceriler

SAP ABAP iş ilanları öngörülebilir bir beceri kümesi etrafında kümelenir. Çoğunlukla belirleyici olan sırayla en sık görülenler:

| Beceri alanı | Mülakatta test edilenler | Bu kitaptaki konumu |
|---|---|---|
| **ABAP sözdizimi & OOP** | Sınıf yazma, internal table kullanma, interface uygulama | Bölümler 6, 7, 11 |
| **DDIC** | Tablo/yapı oluşturma, veri elemanları ile domain'leri açıklama | Bölüm 5 |
| **ALV raporları** | Sıfırdan CL_SALV_TABLE raporu oluşturma | Bölüm 8 |
| **Hata ayıklama** | `/h` kullanma, çağrı yığınında gezinme, izleme noktaları ayarlama | Bölüm 4, Ek C |
| **BAPI / RFC** | BAPI çağırma, return tablosunu işleme, commit work | Bölümler 12, 13 |
| **Geliştirmeler** | BAdI ile user exit arasındaki farkı açıklama, BAdI uygulama | Bölüm 14 |
| **OData / SEGW** | Basit GET_ENTITY / GET_ENTITYSET servisi oluşturma | Bölümler 18–23 |
| **CDS Views** | Association ve annotation'larla temel CDS view yazma | Bölüm 16 |
| **RAP** | Behavior definition'ı açıklama, managed ve unmanaged farkı | Bölüm 35 |
| **Transport yönetimi** | DEV→QAS→PRD akışını açıklama, transport request nedir | Ek B, C |
| **SQL / performans** | SELECT *'den kaçınma, FOR ALL ENTRIES'i doğru kullanma, ST05'i açıklama | Bölümler 6, 7 + Ek C |

---

### Yaygın mülakat soruları — ve nasıl cevaplanır

**"Internal table nedir? Yapıdan farkı nedir?"**
Internal table, bellekte bir satır listesidir (`List<T>` gibi); yapı ise tek bir satırın tanımıdır (`struct` gibi). Work area (`ls_*`) aynı anda bir satır tutar; internal table (`lt_*`) birçok satır tutar.

**"`sy-subrc` nedir ve neden her zaman kontrol etmelisiniz?"**
Sistem dönüş kodudur — veritabanı ifadeleri, `READ TABLE`, `CALL FUNCTION` vb. tarafından başarıda `0`, başarısızlıkta sıfırdan farklı bir değere ayarlanır. Çoğu başarısızlık için otomatik exception yoktur; `sy-subrc`'yi kontrol etmezseniz hatalı verilerle sessizce devam edersiniz.

**"BAPI, RFC ve Function Module arasındaki farkı açıklayın."**
Function Module, temel yeniden kullanılabilir kod birimidir. RFC etkin olmak, başka bir sistemden veya programdan uzaktan çağrılabilmek anlamına gelir. BAPI, SAP'ın resmi, kararlı bir iş nesnesi interface'i olarak yayımladığı RFC etkin Function Module'dür (versiyonlu, belgelenmiş, standart parametre kuralıyla).

**"BAdI nedir ve ne zaman user exit yerine onu kullanırsınız?"**
BAdI, modern geliştirme çerçevesidir — nesne yönelimli, birden fazla eş zamanlı uygulamayı destekler, SE18/SE19 aracılığıyla yönetilir. User exit'ler klasik yaklaşımdır (SMOD/CMOD) — prosedür tabanlı, her exit için yalnızca bir uygulama. Yeni geliştirmede her zaman BAdI'yi tercih edin; eski sistemlerde genellikle SAP standart kodunun hangisini çağırdığını kullanmak zorunda kalırsınız.

**"Transport request nedir ve bir nesneyi ona eklemeyi unutursanız ne olur?"**
Transport request, ABAP nesnelerini DEV'den QAS'a ve PRD'ye taşıyan birimdir. Bir nesneyi transport olmadan oluşturursanız (yerel `$TMP` paketi), yalnızca DEV'de bulunur ve üretime hiçbir zaman gidemez. Bu, "geliştirmede çalışıyor ama üretimde çalışmıyor" durumlarına neden olan yaygın bir başlangıç hatasıdır.

**"STANDARD, SORTED ve HASHED internal table arasındaki fark nedir?"**
STANDARD = dizi benzeri, doğal sıralama yok, O(n) doğrusal arama. SORTED = anahtar alanlarda otomatik sıralanmış, O(log n) ikili arama. HASHED = benzersiz anahtar üzerinde hash haritası, O(1) arama. Büyük tablolarda hızlı anahtar tabanlı okuma gerektiğinde HASHED; hem sıralı düzen hem hızlı arama gerektiğinde SORTED; aksi takdirde STANDARD kullanın.

**"`COMMIT WORK` ne yapar ve ne zaman çağırmanız gerekir?"**
Tamponlanmış veritabanı değişikliklerini kalıcı depolamaya aktarır. Veri oluşturan/değiştiren bir BAPI çağırdıktan sonra, kendi INSERT/UPDATE/DELETE ifadelerinizden sonra `COMMIT WORK` çağırmalısınız. Yapmazsanız verileriniz doğru şekilde eklenmiş gibi görünür ama oturum sona erdiğinde geri alınır.

**"Üretimdeki bir short dump'ı nasıl hata ayıklarsınız?"**
`ST22`'ye gidin, dump'ı tarih/kullanıcı/program bazında bulun, hata mesajını ve oluştuğu noktayı okuyun. Üretimde etkileşimli hata ayıklayıcı çalıştıramazsınız — dump metnini analiz eder, programa loglama ekler ve muhtemelen DEV'de yeniden üretirsiniz.

**"CDS ile Open SQL arasındaki fark nedir?"**
Open SQL, çalışma zamanında veri okuma/yazma için ABAP ifade sözdizimi. CDS (Core Data Services), veritabanının görünüm olarak yürüttüğü yeniden kullanılabilir sanal veri modellerini (view'lar, association'lar, ek açıklamalar) geliştirme zamanında tanımlar. Şöyle düşünün: Open SQL kodda bir LINQ sorgusu gibi; CDS view, aynı zamanda OData ve yetkilendirme için veri sözleşmesi olan bir EF Core view'u gibi.

---

### Önerilen 10 haftalık çalışma planı

Bunu kitabın bölümlerine eşleyin. Tempoyu ayarlayın — ABAP'ta tam zamanlı çalışıyorsanız 1–3. haftaları sıkıştırın.

| Hafta | Odak | Kitap bölümleri | Hedef |
|---|---|---|---|
| **1** | SAP dünyası ve araçlar | Önsöz, Bl. 1–3, Bl. 4 | SAP'ın ne olduğunu anlama; ADT + Docker trial sistemi kurma |
| **2** | DDIC & ABAP temelleri | Bl. 5, Bl. 6 | Tablo oluşturma, program yazma, sy-subrc'yi anlama |
| **3** | Modern ABAP sözdizimi | Bl. 7 | VALUE #(), COND, SWITCH, satır içi DATA(), string şablonlarında akıcı olma |
| **4** | Raporlar & ALV | Bl. 8 | Çalışan bir CL_SALV_TABLE raporu oluşturma — abapGit üzerinden GitHub'a koyma |
| **5** | OOP & Function Module'ler | Bl. 11, Bl. 12, Bl. 13 | Sınıf hiyerarşisi yazma; SE37'de BAPI_ACC_DOCUMENT_POST çağırma |
| **6** | Geliştirmeler & taşıma | Bl. 14, Bl. 15 | BAdI uygulama; BDC kaydını anlama |
| **7** | CDS, AMDP, IDoc'lar | Bl. 16, Bl. 17, Bl. 20 | Association'lı bir CDS view yazma; IDoc akışını anlama |
| **8** | OData temelleri + SEGW | Bl. 18, Bl. 21, Bl. 22, Bl. 23 | Uçtan uca GET_ENTITYSET servisi oluşturma; `/IWFND/GW_CLIENT`'ta test etme |
| **9** | OData derinlemesi | Bl. 24–31 | Filtre/arama, CREATE, association, function import ekleme |
| **10** | RAP + son rötuşlar | Bl. 34, Bl. 35 | Managed RAP senaryosu oluşturma; Ek A–C'yi gözden geçirme; taklidi mülakat |

> 💡 **4., 8. ve 10. haftaların sonunda** abapGit aracılığıyla çalışan kodu herkese açık bir GitHub deposuna commit edin. Bu üç portföy öğesi — ALV raporu, OData servisi ve RAP uygulaması — işi yapabildiğinizin somut kanıtıdır. CV'nizde bunları belirtin ve mülakatlar sırasında üzerinden geçin.

---

## 📌 Başvurmadan önce hızlı portföy kontrol listesi

- [ ] Sınıf, miras ve interface gösteren çalışan ABAP OOP programı
- [ ] Özel Z-tablosundan veri okuyan ALV raporu (CL_SALV_TABLE)
- [ ] Uygun hata yönetimi ve `COMMIT WORK` ile en az bir BAPI çağrısı
- [ ] BAdI uygulaması (basit bile olsa geliştirme çerçevesini bildiğinizi gösterir)
- [ ] GET_ENTITY ve GET_ENTITYSET'i uygulanmış bir OData servisi
- [ ] En az bir association'a sahip CDS view
- [ ] Temel düzeyde de olsa managed RAP senaryosu (SAP'ın nereye gittiğini anladığınızı gösterir)
- [ ] Açık ve net bir README ile abapGit aracılığıyla yukarıdakilerin tamamı herkese açık GitHub deposunda

---

## Kapanış sözü

Dışarıdan bakıldığında gerçekten tuhaf, bir kez yerine oturduğunda gerçekten büyüleyici bir dil ve ekosistem seçtiniz. ABAP 40 yaşındadır ve her gün dünyanın ticari işlemlerinin önemli bir bölümünü işlemeye devam etmektedir. Üzerinde çalışacağınız sistemler önemli, düzelteceğiniz hatalar önemli ve geliştirdiğiniz beceriler gerçek talep görüyor.

Başlangıçta yokuş çok dik — söz dağarcığı yabancı, araçlar alışılmadık ve veritabanı-öncelikli felsefe durum bilgisiz web servisleri oluşturarak öğrendiklerinizin tam tersine gidiyor. Ama sıfırdan başlamıyorsunuz. Bir geliştiricinin nasıl düşüneceğini zaten biliyorsunuz: bir sorunu nasıl parçalara ayırırsınız, tanımadığınız kodu nasıl okursunuz, göremediğinizi nasıl hata ayıklarsınız.

Bu kitaptaki her şey sadece **zaten yaptığınız şeyler için yeni yazımlar.**

Gidin bir şeyler yapın. Bozun. Düzeltin. Gönderin.

Bunu yapabilirsiniz. 🧭

---

*[← İçindekiler](../content.md) | [← Önceki: Ek C: Transaction Kodu Referansı](appendix-c-transaction-codes.md)*
