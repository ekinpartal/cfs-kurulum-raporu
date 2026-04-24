# cFS (core Flight System) İnceleme ve Yerel Kurulum Raporu

**Kapsam:** Issue #4
**Amaç:** NASA'nın core Flight System (cFS) mimarisinin incelenmesi, FPGA tabanlı donanımlarla olan yazılımsal ilişkisinin analiz edilmesi ve cFS altyapısının yerel geliştirme ortamına (WSL) adım adım kurularak çalıştırılması.

## 1. cFS (core Flight System) Mimari Analizi

cFS, NASA tarafından geliştirilmiş, Olay Güdümlü (Event-Driven) ve Yayınla/Abone Ol (Publish/Subscribe) mantığına dayanan bağımsız bir uçuş yazılımı (Flight Software) mimarisidir. Üç temel katmandan oluşur:
- **OSAL (Operating System Abstraction Layer):** Uçuş kodunun, çalıştığı gerçek zamanlı veya standart işletim sisteminden (FreeRTOS, Linux vb.) bağımsız olarak derlenebilmesini sağlayan soyutlama katmanıdır.
- **cFE (core Flight Executive):** Sistemin çekirdek servislerini sağlayan ana yöneticidir.
- **cFS Apps:** Çekirdek üzerine yüklenen spesifik uzay aracı veya modül uygulamalarıdır.

### 1.1 Çekirdek Servisler (cFE Services)
- **SB (Software Bus - Yazılım Veriyolu):** Uygulamalar arası iletişimi sağlayan mesajlaşma altyapısıdır. Global değişkenler yerine Publish/Subscribe mantığı kullanılır. Bir uygulama veri yayınlar, diğerleri bu veriye abone olur.
- **TBL (Table Services - Tablo Servisleri):** Sistemi veya ilgili modülü yeniden başlatmaya (reboot) gerek kalmadan, çalışma zamanında (runtime) sistem parametrelerinin dışarıdan yüklenen tablolarla güncellenmesini sağlayan hafıza yönetim servisidir.
- **EVS (Event Services - Olay Servisleri):** Uzay sistemlerinde fiziksel bir terminal (stdout) bulunmadığı için, hataların ve uyarıların filtrelenerek `CFE_EVS_SendEvent` fonksiyonu aracılığıyla yapılandırılmış olay mesajlarına dönüştürülüp Yer İstasyonuna gönderilmesini sağlar.

### 1.2 Haberleşme Terminolojisi
- **TLM (Telemetry):** Uzay aracından (Uygulamadan) Yer İstasyonuna gönderilen izleme paketleridir.
- **CMD (Command):** Yer İstasyonundan uzay aracına gelen işlevsel emirlerdir.
- **HK (Housekeeping):** Her modülün düzenli frekanslarda (örn. 1 Hz) durumunu, hata sayaçlarını ve sağlığını Yer İstasyonuna raporladığı telemetri paketleridir.

## 2. FPGA ve cFS İlişkisi

Geliştirme yapılan System-on-Chip (SoC) mimarisinde, iş yükü donanım (FPGA) ve yazılım (MSS - İşlemci) arasında bölünmüştür.
- **Donanım Boyutu (FPGA):** Kamera sensöründen yüksek bant genişliğiyle akan görüntü verisini paralel olarak ve milisaniye altında ön işlemeye tabi tutar (örn. sinyal filtreleme veya koordinat hesaplama). Elde edilen ham matematiksel veriyi, yüksek hızlı donanım veriyolları (AXI4) aracılığıyla işlemcinin belleğine aktarır.
- **Yazılım Boyutu (cFS):** SoC'nin işlemci (MSS) çekirdekleri üzerinde çalışan cFS işletim sistemi, FPGA tarafından belleğe bırakılan bu ham veriyi alır. cFS uygulamaları (Apps), bu veri üzerinde karmaşık algoritmik operasyonları gerçekleştirir, alt sistemleri kontrol eder ve nihai sonuçları radyo bağlantısı üzerinden telemetri (TLM) olarak dünyaya ulaştırır.

## 3. Kurulum Aşamasında Karşılaşılan Zorluklar ve Çözümler

Kurulum sürecinde bazı teknik engellerle karşılaşılmış ve aşağıdaki çözüm yaklaşımları uygulanmıştır:
1. **İşletim Sistemi Uyumluluğu:** cFS projesinin C derleyicileri (gcc, make) Windows üzerinde performans ve erişim sorunları yaratmaktadır. Çözüm olarak yerel sisteme **WSL (Windows Subsystem for Linux)** aktif edilerek saf bir Ubuntu ortamı oluşturulmuştur.
2. **Terminal Üzerinden GitHub Kimlik Doğrulaması:** GitHub'ın terminalden klasik şifre ile doğrulamayı kaldırmasından dolayı yetkilendirme hatası (Authentication Failed) alınmıştır. Çözüm olarak **GitHub CLI (`gh`)** aracı kurularak OAuth tabanlı web doğrulaması (Device Login) uygulanmış ve güvenli giriş sağlanmıştır.
3. **Derleyici ve Kütüphane Eksiklikleri:** Temel WSL Ubuntu kurulumunda derleme araçlarının bulunmaması sebebiyle derleme aşamasında komut bulunamadı hataları alınmıştır. İlgili temel C geliştirme paketleri ve kütüphaneler terminal üzerinden manuel olarak entegre edilmiştir.

## 4. Tüm Terminal Komutları ve Adım Adım Kurulum

