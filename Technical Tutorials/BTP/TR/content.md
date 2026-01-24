# 📖 İçindekiler

## Eski ABAP'çılar için SAP BTP Mini Kitabı

---

### [Önsöz](chapters/00-preface.md)
- Bu Kitap Kimin İçin (Klasik ABAP & Fiori Geliştiricileri)
- Feynman Felsefesi: Laf Yok, Sadece Kahve & Netlik
- Bu Kitabı Nasıl Kullanmalı

---

## Bölüm I: Büyük Resim – SAP BTP'yi Anlamak

### [Kısım 1: SAP BTP Aslında Nedir?](chapters/01-what-is-sap-btp.md)
- 1.1 Bulut Araç Kutusu Kavramı
- 1.2 İçinde Ne Var: Veritabanları, AI, İş Akışları ve Entegrasyonlar
- 1.3 Joule Nerede Duruyor – SAP'ın AI Asistanı
- 1.4 Eski Ev vs. Yeni Mahalle (On-Prem vs. Bulut Zihniyeti)

### [Kısım 2: BTP Mimarisi – Yeni Apartman Binanız](chapters/02-btp-architecture.md)
- 2.1 Global Account – SAP ile Kira Sözleşmeniz
- 2.2 Directory'ler – Katları Düzenleme (Opsiyonel)
- 2.3 Subaccount'lar – Asıl Daireleriniz
- 2.4 İzolasyon Neden Önemli: Dev vs. Test vs. Prod
- 2.5 Bölgeler, Uyumluluk ve Veri Yerleşimi
- 2.6 Entitlement'lar & Kotalar – Ne Kadar Kullanabilirsiniz

---

## Bölüm II: RISE with SAP – Yönetilen Bulut Geçişi

### [Kısım 3: RISE with SAP Açıklandı](chapters/03-rise-with-sap.md)
- 3.1 RISE Aslında Nedir (Bir Ürün Değil!)
- 3.2 S/4HANA Cloud Private Edition vs. Public Edition
- 3.3 RISE Paketi: Neler Dahil
- 3.4 Altyapı & Operasyonlar – Basis Endişelerine Veda
- 3.5 Clean Core Sözü

### [Kısım 4: RISE ve BTP Nasıl Birlikte Çalışır](chapters/04-rise-and-btp.md)
- 4.1 Mahalle Görünümü: RISE Evi + BTP Uzantı Kanadı
- 4.2 RISE Sözleşmelerinde BTP Kredileri
- 4.3 ABAP/Fiori Geliştiricileri İçin Yeni Gerçeklik
- 4.4 Karşılaştırma Tablosu: Klasik On-Prem vs. RISE vs. BTP

---

## Bölüm III: ABAP Geliştiricileri İçin Temel BTP Kavramları

### [Kısım 5: Destination'lar – Bağlantılar İçin Telefon Rehberi](chapters/05-destinations.md)
- 5.1 Destination Nedir?
- 5.2 Destination'lar Neden Var (Güvenlik & Yönetişim)
- 5.3 İlk Destination'ınızı Oluşturma
- 5.4 Kimlik Doğrulama Türleri: Basic, OAuth2, Sertifikalar, API Key'ler
- 5.5 Cloud Connector ile On-Premise Erişimi
- 5.6 Akıl Sağlığınızı Koruyan İsimlendirme Kuralları

### [Kısım 6: Bulutta ABAP – ABAP Environment](chapters/06-abap-cloud.md)
- 6.1 BTP ABAP Environment Genel Bakış
- 6.2 Klasik On-Prem ABAP'tan Farkları
- 6.3 RAP (RESTful ABAP Programming) Temelleri
- 6.4 BTP ABAP vs. Klasik ABAP Ne Zaman Kullanılmalı
- 6.5 Migrasyon Yolu: On-Prem'den Kod Taşıma

### [Kısım 7: BTP'de Fiori & UI5](chapters/07-fiori-ui5-btp.md)
- 7.1 SAP Business Application Studio (BAS)
- 7.2 Fiori Uygulamalarını BTP'ye Deploy Etme
- 7.3 Backend Sistemlerine Bağlanma
- 7.4 On-Prem Fiori Launchpad'den Farklar

---

## Bölüm IV: Joule – AI Agent'ları & Skill'leri

### [Kısım 8: Joule Temelleri](chapters/08-joule-fundamentals.md)
- 8.1 Joule Nedir? (SAP AI Copilot)
- 8.2 Skill'ler vs. Agent'lar: Temel Fark
- 8.3 Skill'ler Nasıl "İş Yapar" (Action Akışı)
- 8.4 Skill'ler İçin Destination'lar Neden Gerekli

