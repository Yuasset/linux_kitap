# ifconfig

Bugün GNU/Linux kullanan sistem yöneticilerinin çoğu eski alışkanlıklarından dolayı ağ cihazlarını \(NIC\) ayarlamak için `ifconfig` komutunu kullanırlar, ancak ifconfig uzun süredir geliştirilmeyen ve artık terk edilen \(deprecated\) bir yazılımdır. Dolayısıyla kullanmamak daha doğrudur.

Programın geliştirilmesinin durdurulduğu ilk olarak Debian mail listinde paylaşıldı. Bugün CentOS 7 gibi sistemlerde bu komut bulunmamaktadır.

Duyuruyu şuradan görebilirsiniz: [https://lists.debian.org/debian-devel/2009/03/msg00780.html](https://lists.debian.org/debian-devel/2009/03/msg00780.html)

ifconfig'in artık geliştirilmemesinin pek çok sebebi vardır, ve bunun yerine `ip` komutunun kullanımı tavsiye edilir. Örneğin netmask tanımlanmasına CIDR notasyonu desteklenmez, `255.255.255.248` yerine `/29` yazamazsınız.

Öte yandan, bazı eski sistemlerde veya gömülü sistemlerde `ip` komutu bulunmayabilir. Bunun için nasıl çalıştığı hakkında fikir sahibi olmakta fayda var.

## Mevcut Cihazları Listelemek

`ifconfig` programı parametresiz çağırıldığında mevcut ağ cihazlarını \(NIC, Network Interface Controller\) listeler.

```bash
eaydin@dixon ~ $ ifconfig
eth0      Link encap:Ethernet  HWaddr a0:d3:c1:5e:68:ec  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:7869 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7869 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:3719917 (3.7 MB)  TX bytes:3719917 (3.7 MB)

wlan0     Link encap:Ethernet  HWaddr 80:56:f2:5b:ad:ab  
          inet addr:192.168.99.20  Bcast:192.168.99.255  Mask:255.255.255.0
          inet6 addr: fe80::8256:f2ff:fe5b:adab/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:416423 errors:0 dropped:0 overruns:0 frame:0
          TX packets:281587 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:531301670 (531.3 MB)  TX bytes:38844271 (38.8 MB)
```

Yukarıdaki çıktıda 3 cihaz görülüyor.

`eth0`: Ethernet kartı cihazı. Açık konumda \(UP\) ancak bir IP adresi yok. **HWaddr** ile gösterilen kısım MAC adresi.

`lo`: Lookback interface. Yerel ağ'ın oluşması, 127.0.0.1 IP adresinin bir yere işaret edebilmesi için oluşan sanal cihaz.

`wlan0`: Başka bir ethernet kartı. Adından anlaşılacağı üzere kablosuz \(wireless\) cihaz. Hem IPv4 hem de IPv6 adresleri görülüyor. Ayrıca **RX** ve **TX** ile giden, gelen paket sayısı ve miktarı gösteriliyor. Cihaz açıldığından beri 531.3MB indirme \(download, receive\) yapmış, 38.8MB gönderme \(upload, transmisson\) yapmış.

Tek cihazı görüntülemek için parametre olarak ismi verilebilir.

```bash
eaydin@dixon ~ $ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr a0:d3:c1:5e:68:ec  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

Bazı durumlarda cihazların tamamı listelenmeyebilir. Özellikle DOWN durumda olan cihazları da görüntülemek için `-a` parametresi kullanılmalıdır.

`ifconfig -a`

Komut bazı bilgileri `/proc` altındaki dosyalardan öğrenir. Aşağıda birkaç örneğini görebilirsiniz.

```bash
eaydin@dixon ~ $ cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
eth0:       0       0    0    0    0     0          0         0        0       0    0    0    0     0       0          0
lo: 3925336    8112    0    0    0     0          0         0  3925336    8112    0    0    0     0       0          0
```

```bash
eaydin@dixon ~ $ cat /proc/net/if_inet6
00000000000000000000000000000001 01 80 10 80       lo
fe800000000000008256f2fffe5badab 03 40 20 80    wlan0
```

### iwconfig

Hangi cihazın kablosuz arayüzünün olduğunu \(wireless extension\) anlamak için `iwconfig` komutu kullanılabilir.

```bash
eth0      no wireless extensions.

wlan0     IEEE 802.11bgn  ESSID:"aydin"  
          Mode:Managed  Frequency:2.427 GHz  Access Point: 18:28:61:D3:B2:ED   
          Bit Rate=58.5 Mb/s   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:off
          Link Quality=46/70  Signal level=-64 dBm  
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:1716  Invalid misc:1703   Missed beacon:0

lo        no wireless extensions.
```

iwconfig komutu, bu bilgileri `/proc/net/wireless` dosyasını okuyarak derler.

```bash
eaydin@dixon ~ $ cat /proc/net/wireless
Inter-| sta-|   Quality        |   Discarded packets               | Missed | WE
 face | tus | link level noise |  nwid  crypt   frag  retry   misc | beacon | 22
 wlan0: 0000   44.  -66.  -256        0      0      0   1716   1712        0
```

iwconfig komutu ifconfig ile aynı şekilde kullanılabilir. ifconfig ile kullanacağınız bütün parametreleri iwconfig ile kullıp kablosuz cihazlarınızı ayarlayabilirsiniz.

## Temel Komutlar

Aşağıda ifconfig ile temel cihaz ayarlamalarının nasıl yapıldığını göreceğiz.

**ÖNEMLİ NOT**: Bu ayarlar sistemi doğrudan etkiler, reboot gerektirmez. Dolayısıyla cihazın IP adresini veya benzer bilgileri değiştirirseniz internet erişimini etkileyebilirsiniz. Uzaktan bağlı olduğunuz cihazlarda bu komutları kullanırken dikkatli olmanızı tavsiye ederiz. Bu ayarlar sistem reboot olduğunda kaybolur, yani `/etc/network/interfaces` veya `/etc/sysconfig/network-scripts/ifcfg-eth0` gibi dosyaları düzenlediğinizde olduğu gibi kalıcı değildir.

### Cihaz Açıp Kapatmak

Cihazları \(örn. eth0\) aktif hale getirmek için

```bash
ifconfig eth0 up
```

veya

```bash
ifup eth0
```

Pasif hale getirmek içinse

```bash
ifconfig eth0 down
```

veya

```bash
ifdown eth0
```

### IP Adresi Atamak

Bir cihaza IP adresi atamak için cihaz isminden sonra doğrudan IP'yi yazabilirsiniz.

```bash
ifconfig eth0 192.168.42.5
```

### Netmask Atamak

Cihazın netmaskını belirtmek için kullanılır, daha önce belirttiğimiz gibi CIDR notasyonunu desteklemez.

```bash
ifconfig eth0 netmask 255.255.255.0
```

### Broadcast Tanımlamak

Netmask tanımlamaya oldukça benzerdir.

```bash
ifconfig eth0 broadcast 192.168.42.2555
```

### IP Adresi, Netmask ve Broadcast'i Aynı Anda Tanımlamak

Genellikle yukarıdaki işlemler tek komut ile gerçekleştirilir.

```bash
ifconfig eth0 192.168.42.5 netmask 255.255.255.0 broadcast 192.168.42.255
```

### MTU Ayarlamak

MTU \(Maximum Transmission Unit - Maksimum Aktarım Birimi\) ayarlanabilir. Bu ayarı her kart desteklemeyebilir, çoğunlukla değiştirmenize de gerek olmaz. Ancak bir sebepten değiştirmek isterseniz aşağıdaki şekilde ayarlayabilirsiniz. Buradaki MTU birimi Byte cinsindendir.

```bash
ifconfig eth0 mtu 900
```

### Promiscuous Mode

Ethernet kartınız destekliyorsa \(son yıllarda büyük çoğunluğu destekliyor\) kartınıza gelen ancak sizi ilgilendirmeyen paketleri de CPU'ya gönderip işlemenize olanak sağlayabilirsiniz. Böylece ağınızdaki trafik hakkında fikir sahibi olabilirsiniz.

Açmak için

```bash
ifconfig eth0 promisc
```

Kapatmak için

```bash
ifconfig eth0 -promisc
```

### MAC Adresini Değiştirmek

Cihazınızın MAC adresini değiştirebilirsiniz. Genellikle ağdaki ARP tablolarının karışmasına sebep olacaktır bu yüzden dikkatli kullanmanızda fayda var.

```bash
ifconfig eth0 hw ether AA:BB:CC:DD:EE:FF
```



