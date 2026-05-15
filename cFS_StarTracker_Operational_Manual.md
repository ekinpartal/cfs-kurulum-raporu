# NASA cFS Otonom Star Tracker Entegrasyonu ve Operasyonel Arayüz Kontrol Kılavuzu

## Özet (Abstract)
Bu proje; NASA core Flight System (cFS) mimarisi üzerinde çalışan bir Otonom Star Tracker (Yıldız Takipçisi) payload'unun entegrasyonunu ve bu sistemi kontrol etmek/izlemek için geliştirilen çok katmanlı kontrol arayüzlerini içermektedir. 

Aşağıdaki kılavuz, sistemin çalıştırılması esnasında açılan fiziksel terminallerin, NASA arayüzündeki butonların ve özel geliştirilen Dashboard'un **operasyonel işlevlerini** açıklayan pratik bir kullanıcı ve sistem referans rehberidir.

---

## 🖥️ 1. Sistem Çalışma Düzeni ve Terminal Matrisi (Ubuntu/WSL Terminalleri)

Sistem tam kapasite çalıştığında arka planda **3 ila 4 farklı Ubuntu/WSL terminali** aktif görev yapar. Her terminal, uydunun ve yer sisteminin fiziksel bir katmanını temsil eder:

| Terminal Adı | Çalıştırılan Komut / Görev | Operasyonel Karşılığı |
| :--- | :--- | :--- |
| **Terminal 1: Uydunun Beyni** | `./core-cpu1` | Gerçek uydunun üzerindeki ana uçuş bilgisayarını (OBC) temsil eder. Kapatılırsa uydu çöker. |
| **Terminal 2: Yer İstasyonu** | `python3 GroundSystem.py` | NASA'nın fiziksel yer kontrol merkezindeki ana veri dağıtım sunucusunu (Router) başlatır. |
| **Terminal 3: Görsel Monitör** | `start-tracker` veya `UYDU_DASHBOARD.bat` | Uydudan gelen verileri anlık olarak çizdiren Dark-SciFi görsel takip monitörümüzdür. |
| **Terminal 4: Komut Kanalları** | `GroundSystem.py` tarafından otomatik açılır. | Yer istasyonundaki gri alt pencerelerin veri loglarının aktığı terminal ekranıdır. |

---

## 📡 2. NASA Yer İstasyonu Panelleri ve Kritik Butonlar

Yer istasyonu (Ground System) arayüzünde karşımıza çıkan gri pencereler ve bu pencerelerdeki butonların fiziksel dünyadaki görevleri şunlardır:

### 🟦 A. TO LAB (Telemetry Output Lab) Paneli
Bu panel, uydunun veri gönderme antenini aktif etmek için kullanılır.
*   **Kritik Buton:** `TO_LAB_OUTPUT_ENABLE_CC`
*   **Fiziksel Görevi:** Uydunun dış dünya ile olan telsiz/UDP bağlantısını açar. Bu butona basıp `Send` demeden önce Dashboard'a veya grafik ekranına tek bir bayt bile veri akmaz.