### [Kısım 9: İlk Joule Skill'inizi Oluşturma](chapters/09-first-joule-skill.md)
- 9.1 Uçtan Uca Akış
  - Adım 1: Destination Oluştur
  - Adım 2: SAP Build'de Action Project Oluştur
  - Adım 3: Joule Studio'da Skill Oluştur
  - Adım 4: Test Et ve Deploy Et
- 9.2 REST Endpoint'lerini Skill Olarak Ekleme
- 9.3 OpenAPI Spec'leri vs. Manuel Action Tanımları

### [Kısım 10: Joule Agent'ları Oluşturma](chapters/10-building-agents.md)
- 10.1 Bir Agent'ı "Akıllı" Yapan Nedir
- 10.2 Agent'lara Skill Atama
- 10.3 Çok Adımlı Akıl Yürütme ve Zincirleme
- 10.4 Agent'ları Müşteri Dökümanlarıyla Temellendirme

### [Kısım 11: Agent Yaşam Döngüsü & Deployment](chapters/11-agent-lifecycle.md)
- 11.1 Dev → Test → Prod: Standart Yaşam Döngüsü
- 11.2 Tenant'lar Arası Proje Transferi
- 11.3 SAP Build Promote Özelliği
- 11.4 Manuel + Git Versiyon Kontrolü (Danışman Favorisi)
- 11.5 Ortama Özel Konfigürasyon (Destination'lar, Modeller)

---

## Bölüm V: Çoklu Müşteri Gerçekliği – Danışman Rehberi

### [Kısım 12: BTP'de Birden Fazla Müşteriyi Yönetme](chapters/12-multi-client-management.md)
- 12.1 Klasik Tuzak: Her Şey İçin Tek Subaccount
- 12.2 En İyi Uygulama: Her Müşteri/Proje İçin Bir Subaccount
- 12.3 Çoklu Müşteri Ortamları İçin İsimlendirme Kuralları
- 12.4 Güvenlik & Erişim Yönetimi
- 12.5 Maliyet Dağıtımı ve Geri Faturalandırma

### [Kısım 13: Müşteriler Arası Deployment'lar](chapters/13-cross-customer-deployments.md)
- 13.1 Kalıp 1: Şablon + Her Müşteri İçin Yeniden Oluşturma
- 13.2 Kalıp 2: Çok Kiracılı SaaS Modu
- 13.3 Hangi Kalıp Ne Zaman Kullanılmalı
- 13.4 Karşılaştırma Tablosu: Efor, İzolasyon, Devir

### [Kısım 14: Kurumsal Müşteriler İçin C-Seviye Agent'ları Oluşturma](chapters/14-c-level-agents.md)
- 14.1 C-Seviye Agent'ı Yapan Nedir
- 14.2 Çapraz Sistem Orkestrasyonu
- 14.3 AI Unit Tüketimini Yönetme
- 14.4 Yönetici Dashboard'ları İçin Pratik İpuçları

---

## Bölüm VI: Entegrasyon & İleri Düzey Konular

### [Kısım 15: SAP Integration Suite Temelleri](chapters/15-integration-suite.md)
- 15.1 Integration Suite Ne Zaman Kullanılmalı
- 15.2 API'ler, iFlow'lar ve Event Mesh
- 15.3 BTP'yi Harici Sistemlere Bağlama

### [Kısım 16: On-Premise Erişimi İçin Cloud Connector](chapters/16-cloud-connector.md)
- 16.1 Cloud Connector Nedir?
- 16.2 Legacy Sistem Erişimi İçin Kurulum
- 16.3 Yaygın Senaryolar: S/4 On-Prem + Bulutta Joule

### [Kısım 17: Eski Alışkanlıkları Sınırlayan Clean Core Kuralları](chapters/17-clean-core.md)
- 17.1 Artık Ne Yapamıyorsunuz?
- 17.2 Uzantılar vs. Modifikasyonlar
- 17.3 Geliştirme Zihniyetinizi Adapte Etme

---

## Ekler

### [Ek A: Hızlı Referans Tabloları](chapters/appendix-a-reference-tables.md)
- A.1 BTP Hiyerarşi Kopya Kağıdı
- A.2 Destination Konfigürasyon Seçenekleri
- A.3 Joule Skill vs. Agent Karşılaştırması
- A.4 RISE vs. Klasik Karşılaştırması

### [Ek B: Eski ABAP'çılar İçin Sözlük](chapters/appendix-b-glossary.md)
- Yaygın BTP Terimleri Tercüme Edildi

### [Ek C: Faydalı Linkler & Kaynaklar](chapters/appendix-c-resources.md)
- SAP Yardım Dokümantasyonu
- SAP Community
- GitHub Örnekleri

### [Ek D: Yaygın Sorunları Giderme](chapters/appendix-d-troubleshooting.md)
- Destination Hataları
- Joule Deployment Sorunları
- Kimlik Doğrulama Problemleri

---

*[README'ye Dön](README.md)*
