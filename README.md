# 🏫 Dershane Otomasyon Yazılımı (Proje Raporu)

Bu proje; ön yüzde responsive HTML5/CSS3 tabanlı "Mentor" eğitim şablonunu kullanan, arka yüzde ise ilişkisel MSSQL Server veritabanı mimarisini birleştiren, bir özel öğretim kurumunun tüm operasyonel süreçlerini yönetmek amacıyla geliştirilmiş dinamik bir web uygulamasıdır.

---

## 📌 1. Genel Yapı (Proje Özeti)
Dershane Otomasyon Yazılımı; idari personelin iş yükünü azaltmak, manuel veri giriş hatalarını sıfırlamak, ders/sınıf çakışmalarını önlemek ve veri bütünlüğünü korumak üzere tasarlanmıştır. Sistem; web arayüzünden gelen dinamik istekleri arka planda ilişkisel veri tabanı kurallarıyla işleyen uçtan uca bir yönetim deneyimi sunar.

* **Arayüz Teknolojisi:** HTML5, CSS3, Bootstrap (Mentor Web Şablonu)
* **Veritabanı Yönetim Sistemi:** Microsoft SQL Server (MSSQL)
* **Tasarım Yaklaşımı:** Nesne Yönelimli Programlama (OOP) ve Veritabanı Seviyesinde İş Kuralları Tasarımı
* **Mimari Yapı:** Dinamik Web Entegrasyonu ve Parametrik Veri Yönetimi

---

## 🔍 2. Problem Tanımı
Geleneksel dershane yönetiminde; departmanların (muhasebe, öğrenci işleri, ders planlama) verileri birbirinden bağımsız Excel tablolarında veya fiziksel defterlerde tutulmaktadır. Bu durum şu kronik problemlere yol açar:
* **Kontenjan Aşımı:** Sınıfların kapasitesinden fazla öğrenci kaydedilmesi ve sınıf içi düzenin bozulması.
* **Ders ve Program Çakışmaları:** Aynı sınıfa veya aynı öğretmene aynı gün ve saatte birden fazla ders atanması.
* **Finansal Takip Zorluğu:** Öğrencilerin kalan taksit tutarlarının ve ödeme durumlarının anlık olarak süzülememesi.
* **Akademik Performans Takibi:** Öğrencilerin girdikleri deneme ve yazılı sınav sonuçlarının geçmişe dönük analiz edilememesi.

**Çözüm:** Bu proje ile tüm web arayüzü tek bir ilişkisel veritabanına bağlanmıştır. Geliştirilen SQL kuralları sayesinde sisteme hatalı, mükerrer veya mantıksız veri girişi (örn: asgari ücretin altında öğretmen maaşı veya 0-100 dışı sınav notu) web arayüzünden yapılsa dahi veritabanı seviyesinde kesin olarak engellenir.

---

## 🔬 3. Yapılan Araştırmalar & Teknik Çözümler
Geliştirme sürecinde karşılaşılan performans ve veri güvenliği sorunları, doğrudan SQL Server yetenekleri kullanılarak çözülmüştür:
* **Mükerrer Kayıtların Önlenmesi:** T.C. Kimlik Numaraları ve E-posta adresleri için `UNIQUE` kısıtlamaları getirilerek aynı kişinin sisteme iki kez kaydedilmesi önlenmiştir.
* **Sorgu Performansının Artırılması:** Veritabanında milyonlarca kayıt olsa dahi arama ve filtreleme işlemlerinin milisaniyeler içinde bitmesi için indeksleme (`NONCLUSTERED INDEX`) mimarisinden yararlanılmıştır.
* **Karmaşık Sorguların Sadeleştirilmesi:** Web tarafında uzun ve yorucu `JOIN` sorguları yazmak yerine, verileri önceden birleştiren sanal tablolar (`VIEW`) oluşturulmuştur.
* **Arayüz Güvenliği:** Doğrudan SQL cümleleri yazmak yerine parametrik yapıda çalışan Saklı Yordamlar (`STORED PROCEDURE`) tercih edilerek **SQL Injection** gibi siber güvenlik açıklarının önüne geçilmiştir.

---
## 🏗️ 4. Yazılım Mimarisi (Veri Modelleri ve Katmanlar)
Proje, ilişkisel veritabanı modeline uygun olarak aşağıdaki temel modüller ve SQL tabloları üzerinden mantıksal olarak yapılandırılmıştır:

Arayüzde yer alan HTML sayfaları ile veritabanındaki (SQL) tabloların mantıksal eşleşmesi ve veri trafiği şu mimariyle yönetilmektedir:

