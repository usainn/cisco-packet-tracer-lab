# Cisco Packet Tracer: Basit Ağ Kurulumu ve Sorun Giderme (DHCP & DNS)

Bu projede, Cisco Packet Tracer kullanarak temel bir yerel ağ (LAN) oluşturulmuş, uç cihazların yapılandırması sağlanmış ve ağ bağlantısı sırasındaki DHCP/DNS yapılandırma sorunları çözümlenmiştir.

##  Projenin Amacı
- Uç cihazları (PC, Laptop) ağ geçidine (Wireless Router) bağlamak.
- Ağ geçidini bir kablo modem üzerinden dış ağa (Internet Cloud) çıkarmak.
- Cihazların DHCP üzerinden otomatik IP ve DNS almasını sağlamak.
- Adres çözümleme (Host Name Unresolved) hatalarını teşhis edip gidermek.

##  Öğrenilen Temel Ağ Kavramları (Key Takeaways)

Bu laboratuvar sürecinde ağ mimarisinin temellerine dair şu kritik kavramlar pekiştirilmiştir:

### 1. Default Gateway (Varsayılan Ağ Geçidi) Mantığı
Yerel ağımızdaki (LAN) cihazların dış dünyaya (internete) çıkabilmesi için bir çıkış kapısına ihtiyacı vardır. Tıpkı bir sokaktaki evlerin, sokağın sonundaki kavşaktan ana caddeye çıkması gibi. Bu topolojide **Wireless Router**, yerel ağdaki (`192.168.0.x`) trafiği toplayıp ISP'ye (İnternet Servis Sağlayıcısına) yönlendiren **Ağ Geçidi (Gateway)** görevini üstlenmektedir. PC ve Laptop'un dışarı çıkmak için başvurduğu adres `192.168.0.1`'dir.

### 2. Bu Laboratuvarda Neden Bu Kabloları Tercih Ettik?
Ağ kurulumunda cihazları birbirine bağlarken Packet Tracer'ın rastgele kablo seçimine izin vermemesinin elektriksel ve donanımsal sebepleri vardır. Bu projede kablo seçimlerimizi şu mantıkla yaptık:

* **Copper Straight-Through (Düz Bakır Kablo):** * Laboratuvarımızda PC'mizin `FastEthernet0` portunu, Wireless Router'ın yerel ağ (`Ethernet 1`) portuna bağlarken bu kabloyu kullandık. 
  * Aynı şekilde, Router'ın dış ağa açılan `Internet` portunu da Cable Modem'in `Port 1` girişine yine bu kabloyla bağladık.
  * **Tercih Sebebimiz:** Çünkü PC (uç cihaz), Router (yönlendirici) ve Modem farklı türde cihazlardır. Packet Tracer'da bu farklı cihazların donanımsal "veri gönderme" (TX) ve "veri alma" (RX) pinleri zıt konumlardadır. Düz kablo, bu pinleri çakışma yaratmadan doğru şekilde eşleştirerek cihazların birbirini duymasını sağladı.

* **Coaxial (Koaksiyel Kablo):** * Uygulamamızda Cable Modem'in `Port 0` çıkışından, dış dünyayı temsil eden Internet Cloud'un `Coaxial 7` girişine giderken koaksiyel kabloyu seçmek zorundaydık.
  * **Tercih Sebebimiz:** Çünkü kurduğumuz yerel ağ (LAN) modemde biter ve servis sağlayıcının dünyası (WAN) başlar. Packet Tracer'daki kablo modem cihazı, Router'dan gelen standart dijital (ethernet) sinyallerini, internet servis sağlayıcının (ISP) bulut altyapısına uygun radyo frekansı sinyallerine dönüştürür. Dış dünya ile aramızdaki bu frekans tabanlı geniş bant iletişimini simüle etmek için fiziksel olarak koaksiyel bağlantı kullanmamız gerekti.

### 3. Subnet Mask (Alt Ağ Maskesi) ve IP Dağılımı
Ağdaki `255.255.255.0` alt ağ maskesi, IP adresinin `192.168.0` kısmının **Ağ Kimliği** (sokak adı), sondaki sayının ise **Cihaz Kimliği** (ev numarası) olduğunu belirler. Bu sayede yönlendirici, DHCP aracılığıyla her cihaza aynı sokakta benzersiz bir numara atar.

### 4. Donanım Yükseltmesi ve WPC300N Modülü (Fiziksel Katman)
Uç cihazların kablosuz ağlara katılabilmesi için uygun donanıma (Wireless NIC) sahip olması gerekir. Laboratuvardaki standart dizüstü bilgisayarda yalnızca Ethernet arayüzü bulunduğu için, cihaz fiziksel olarak kapatılmış ve Ethernet modülü yerine radyo frekansı (RF) ile haberleşebilen **WPC300N** kablosuz ağ modülü entegre edilmiştir. Bu adım, gömülü sistem projelerinde yerleşik haberleşme yeteneği olmayan standart mikrodenetleyicilere harici Wi-Fi donanımları ekleme mantığıyla birebir aynıdır.