### 🟩 B. SAMPLE APP (CPU1) Paneli
Yıldız takipçisi (Star Tracker) payload'unu doğrudan yönettiğimiz ana komuta merkezidir.
*   **Kritik Buton:** `SAMPLE_APP_FIND_STARS_CC`
*   **Fiziksel Görevi:** Uydunun üzerindeki bilimsel kamerayı tetikler. Kamera o an hangi yöne bakıyorsa bir poz çeker, uydunun C motoru bu görseli piksel piksel tarayarak yıldızları tespit eder ve koordinatlarını anında yeryüzüne (Dashboard'a) gönderir.

---

## 🪐 3. Görsel Operasyon Paneli (V.O.D. Dashboard) Butonları ve İşlevleri

Geliştirdiğimiz PyQt5 tabanlı arayüz, uydunun canlı durumunu takip etmemizi ve görsel ayarları yapmamızı sağlar. Ekrandaki butonların işlevleri şu şekildedir:

### 🎯 A. TARGET ACTIVE SCENE (Sahne Seçici Menü)
*   **Görevi:** Uydunun kamerasını fiziksel olarak farklı bir gök bölgesine çevirmesini simüle eder.
*   **Çalışma Şekli:** Menüden "Scene 2" seçildiği an, Dashboard uydunun beynindeki tampon belleğe (`test_yildiz.bin`) yeni görseli yazar. Ardından Ground System'dan `FIND_STARS` komutu gönderildiğinde, uydu bu yeni sahneyi işlemeye başlar.

### 🔳 B. VISUAL CONTROLS (Görsel Mod Seçiciler)
*   **STANDARD VIEW:** Uydunun CMOS kamerasından gelen ham, siyah-beyaz optik görüntüyü ekrana yansıtır.
*   **THERMAL CAMERA:** Kızılötesi (Infrared) görüntüleme modunu açar. Görüntüyü bilimsel **Jet Colormap** algoritmasıyla renklendirerek, soğuk derin uzayı lacivert, parlayan sıcak yıldızları ve bulutsuları ise sarı-kırmızı tonlarda gösterir.
*   **FIT IMAGE TO SCREEN:** Görüntüyü pencere boyutuna tam olarak sığdırır. (İstendiğinde farenin tekerleği ile zoom yapılabilir veya fareyle sürükleyerek görüntü içinde gezilebilir).

### 📐 C. DYNAMIC OVERLAYS (Dinamik Vektör Katmanları)
*   **BUILD CONSTELLATION NET:** Tespit edilen yıldızları birbirine bağlayan neon mavi renkte kesikli lazer çizgilerini açar/kapatır. Yıldızlar arasındaki pikselsel mesafeleri (örn: `371.4px`) anlık olarak hesaplayıp çizgilerin ortasına yazar. (Navigasyonel nirengi hesaplaması için kullanılır).
*   **ENABLE ACTIVE RADAR SCANNER:** Ekranın merkezinden yayılan ve saniyede 30 tur dönen konik radar dalgasını aktif eder. Uydunun o an boşta durmadığını, optik sensörlerinin aktif olarak okuma yaptığını (CMOS Readout) ve çevrede tehdit aradığını gösterir.

---

## 🛠️ 4. Proje Sürecinde Giderilen Kritik Hatalar (Hata Günlüğü)

Projenin geliştirilmesi sırasında karşılaşılan ve çözülen operasyonel sorunlar şunlardır:

### 🔴 Hata 1: Sıfır Veri Akışı Sorunu (Scheduler Hatası)
*   **Belirti:** Tüm sistemler açık olmasına rağmen `CMD COUNT` artmıyor ve veri akmıyordu.
*   **Çözüm:** cFS çekirdek tablosunda (`sch_lab_table.c`) Housekeeping tetikleyicisinin eksik olduğu saptandı. Tabloya 1Hz tetikleme kodu girilip sistem yeniden derlendiğinde veri akışı başladı.

### 🔴 Hata 2: Nişangahların Ekran Dışına Taşması (Dizilim Uyuşmazlığı)
*   **Belirti:** Dashboard üzerinde yıldızların etrafına çizilmesi gereken hedef yuvarlakları ekranın dışında (Y=710px gibi) anlamsız konumlarda çıkıyordu.
*   **Çözüm:** Uydunun hafızaya ardışık (Sequential) yazma düzeni ile Yer İstasyonunun çaprazlama (Interleaved) okuma düzeninin çakıştığı tespit edildi. Dashboard veri çözücü kodu bayt düzeyinde yeniden düzenlenerek nişangahların tam yıldız merkezlerine kilitlenmesi sağlandı.

---

## 🚀 5. Sistemi Sıfırdan Açma Protokolü

Uygulamayı sorunsuz bir şekilde başlatmak için sırasıyla şu adımlar izlenmelidir:

1.  **Uydunun Beyni (WSL Terminal 1):** `cd ~/cFS/build/exe/cpu1` dizinine gidip `./core-cpu1` çalıştırılır.
2.  **Yer İstasyonu (WSL Terminal 2):** `cd ~/cFS/tools/cFS-GroundSystem` dizinine gidip `python3 GroundSystem.py` çalıştırılır.
3.  **Görsel Takip Ekranı (Windows):** Masaüstündeki **`UYDU_DASHBOARD.bat`** dosyasına çift tıklanır.
4.  **Telemetri Bağlantısı:** Yer istasyonundan `TO LAB CPU1` seçilir, hedef IP `127.0.0.1` yazılarak `TO_LAB_OUTPUT_ENABLE_CC` komutu **Send** edilir.
5.  **Sahne Analizi:** Dashboard'dan sahne seçilir ve yer istasyonundaki `SAMPLE APP` panelinden `SAMPLE_APP_FIND_STARS_CC` komutu **Send** edilerek işlem tamamlanır.

---
### Uçuş Durumu: TÜM SİSTEMLER OPERASYONEL VE NOMİNAL. 🚀📡🛰️