```text
📁 Web Arayüzü (Frontend)                📁 Veritabanı Modülleri (Backend)
├── index.html (Ana Sayfa) ----------> Genel İstatistikler (Öğrenci/Sınıf Sayıları)
├── courses.html (Kurslar) ----------> Tbl_Dersler & View_HaftalikDersProgrami
├── course-details.html -------------> sp_SinifOgrencileriniGetir (Sınıf Detayları)
├── events.html (Sınavlar/Etkinlik) -> Tbl_Sinavlar & Tbl_SinavSonuclari
└── contact.html (Kayıt/İletişim) ---> sp_OgrenciEkle (Yeni Öğrenci Kayıt Formu)
```
### 🔑 Yönetim ve Eğitim Kadrosu
`Ogretmenler` ve `Siniflar` tabloları kurumun fiziksel ve idari altyapısını yönetir. `CHK_OgretmenMaas` kısıtlaması ile öğretmen maaşlarının asgari ücret sınırının (17002.00 TL) altına düşmesi engellenirken, `CHK_SinifKontenjan` ile sınıfların fiziksel kapasitesi 5-30 kişi arasında sınırlandırılmıştır.

### 👥 Öğrenci İşleri Modülü
`Ogrenciler` tablosu, öğrencilerin kişisel ve iletişim verilerini tutarken `Siniflar` ile dinamik olarak ilişkilendirilmiştir. `contact.html` üzerindeki kayıt formu doldurulduğunda arka planda `sp_OgrenciEkle` saklı yordamı tetiklenir. Yeni kayıt esnasında `TRG_KontenjanKontrol` trigger'ı devreye girerek sınıf mevcudunu anlık denetler.

### 📅 Akademik Planlama
`Dersler` ve çoka çok (Many-to-Many) ilişkiyi çözen köprü vazifesindeki `DersProgrami` tabloları haftalık takvimi yönetir. `UC_Program` benzersizlik kısıtlaması sayesinde aynı sınıfa, aynı gün ve saatte birden fazla ders atanması (ders çakışması) veritabanı seviyesinde önlenir. Veriler web arayüzünde `courses.html` üzerinde dinamik olarak listelenir.

### 💳 Finans ve Muhasebe Modülü
`Odemeler` tablosu üzerinden öğrencilerin taksit bazlı borç ve tahsilat takibi yapılır. `CHK_Taksit` kısıtlaması ile bir eğitimin maksimum 12 taksite bölünebileceği kuralı çiğnenemez hale getirilmiş, `IX_Odemeler_Durum` indeksi sayesinde muhasebe biriminin web panelinden yaptığı "Ödenmedi" durumundaki borçlu sorgulamaları milisaniyeler seviyesine indirilmiştir.

### 📊 Ölçme ve Değerlendirme
`Sinavlar` ve `SinavSonuclari` tabloları öğrencilerin gelişim grafiklerini çıkarır. `TRG_NotKontrol` trigger'ı sayesinde sisteme 0-100 aralığı dışında hatalı not girişi yapılması engellenir. `UC_OgrenciSinavDers` kısıtlaması ile bir öğrenciye aynı sınavda aynı ders için mükerrer (çift) not girilmesinin önüne geçilir; sonuçlar `events.html` sayfasında dinamik duyurulur.

### 📊 5. Veri Tabanı Tasarımı (ER Yapısı ve İlişkiler)
<img width="989" height="809" alt="ER DİYAGRAMI" src="https://github.com/user-attachments/assets/41a6505a-7636-4a10-b466-984033f2c399" />


Sistem veri tutarlılığını (Data Integrity) korumak amacıyla üçüncü normal formda (3NF) ilişkisel bir MSSQL şeması üzerine kurulmuştur:

🔹 İlişkiler ve Silme Senaryoları (Cascading)
Sınıf - Öğrenci İlişkisi (1-N): Bir sınıfta birden fazla öğrenci bulunabilir. Siniflar tablosundan bir sınıf silindiğinde öğrencilerin sistemde kalması ve boşa çıkması için ON DELETE SET NULL kuralı uygulanmıştır.

Öğrenci - Ödemeler / Sınav Sonuçları İlişkisi (1-N): Bir öğrenci dershaneden ayrılıp kaydı tamamen silindiğinde, veritabanında çöp veri kalmaması için o öğrenciye ait tüm geçmiş finansal taksitler ve akademik notlar ON DELETE CASCADE ile otomatik olarak temizlenir.

### Sınıf - Ders İlişkisi (N-N): Sınıflar ile Dersler arasındaki "Çoka Çok" ilişkiyi çözmek için araya bir DersProgrami tablosu köprü olarak eklenmiştir.

