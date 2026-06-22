# NASA cFS Otonom Star Tracker Entegrasyonu ve Operasyonel Arayüz Kontrol Kılavuzu

## Özet (Abstract)
Bu proje; NASA core Flight System (cFS) mimarisi üzerinde çalışan bir Otonom Star Tracker (Yıldız Takipçisi) payload'unun entegrasyonunu, bilimsel görüntü işleme katmanlarını ve bu sistemi kontrol etmek için geliştirilen çok katmanlı görsel arayüzleri kapsamaktadır. 

Bu kılavuz, hem sistemin çalıştırılması esnasında kullanılan fiziksel buton ve terminallerin görevlerini açıklayan bir **Kullanıcı Kılavuzu**, hem de bugün sıfırdan geliştirip entegre ettiğimiz **Gelişmiş Mühendislik ve Görüntü İşleme Çözümlerini** içeren kapsamlı bir teknik referans belgesidir.

---

## 🌟 1. Geliştirilen Uygulama Yetenekleri ve Mühendislik Başarıları

Bugün gerçekleştirdiğimiz çalışmalarla uydunun beynini ve takip arayüzünü sadece veri okuyan değil, gelişmiş görüntü işleme ve CAD tipi analizler yapabilen bilimsel bir merkeze dönüştürdük:

### 🌌 A. Yüksek Çözünürlüklü Çift Uzay Sahnesi ve Dinamik Tampon Bellek (Dual-Scene Buffer)
*   Sistemde tek bir referans fotoğrafla sınırlı kalınmamış; NASA Hubble/JWST gözlemlerinden ilham alan, yüzlerce keskin yıldız ve kozmik bulutsular barındıran **2. bir özel test görseli (Dense Star Field)** tasarlanarak sisteme entegre edilmiştir.
*   Uydunun kamerasının fiziksel olarak yön değiştirmesini simüle etmek amacıyla, Dashboard arayüzünden seçilen görüntü anlık olarak `float32` ikili (binary) formatına serileştirilmekte ve uydunun kamera bellek dosyası olan **`test_yildiz.bin`** üzerine yazılarak eş zamanlı güncellenmektedir.

### 🎨 B. Bilimsel Termal Görüntüleme Motoru (Infrared Jet Colormap)
*   Optik izlemede kızılötesi dalga boyunu simüle etmek için **Indexed8** tabanlı hızlı bir termal renklendirme motoru yazılmıştır.
*   Geliştirilen **Jet Colormap** algoritması sayesinde; soğuk derin uzay bölgeleri koyu lacivert tonlarda ezilmeden gösterilirken, sıcak yıldız çekirdekleri ve nebulalar parlak sarı-kırmızı renk spektrumuna hassas bir şekilde dağıtılır.

### 🔍 C. CAD Tipi İnteraktif Gözlem ve Zoom Altyapısı
*   Görsel takip ekranı, standart sabit pencerelerin ötesine geçerek tam bir CAD yazılımı gibi çalışacak şekilde tasarlanmıştır:
    *   **İmleç Odaklı Zoom:** Fare tekerleği döndürüldüğünde görüntüyü doğrudan fare imlecinin bulunduğu koordinat merkezinde büyütüp küçültebilir.
    *   **Sürükle-Bırak (Pan):** Fare sol tuşu basılı tutulup sürüklenerek, derin uzay fotoğrafının içinde tıpkı bir uzay teleskobu operatörü gibi gezinti yapılabilir.

### 📐 D. Takımyıldız Ağı ve Uzamsal Nirengi (Constellation Tracking Net)
*   Uyduların uzayda yön belirlemek (Attitude Determination) için kullandığı yıldız eşleştirme matematiğini simüle eden bir **Vektör Bağlantı Ağı** yazılmıştır.
*   Sistem, uydudan gelen 5 ana yıldızın arasına neon mavi kesikli lazer çizgileri çeker.
*   Her çizgideki Öklidyen mesafeyi ($d = \sqrt{\Delta x^2 + \Delta y^2}$) pikselsel olarak anlık hesaplar ve çizginin tam ortasına bir etiketle (Örn: `245.8px`) basar.