### 5. Web Tarayıcısı Testi ve DNS Çözümleme Mantığı (Uygulama Katmanı)
Ağ bağlantısının başarılı olduğu sadece IP üzerinden "ping" atılarak değil, doğrudan web tarayıcısına `cisco.srv` yazılarak test edilmiştir. Bunun temel nedeni **DNS (Alan Adı Sistemi)** mekanizmasının doğru çalıştığını kanıtlamaktır.
DNS, en basit tabirle ağın **"Telefon Rehberi"**dir. Bilgisayarlar kelimelerden anlamaz, sadece IP adresleriyle (`192.168.0.5` gibi) haberleşir. Tarayıcıya `cisco.srv` yazıldığında sistem arka planda DNS sunucusuna gidip "Bu ismin IP numarası nedir?" diye sorar ve aldığı cevapla bağlantıyı kurar. Sadece ping atmak "kablolarımız sağlam" anlamına gelirken, isme istek atmak "cihazımız ağın rehberini (DNS) başarıyla kullanabiliyor" anlamına gelir. 

##  Ağ Topolojisi
Kurduğumuz ağın son hali şu şekildedir:

![Ağ Topolojisi](ag_topoloji.png)
*(Topolojinin eksiksiz ve yeşil bağlantılı ekran görüntüsü)*

##  Adım Adım Kurulum

### Adım 1: Cihazların Çalışma Alanına Eklenmesi
* **PC** ve **Laptop** (End Devices)
* **Wireless Router** (Network Devices > Wireless Devices)
* **Cable Modem** (Network Devices > WAN Emulation)
* **Internet Cloud** (WAN Emulation)

### Adım 2: Fiziksel Kablolama
1.  **PC -> Wireless Router:** Düz bakır kablo ile PC'nin *FastEthernet0* portu, Router'ın *GigabitEthernet 1* (LAN) portuna bağlandı.
2.  **Wireless Router -> Cable Modem:** Düz bakır kablo ile Router'ın *Internet* (WAN) portu, Modemin *Port 1* girişine bağlandı.
3.  **Cable Modem -> Internet Cloud:** Koaksiyel kablo ile Modemin *Port 0* girişi, Bulutun *Coaxial 7* girişine bağlandı.

### Adım 3: DHCP Yapılandırması ve Doğrulama
Bağlantılar tamamlandıktan sonra cihazların IP ayarları yapılandırıldı. Komut satırından `ipconfig /all` komutu ve web tarayıcısı üzerinden bağlantılar test edildi.

![IP Config Çıktısı](ip_config.png)

##  Karşılaşılan Sorunlar ve Çözümleri

**Sorun 1: Web Tarayıcısında "Host Name Unresolved" Hatası**
* **Teşhis:** Laptop Wi-Fi ağına katılmış ancak DHCP'den bir DNS adresi alamamıştı (`0.0.0.0`).
* **Çözüm:** Router ile Modem arasındaki kablonun yanlışlıkla yerel ağ (LAN) portuna takıldığı fark edildi. Bağlantı, dış ağ çıkışı olan *Internet (WAN)* portuna taşındıktan sonra cihaz başarıyla DNS alabildi ve URL çözümlendi.

**Sorun 2: Doğru IP Yapılandırmasına Rağmen Activity Motorunun Hata Vermesi**
* **Teşhis:** Cihazlar doğru IP'leri (`192.168.0.x`) almasına rağmen Packet Tracer aktivite motoru durumu "Incorrect" olarak işaretliyordu.
* **Çözüm:** Simülasyonun cihazları varsayılan isimleriyle (`PC0`, `Cable Modem0`) değil, tam eşleşen isimleriyle okuduğu tespit edildi. Cihaz isimlerindeki fazla karakterler temizlenip IP'ler DHCP üzerinden yenilendiğinde sistem %100 başarıyla tamamlandı.

###  Mühendislik Yaklaşımı ve Sistematik Sorun Giderme (Troubleshooting)
Karşılaşılan hataların çözümünde rastgele ayar değiştirmek yerine **OSI Modeli** temel alınarak aşağıdan yukarıya (Bottom-Up) bir hata ayıklama yöntemi izlenmiştir:
1. **Fiziksel Katman (Layer 1) Kontrolü:** "Kablolar doğru portlara takılı mı?" sorusu sorularak LAN/WAN port karmaşası fiziksel olarak çözüldü.
2. **Ağ Katmanı (Layer 3) Kontrolü:** Cihazların otomatik IP atamak yerine (APIPA), doğrudan Gateway üzerinden `192.168.0.x` bloğundan IP ve Alt Ağ Maskesi aldığı teyit edildi.
3. **Uygulama Katmanı (Layer 7) Kontrolü:** DNS sunucusunun IP'leri çözümleyip çözümleyemediği kontrol edilerek `cisco.srv` adresine sorunsuz erişim sağlandı.

###  Gerçek Dünya Uygulamaları (IoT ve Veri Panelleri)
Burada kurulan mimari, sadece sanal bilgisayarların değil, günümüzdeki endüstriyel ağların ve veri tabanlı projelerin temelini oluşturur. Sahada çevresel verileri toplayan bir donanımın elde ettiği canlı verileri modern web tabanlı bir panele (dashboard) iletebilmesi için arka planda tam olarak bu topolojideki Ağ Geçidi yönlendirmelerine ve API uç noktalarını çözümleyen DNS mekanizmalarına ihtiyaç vardır. Bu laboratuvar, bu tür sistemlerin ağ altyapısını kavramak için güçlü bir temel sağlamıştır.