###🔹 Tablo Kısıtlamaları ve Kurallar (Constraints)
CHK_OgretmenMaas: Öğretmen maaşlarının asgari ücret sınırının (>= 17002.00 TL) altında girilmesini engeller.
CHK_SinifKontenjan: Sınıf kontenjanlarının dershane mantığı gereği 5 ile 30 kişi arasında kalmasını zorunlu kılar.
CHK_DersSaat: Bir dersin haftalık saatinin 1 ile 10 saat arasında olmasını şart koşar.
CHK_Taksit & CHK_Durum: Ödemelerin maksimum 12 taksit olmasını ve durumun sadece 'Ödendi' veya 'Ödenmedi' değerlerini alabilmesini sağlar.
UC_Program: Aynı sınıfa, aynı gün ve saatte birden fazla ders atanmasını (ders çakışmasını) engeller.

### 🔄 6. Akış Şeması (Flowchart) ve Tetikleyiciler (Triggers)
Sistemde veri tabanı seviyesinde çalışan ve iş mantığını otomatikleştiren iki kritik tetikleyici (TRIGGER) bulunmaktadır:
1️⃣ Sınıf Kontenjan Kontrolü (TRG_KontenjanKontrol)
Yeni bir öğrenci eklendiğinde veya bir öğrencinin sınıfı güncellendikten hemen sonra tetiklenir. Sınıfın güncel öğrenci sayısı, Siniflar tablosundaki maksimum kontenjan değerini aşıyorsa işlemi ROLLBACK TRANSACTION ile iptal eder ve web arayüzüne hata fırlatır.

2️⃣ Not Kontrolü (TRG_NotKontrol)
Sınav sonuçları girilirken veya güncellenirken girilen puan değerinin mantıksal aralıkta (0 - 100) kalmasını sağlar. Hatalı bir not girişinde işlemi durdurur.
graph TD
```text
    A[Web Arayüzünden Öğrenci Kayıt İsteği] --> B[SQL INSERT Komutu Tetiklendi]
    B --> C[TRG_KontenjanKontrol Aktifleşti]
    C --> D{Mevcut Sayı > Kontenjan?}
    D -- Evet --> E[RAISERROR: Sınıf Dolu!]
    E --> F[ROLLBACK: Kayıt İptal Edildi]
    D -- Hayır --> G[COMMIT: Öğrenci Başarıyla Kaydedildi]
```

### 💻 7. Projenin Yüklenmesi ve Çalıştırılması
Gereksinimler
HTML5 / CSS3 Destekli Güncel Bir Web Tarayıcısı
MS SQL Server Management Studio (SSMS)
Backend Entegrasyonu İçin Tercih Edilen Geliştirme Ortamı (Visual Studio / VS Code vb.)

Kurulum Adımları
Depoyu Klonlayın:
git clone [https://github.com/nisasuraydin/mentor.git](https://github.com/nisasuraydin/mentor.git)

Veritabanını Hazırlayın:
Proje klasöründeki SQL script dosyasını SQL Server Management Studio (SSMS) üzerinde Execute (F5) ederek tabloları, view'ları, procedure'leri, trigger'ları ve örnek test verilerini oluşturun.

Bağlantı Dizesini (Connection String) Düzenleyin:
Projenin backend yapılandırma dosyasındaki bağlantı adresini kendi yerel SQL Server bilgilerinize göre güncelleyin:
<connectionStrings>
    <add name="DershaneDb" connectionString="Data Source=YOUR_SERVER_NAME;Initial Catalog=DershaneDB;Integrated Security=True;" />
</connectionStrings>

Projeyi Başlatın:
index.html dosyasını tarayıcınızda açarak arayüzü inceleyebilir, backend sunucunuzu ayağa kaldırarak dinamik veritabanı bağlantısını test edebilirsiniz.

### 8.Geliştirilen Arayüzden Örnek Görseller
<img width="1315" height="598" alt="otomasyon sistemi ana sayfası" src="https://github.com/user-attachments/assets/824c6df7-13f8-4275-b3ca-91dcff6eebca" />


### 9.Referanslar
Microsoft Learn, SQL Server ve ADO.NET Veri Erişimi Dokümantasyonu.
BootstrapMade, Mentor Free Education Bootstrap Template Documentation.
T.C. Çalışma ve Sosyal Güvenlik Bakanlığı, Asgari Ücret Verileri (Maaş CHECK kısıtlaması için veri kaynağı).
Database Systems: The Complete Book (Veritabanı Normalizasyonu ve Cascading Kuralları).