### 🛸 E. 30 FPS Aktif Konik Radar Arama Modülü
*   Görüntünün durağanlığını kırmak ve sistemin "aktif tarama" anını sembolize etmek amacıyla, arayüze saniyede 30 kare hızla dönen konik bir radar animasyon katmanı eklenmiştir.
*   Bu modül; uydunun ilk açılışta yaptığı **Lost-in-Space** yönelim aramasını ve bilimsel CMOS sensörlerin aktif piksel okuma (Readout) sürecini görselleştirir.

### 🗺️ F. Gerçek Zamanlı Yörünge ve Dünya Takip Paneli (LEO Orbital Tracker)
*   Sisteme LEO (Alçak Dünya Yörüngesi) yörünge mekaniğini simüle eden matematiksel bir izleyici eklenmiştir.
*   Arayüzün altında yer alan fütüristik bir dünya haritası üzerinde uydunun (SAT-1) yörünge izi (ground track) neon kesikli çizgilerle çizilir ve uydunun konumu anlık güncellenir.
*   Enlem (Latitude), Boylam (Longitude), İrtifa (Altitude), Yörünge Hızı (Velocity) ve tamamlanan tur sayısı (Orbit Revs) canlı olarak hesaplanarak yan panelde gösterilir.

### 🔴 G. Gerçek Zamanlı Canlı ISS ve Dünya İzleme Sekmesi (Live Space Tracker)
*   Arayüze yerleştirilen `QTabWidget` sayesinde, cFS simülasyonunun yanı sıra gerçek dünya uzay operasyonlarını izleme yeteneği kazandırılmıştır.
*   **Çoklu Uydu Canlı Takibi (Multi-Satellite Tracking):** Harita üzerinde 4 farklı uzay aracının/uydunun gerçek zamanlı konumları ve yörünge izleri (ground track) farklı neon renklerle çizilir:
    *   **Uluslararası Uzay İstasyonu (ISS):** `open-notify` API'sinden çekilen anlık koordinatlarla takip edilir. (Renk: **Neon Turuncu**)
    *   **Hubble Uzay Teleskobu (HST):** Gerçek yörünge eğikliği ($28.5^\circ$) ve periyoduyla simüle edilir. (Renk: **Neon Camgöbeği**)
    *   **Starlink-1017 Uydusu:** Küresel internet takım uydusu yörüngesi ($53^\circ$ eğiklik) simüle edilir. (Renk: **Neon Sarı**)
    *   **NOAA-19 Meteoroloji Uydusu:** Polar (kutsal) yörünge ($99^\circ$ eğiklik) takip edilir. (Renk: **Neon Macenta**)
*   **Güneş Havası Durumu (Solar Weather):** NOAA uzay hava tahmin merkezinden alınan güneş rüzgarları hızı, Kp manyetik fırtına indeksi ve güneş patlaması risk seviyeleri canlı olarak gösterilir.
*   **Uzaydaki İnsan Sayısı (Astronauts in Space):** Şu an uzayda aktif görev yapan astronot sayısı (`TOTAL CREW`) ve isimleri `astros.json` API'sinden çekilerek dinamik scroll panelinde listelenir.
*   **Canlı Dünya Görüntüsü (DSCOVR/EPIC):** NASA'nın Lagrange-1 noktasındaki **DSCOVR** uydusunda yer alan **EPIC (Earth Polychromatic Imaging Camera)** kamerasından çekilmiş en güncel gerçek Dünya fotoğrafları arka planda indirilerek canlı yayın modülüyle ekrana basılır.

### 🛡️ H. Otonom Hata Tespit, Yalıtım ve Kurtarma Sistemi (FDIR)
Uydunun uzay radyasyonu ve donanımsal arızalara karşı güvenliğini otonom olarak yöneten **FDIR (Fault Detection, Isolation, and Recovery)** modülü sol kontrol paneline entegre edilmiştir.

