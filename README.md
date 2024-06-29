## Proje Özeti

"Dinamik Veri Aktarımı" projesi, bir kaynak veritabanından hedef veritabanına dinamik veri aktarımı sağlayan ve Gerçek Zamanlı Güncellemeler ile Yavaş Değişen Boyutları (SCD) içeren bir Veri Ambarı (DWH) çözümüdür. Proje, iki ana pakete ayrılmıştır:

1. **Dinamik Veri Aktarımı (Dinamik Veri Aktarımı)**
2. **Yavaş Değişen Boyut (SCD)**

### 1. Dinamik Veri Aktarımı

Dinamik Veri Aktarımı paketi, verileri kaynak veritabanından hedef veritabanına aktarmakla sorumludur. Süreç birkaç ana adımdan oluşur:

- **Dump Tablosunu Temizle (Truncate Dump Table)**: Bu adım, yeni veriler için dump tablosunu temizler.
  - **Görev**: TRUNCATED DUMP
  - **Açıklama**: Hedef veritabanında dump tablosunu temizlemek için bir SQL komutu çalıştırır.
  - **SQL Komutu**: `TRUNCATE TABLE [dbo].[OLE DB Destination Dump]`

- **SQL Görevi Çalıştır (Execute SQL Task)**: Bu görev, yalnızca en güncel verilerin işlendiğinden emin olmak için kaynak tablodan en son tarihi alır.
  - **Görev**: Execute SQL Task
  - **Açıklama**: Kaynak tablodan maksimum tarihi almak için bir SQL komutu çalıştırır.
  - **SQL Komutu**: `SELECT MAX(ISNULL(createddate,updateddate)) as MaxDate FROM [dbo].[OLE DB Destination]`
  - **Sonuç Seti**: Tek satır

- **Veri Akış Görevi (Data Flow Task)**: Bu görev, verileri kaynak tablodan dump tablosuna aktarır.
  - **Görev**: Data Flow Task
  - **Açıklama**: Verileri kaynak tablodan hedef veritabanındaki dump tablosuna aktarır.

- **Güncelleme Görevi (Update Task)**: Bu adım, dump tablosundan gelen yeni verilerle hedef veritabanını günceller.
  - **Görev**: UPDATE
  - **Açıklama**: En son tarihe dayalı olarak dump tablosundan gelen verilerle ana hedef tablosunu günceller.
  - **SQL Komutu**: Yalnızca daha güncel kayıtların güncellendiğinden emin olmak için uygun bir SQL UPDATE komutu.

### 2. Yavaş Değişen Boyut (SCD)

Yavaş Değişen Boyut paketi, boyut tablolarındaki tarihsel veri değişikliklerini yönetmekle sorumludur. Bu, boyut verilerindeki değişikliklerin zamanla izlenmesini sağlar.

- **Değişiklikleri Belirleme**: Mevcut veriler ile yeni verileri karşılaştırarak boyut verilerindeki değişiklikleri belirleme.
- **Kayıtları Ekleme/Güncelleme**: Belirlenen değişikliklere göre yeni kayıtları ekleme ve mevcut kayıtları güncelleme.
- **Tarihsel İzleme**: Boyut verilerindeki değişikliklerin tarihsel kayıtlarını tutarak verilerin zaman içindeki evrimini tam olarak gösterme.

### Yapılandırma

- **Bağlantı Yöneticileri**: Proje, farklı veritabanlarına bağlantıları yönetmek için birkaç bağlantı yöneticisi kullanır (örneğin, `LocalHost.AdventureWorks2022`, `LocalHost.MainDB`, `LocalHost.TestDB`).
- **Değişkenler**: Paket, `MaxDate` gibi değişkenleri kullanarak maksimum tarih değerini görevler arasında saklar ve iletir.

### Çalıştırma

Projeyi çalıştırmak için:

1. SQL Server Data Tools (SSDT) içinde çözüm dosyasını (`Dinamik Veri Aktarimi.sln`) açın.
2. Bağlantı yöneticilerini, çevreniz için uygun bağlantı dizeleri ile yapılandırın.
3. SSIS paketlerini (`Dinamik Veri Aktarımı.dtsx` ve `Slowly Changing Dimension.dtsx`) sırayla çalıştırın.

### Notlar

- Hedef tablolarda (`[dbo].[OLE DB Destination Dump]` ve `[dbo].[OLE DB Destination]`) hedef veritabanında mevcut olduğundan emin olun.
- SQL komutlarını, belirli tablo şemalarınıza ve iş mantığınıza uygun şekilde değiştirdiğinizden emin olun.
- Paketleri üretim ortamına dağıtmadan önce, veri aktarımı ve SCD gereksinimlerinizi karşıladığını doğrulamak için bir geliştirme ortamında test edin.