Başka bir geliştiricinin sıfırdan sistemi kurabilmesi için Ubuntu terminalinde izlenen komutların tamamı atlanmadan aşağıya çıkarılmıştır:

**Adım 1: Bağımlılıkların ve Araçların Kurulması**
Ubuntu terminali ilk açıldığında uzay projesi için gereken C derleyicileri ve paketler kurulmalıdır:
```bash
sudo apt-get update
sudo apt-get install -y build-essential gcc make cmake libcfitsio-dev git gh
```

**Adım 2: GitHub Kimlik Doğrulaması**
Özel depolara (Private Repositories) erişmek için GitHub hesabına giriş yapılır:
```bash
gh auth login
```
*(Gelen terminal sorularında ok tuşlarıyla sırasıyla: `GitHub.com` -> `HTTPS` -> `Yes` -> `Login with a web browser` seçilir. Ekranda verilen 8 haneli kod bilgisayar tarayıcısından girilerek yetkilendirme tamamlanır).*

**Adım 3: Proje Deposunun Çekilmesi ve Test Edilmesi**
Projenin ana deposu indirilir ve çalıştırılması denenir:
```bash
gh repo clone Meto423/startracker-c-pipeline
cd startracker-c-pipeline/csrc
make all
```
*Not: Bu aşamada `make: *** No rule to make target 'all'` hatası alınmıştır. Bunun sebebi, depo sahibinin o an itibariyle ilgili klasörün içine `Makefile` dosyasını ve C kaynak kodlarını henüz yüklememiş (repo içi boş bırakılmış) olmasıdır. Bu normal bir durumdur, kod eklendiğinde derleme çalışacaktır.*

**Adım 4: NASA cFS Deposunun Çekilmesi**
Simülasyonu ayağa kaldırmak için doğrudan NASA'nın ana kodları çekilir:
```bash
cd ~
git clone --recurse-submodules https://github.com/nasa/cFS.git
```
*(Not: Eğer bir hata yapılıp daha önceden cFS adında boş bir klasör oluşturulduysa, indirme öncesinde `rm -rf ~/cFS` komutuyla silinmelidir).*

**Adım 5: cFS Konfigürasyon ve Derleme (Build) Aşaması**
Örnek derleme ayarları kopyalanıp NASA kodları sistemimizde derlenir:
```bash
cd ~/cFS
cp cfe/cmake/Makefile.sample Makefile
cp -r cfe/cmake/sample_defs sample_defs
make SIMULATION=native prep
make
make install
```

**Adım 6: Uzay Simülasyonunun (cFS) Başlatılması**
Derlenen çalıştırılabilir işletim sistemi ayağa kaldırılır:
```bash
cd build/exe/cpu1
./core-cpu1
```
*(Bu komutla birlikte sistem uyarı mesajları vererek uyanır ve en son "OPERATIONAL state" konumuna geçerek terminalde öylece bekler. Sistemi kapatmak için `Ctrl+C` kullanılır).*

**Adım 7: Yer İstasyonu (Ground Station) Arayüzünün Kurulması**
Yukarıdaki terminal arka planda çalışıp simülasyonu sürdürürken, **yepyeni ikinci bir Ubuntu terminal penceresi** açılır ve arayüz kütüphaneleri indirilir:
```bash
sudo apt-get install -y python3-pyqt5 python3-zmq
```

**Adım 8: Yer İstasyonu Arayüzünün Başlatılması**
İkinci terminalden görsel kontrol paneli (GUI) çalıştırılır:
```bash
cd ~/cFS/tools/cFS-GroundSystem
python3 GroundSystem.py
```

**Adım 9: Hızlı Başlatma İçin Kısayol (Alias) Oluşturma**
Kurulum sonrası sistemi her seferinde uzun dizin yolları yazarak başlatmamak adına Ubuntu `~/.bashrc` dosyasına kalıcı kısayollar eklenmiştir:
```bash
echo 'alias uydu="cd ~/cFS/build/exe/cpu1 && ./core-cpu1"' >> ~/.bashrc
echo 'alias istasyon="cd ~/cFS/tools/cFS-GroundSystem && python3 GroundSystem.py"' >> ~/.bashrc
source ~/.bashrc
```
*Bu sayede sistemi tekrar çalıştırmak için herhangi bir indirme veya derleme işlemi gerekmez. Terminale sadece `uydu` yazılarak arka planda cFS simülasyonu başlatılır, ikinci terminale `istasyon` yazılarak doğrudan yer kontrol paneli açılır.*

## 5. Sonuçlar ve Veri Doğrulama

Sistemin başlatılmasının ardından, terminal üzerinde cFE Core bileşenlerinin başarılı bir şekilde uyanış mesajları (boot sequence) ve işletim sistemi kayıtları gözlemlenmiştir. 
- OSAL ve EEPROM simülasyonları başarıyla konfigüre edilmiştir.
- Çekirdek servisler (`EVS`, `SB`, `TBL`, `TIME`) Initialize durumuna geçmiştir.
- Temel laboratuvar uygulamaları (`SAMPLE_APP`, `CI_LAB`, `TO_LAB`) Yazılım Veriyoluna (SB) başarıyla kaydedilmiştir.
- Arayüz (`GroundSystem.py`) çalıştırıldığında, **Command System** sekmesinden `Enable Tlm` komutu yollanmış ve **Telemetry System** ekranından `Packet Count` (Paket Sayaçları) değerlerinin artmaya başladığı doğrulanmıştır. `ES HK Tlm` (Housekeeping) verileri görsel arayüz üzerinden başarılı bir şekilde anlık olarak izlenmiştir.