*   **Hata Enjeksiyon Paneli (Fault Injector):** Mühendislik testleri için 3 farklı arıza elle tetiklenebilir:
    1.  **TEMP RADIATION SEU:** Sıcaklık sensörünün radyasyon sebebiyle $+185.30^\circ C$ gibi hatalı bir değere kilitlenmesini sağlar.
    2.  **CAM MEMORY CORRUPTION:** Yıldız kamerasındaki piksel okuma hatasını simüle eder, algılanan yıldız sayısını 0'a düşürür ve kamera görüntüsünü karartır.
    3.  **TELEMETRY LINK GLITCH:** Yer istasyonu bağlantısının koptuğunu simüle ederek veri akışını dondurur.
*   **Otonom Watchdog Algoritması:** 
    *   Sistem verilerini sürekli izler. Limit dışı veya anomali tespit ettiği an (Örn: $>80^\circ C$ sıcaklık veya sıfır yıldız) uydunun durumunu otomatik olarak **`🔴 SAFE MODE`** (Güvenli Mod) durumuna alır.
*   **Çok Aşamalı Kurtarma Senaryosu (Multi-Stage Recovery Escalation):**
    1.  **Aşama 1 (Yazılımsal Reset):** Arıza algılandıktan 3 saniye sonra, FDIR modülü arızalı alt sisteme otonom bir yazılımsal reset komutu gönderir (`RESET CMD`).
    2.  **Aşama 2 (Yedek Donanıma Geçiş - Hardware Redundancy):** Eğer reset işlemi arızayı çözmezse, FDIR hatayı izole eder ve uydunun yedek kanalı olan **`CH-B` (Yedek Kanal B)** sensör/verici sistemine otomatik geçiş yapar.
    3.  **Kurtarma Başarılı (Recovery Success):** Yedek kanala geçildikten sonra veriler tekrar nominal değerlerine döner, alarm temizlenir ve uydu görevine otonom olarak geri döner.

---

## 🖥️ 2. Sistem Çalışma Düzeni ve Terminal Matrisi (Ubuntu/WSL Terminalleri)

Arka planda çalışan Ubuntu terminalleri, uydunun ve yer sisteminin fiziksel birer katmanını temsil eder:

| Terminal Adı | Çalıştırılan Komut / Görev | Operasyonel Karşılığı |
| :--- | :--- | :--- |
| **Terminal 1: Uydunun Beyni** | `./core-cpu1` | Gerçek uydunun üzerindeki ana uçuş bilgisayarını (OBC) temsil eder. C kodlu yıldız algılama algoritmaları burada koşar. |
| **Terminal 2: Yer İstasyonu** | `python3 GroundSystem.py` | NASA'nın yer kontrol merkezindeki ana veri dağıtım sunucusunu (Router) başlatır. |
| **Terminal 3: Görsel Monitör** | `start-tracker` veya `UYDU_DASHBOARD.bat` | Uydudan gelen verileri işleyen ve yukarıda açıklanan 5 katmanlı görselleştirmeyi yapan monitörümüzdür. |

---

## 📡 3. NASA Yer İstasyonu Panelleri ve Kritik Butonlar

NASA GroundSystem arayüzündeki butonların operasyonel dünyadaki karşılıkları şunlardır:

### 🟦 A. TO LAB (Telemetry Output Lab) Paneli
*   **Kritik Buton:** `TO_LAB_OUTPUT_ENABLE_CC`
*   **Fiziksel Görevi:** Uydunun yer istasyonuna veri gönderen verici antenini (Telemetry Transmitter) açar. Bu düğmeye basılmadan hiçbir telemetri verisi alınamaz.

### 🟩 B. SAMPLE APP (CPU1) Paneli
*   **Kritik Buton:** `SAMPLE_APP_FIND_STARS_CC`
*   **Fiziksel Görevi:** Uydunun bilimsel kamerasının deklanşörünü tetikler. Kamera o an hangi sahneye bakıyorsa fotoğrafı çeker, C motoru bu görseli tarayarak yıldızları bulur ve koordinatları dünyaya gönderir.

---

## 🪐 4. Görsel Operasyon Paneli (V.O.D. Dashboard) Kontrolleri

Dashboard arayüzünde yer alan ve uygulamaya eklediğimiz o efsanevi özellikleri yöneten kontroller:

*   **🎯 TARGET ACTIVE SCENE:** Uydunun kamerasını fiziksel olarak farklı bir gök bölgesine (Scene 1 / Scene 2) çevirmesini simüle eder ve tampon belleği ezer.
*   **🔳 STANDARD VIEW:** CMOS kameradan gelen ham, siyah-beyaz astronomik görüntüyü gösterir.
*   **🔳 THERMAL CAMERA:** Jet Colormap renk algoritmasını kullanarak kızılötesi görüntülemeyi aktif eder.
*   **🔳 FIT IMAGE TO SCREEN:** Görüntüyü pencereye sığdırır, interaktif zoom modunun başlangıç ayarıdır.
*   **📐 BUILD CONSTELLATION NET:** Yıldızları lazer çizgileriyle bağlayıp navigasyonel mesafe nirengi hesaplamasını başlatır.
*   **🛸 RADAR BEAM ON / OFF BUTONLARI:** Saniyede 30 tur dönen konik sensör okuma radarını başlatmak veya tamamen gizlemek için kullanılan özel kontrol butonlarıdır.
*   **🗺️ LEO ORBITAL TRACKER PANELİ:** Uydunun yörünge hareketini dünya haritası üzerinde canlı olarak izleyen ve konum telemetrisini gösteren izleme panelidir.

---

## 🛠️ 5. Giderilen Kritik Hatalar ve Hata Günlüğü (Bug Logging)

### 🔴 Hata 1: Sıfır Veri Akışı Sorunu (Scheduler Tablo Eksikliği)
*   **Analiz:** Sistemler açık olmasına rağmen veri akmıyordu.
*   **Çözüm:** cFS çekirdek tablosundaki (`sch_lab_table.c`) Housekeeping tetikleyicisinin eksik olduğu görüldü. 1Hz tetikleme kodu girilip sistem derlendiğinde sorun çözüldü.

### 🔴 Hata 2: Nişangahların Ekran Dışına Taşması (Sequential-Interleaved Uyuşmazlığı)
*   **Analiz:** Y koordinatları `710px` gibi limit dışı çıkıyordu ve nişangahlar yıldızları ıskalıyordu.
*   **Çözüm:** Uydunun C kodundaki ardışık (Sequential) veri paketleme sırası ile Yer İstasyonunun beklediği çaprazlama (Interleaved) okuma sırasının çakıştığı keşfedildi. Dashboard veri çözücüsü bayt bazında ardışık düzene göre baştan yazılarak %100 doğruluk sağlandı.

---

## 🚀 6. Sistemi Devreye Alma Protokolü

1.  **Uydunun Beyni:** `cd ~/cFS/build/exe/cpu1` dizininden `./core-cpu1` başlatılır.
2.  **Yer İstasyonu:** `cd ~/cFS/tools/cFS-GroundSystem` dizininden `python3 GroundSystem.py` başlatılır.
3.  **Görsel Takip Ekranı:** Masaüstündeki **`UYDU_DASHBOARD.bat`** dosyasına çift tıklanır.
4.  **Telemetri Bağlantısı:** Yer istasyonundan `TO LAB CPU1` seçilir, IP `127.0.0.1` girilir ve `TO_LAB_OUTPUT_ENABLE_CC` komutu **Send** edilir.
5.  **Sahne Analizi:** Dashboard'dan sahne belirlenir ve `SAMPLE APP` üzerinden `SAMPLE_APP_FIND_STARS_CC` **Send** edilir.

---
### Uçuş Durumu: TÜM SİSTEMLER VE GÖRÜNTÜ İŞLEME KATMANLARI NOMİNAL. 🚀📡🌌🛰️
