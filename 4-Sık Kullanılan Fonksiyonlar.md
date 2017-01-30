# Sık kullanılan fonksiyonlar

Tüm komut listesi için etkileşimli satıra **lsc\(\)** -_list commands_- yazabilirsiniz.

İlgili komutların Scapy'deki yardım dosyaları için **help **fonksiyonuna parametre olarak verebilirsiniz. Ör. **help\(send\)**

##

## Paket alma ve gönderme

### send fonksiyonu

Temel olarak paket yollamaya yarar.3. katmanda\(_ağ katmanı_\) çalışır.Geri herhangi bir cevap dönüp dönmediğini kontrol etmez.

İlk parametre olarak göndereceği paketi alır.Sonrakiler isteğe bağlıdır:

**count** = &lt;_Kaç adet paket gönderileceğini belirtir_&gt;

**loop** = _&lt;Paket göndermede döngüye girer,0 için dönmez, pozitif tüm değerler için sonsuz döner,negatif değerler için sayı kadar döner&gt;_

**inter** = _&lt;Paketin kaç saniye aralıklarla gönderileceğini belirtir&gt;_

**iface** = _&lt;Paket alıp göndermede kullanılacak ağ arayüzünü belirtir&gt;_

```
>>> p = IP(dst="192.168.2.1")/TCP(dport=3350,flags="S")
>>> packet = send(p,loop=-10,inter=0.25,iface="wlan0")
```

Artık söz dizimine aşina olduğunuzdan dolayı karmaşık yazmakta sakınca görmüyorum.İlk satırda **/** operatörünü kullanarak IP sınıfının **dst** parametresi ile örneklediğimiz nesneyi,TCP sınıfının **dport**\(_destination port_\) ve **flags** parametresi ile örneklediğimiz nesne ile birleştirdik.

İkinci satırdaki **send **fonksiyonunda loop değerine verilen -10 değeri, döngünün toplam 10 kere dönmesini, inter değeri her 0.25 saniyede bir paket yollanmasını,iface değeri de wlan0 arayüzünü kullanacağımızı belirtir.

### sr fonksiyonu

**sr **fonksiyonu 3. katmanda\(_ağ katmanı_\) çalışır.Bu fonksiyonun ilk parametresi göndereceğiniz paket olmalıdır.Diğer isteğe bağlı parametreleri ise aşağıda gösterilmiştir.

**filter** = _&lt;_[_Berkeley Packet Filter_](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter)_ ile filtreler&gt;_

**retry** = _&lt;Yanıtlanmamış paketler için tekrar deneme sayısı&gt;_

**timeout** = _&lt;Zaman aşımı süresi \(saniye\) &gt;_

**iface** = _&lt;Paket alıp göndermede kullanılacak ağ arayüzünü belirtir&gt;_

Örneğin:

```
>>> p = IP(dst="192.168.2.1")/TCP(dport=3350,flags="S")
>>> packets = sr(p,retry=3,timeout=1.5,iface="wlan0",filter="host 192.168.2.1 and port 3350")
```

İlk satır üstte **send** fonksiyonunda verilenin aynısı zaten.

Sonra **sr\(\) **fonksiyonuna verdiğimiz parametreler ile 3 kere paket göndereceğini\(_eğer cevap gelmezse,birine cevap gelirse paket göndermeyi kesecektir_\), zaman aşımının 1.5 saniye olacağını, ağ arayüzü olarak wlan0 kullanılacağını ve sadece hedefin ip adresinin 192.168.2.1 ve hedef portun 3350 olduğunu söyledik.Cümle biraz karmaşık oldu ama gereksiz açıkladığımı düşünüyorum.Zaten satırlarda her şey belli oluyor :\)

### sr1 fonksiyonu

1. katmandan\(_ağ katmanı_\) paket yollar ve ilk cevabı döndürür.**sr fonksiyonu**yla aynı parametreleri alır kullanımı aynıdır.Tek farkı

**sr1**'in aldığı ilk cevabı döndürmesidir.

**Not**: Aslında **sr1** in çok daha farklı yönleri var.Ama sanırım bu biraz uzmanlık istiyor.Şu durumda ben Scapy'yi yeni yeni öğrendiğim için bu konuda net bir şey söyleyemem sadece şöyle bir örnekle açıklayayım.

```
>>> a = sr(p)
Begin emission:
.Finished to send 1 packets.
*
Received 2 packets, got 1 answers, remaining 0 packets
>>> b = sr1(p)
Begin emission:
Finished to send 1 packets.
*
Received 1 packets, got 1 answers, remaining 0 packets
>>> type(a)
<class 'tuple'>
>>> type(b)
<class 'scapy.layers.inet.IP'>
```

### sendp, srp ve srp1 fonksiyonları

Bu fonksiyonların kullanımı yukarıdaki **send, sr **ve **sr1 **fonksiyonlardan önemli bir noktada ayrılır.Üstte belirttiğimiz gibi **send,sr **ve **sr1** fonksiyonu 3. katmanda çalışır.**sendp,srp **ve **srp1** ise 2. katmanda çalışır.

Yani sonunda **p **olan fonksiyonlar, 3. katman yerine 2. katmanda çalışacağını belirtir.

##

## Ağ izleme

### sniff fonksiyonu

Adından da anlaşılacağı üzere ağı izlemeye\(_koklamak_\) yarar.Yazının devamında izlemek yerine koklamak deyimi kullanılacaktır.

Aldığı parametreler:

**count** = _&lt;Kaç paket koklayacağını belirtir.0 sonsuz anlamına gelir&gt;_

**filter** = _&lt;Berkeley Packet Filter ile filtreler&gt;_

**prn** = _&lt;Her pakete uygulanacak fonksiyon.Genelde _[_lambda fonksiyonlar_](http://belgeler.istihza.com/py3/ileri_fonksiyonlar.html#lambda-fonksiyonlari)_ kullanılır.&gt; _Ör: **prn = lambda x : x.summary\(\)**

**lfilter** = _&lt;Her pakete uygulanıp başka işlemlerin yapıp yapılmayacağını belirler.Yine lambda fonksiyonu olarak kullanılır.&gt; _Ör: **lfilter = lambda x : x.haslayer\(Padding\)**

**offline **= _&lt;Koklamak yerine, pcap dosyasından paketleri okumaya yarar&gt; _Ör: **offline="izle.pcap"**

**timeout** = _&lt;Verilen süre kadar koklanır.Süre dolduğunda durur.Varsayılan değeri **None**'dur&gt;_

**L2socket** = _&lt;Verilen 2. katman soketini kullanır&gt;_

**opened\_socket** = _&lt;Soket nesnesi sağlar&gt;_

**stop\_filter** = _&lt;Her pakete uygulanıp o paketten sonra koklamanın durup durmayacağını belirler&gt; _Ör: **stop\_filter= lambda x : x.haslayer\(TCP\)**

**exceptions** = _&lt;Kullanıcı tarafından kesmelerde, KeyboardInterrupt\(Ctrl+C\) gibi _[_istisna_](http://www.istihza.com/resmi/py3/kilavuz/errors.html#istisnalar)_ları yakalar&gt;_

```
>>> var = sniff(count=3,filter="udp and host 192.168.2.1",prn=lambda x:x.summary(),timeout=30)
Ether / IP / UDP / DNS Qry "'ssl.gstatic.com.'"
Ether / IP / UDP / DNS Qry "'ssl.gstatic.com.'"
Ether / IP / UDP / DNS Qry "'drive.google.com.'"
>>> var
<Sniffed: TCP:0 UDP:3 ICMP:0 Other:0>
```

Yukarıdaki komutta,Scapy'ye toplam 3 adet UDP protokolü kullanılarak ve 192.168.2.1 ip adresine sahip cihaza gidip/gelen paketleri koklamasını, her paketin **summary\(\)** özellik fonksiyonu ile özetini çıkarmasını ve en fazla 30 saniye koklamasını belirttik.

Çıktıyı **var** adlı bir değişkene atadık.Bu değişken liste gibi yönetilebilir.

Bir paketin kullanışlı fonksiyonları aşağıdadır:

**str\(paket\)** = _&lt;Paketini birleştirir\(assemble\)&gt;_

**hexdump\(paket\)** = _&lt;Paketin hexadecimal\(onaltılık\) hâlini verir&gt;_

**ls\(paket\)** = _&lt;Paketin alanlarının listesini verir&gt;_

**paket.summary\(\)** = _&lt;Paketin tek satır özetini verir&gt;_

**paket.show\(\)** = _&lt;Paketin gelişmiş görünümünü verir&gt;_

**paket.show2\(\)** = _&lt;show metoduyla aynıdır fakat birleştirip\(assemble\) gösterir.&gt;_

**paket.sprintf\(\)** = _&lt;İstenilen özelliği yazdırır&gt;_ Ör: **paket.sprintf\("%TCP.flags%"\)**

**paket.psdump\(\)** = _&lt;PostScript diyagramı çizer \(İndirme kısmında bahsedilen ayrı paketlere ihtiyaç duyar\)&gt;_

**paket.pdfdump\(\)** = _&lt;PDF çizer \(İndirme kısmında bahsedilen ayrı paketlere ihtiyaç duyar\)&gt;_

##

## PCAP dosyalarını yönetme

### wrpcap fonksiyonu

Pcap dosyası yazmaya yarar -_write pcap_-. Dosyanın bulunmasına gerek yoktur.Yeni dosya yaratabilir.

İlk olarak dosya adını, sonra yazılacak paket\(ler\)i alır.

```
>>> v = IP(dst="192.168.2.1")/TCP(flags="S")
>>> wrpcap("test.pcap",v)
>>> sniff(offline="test.pcap")
<Sniffed: TCP:1 UDP:0 ICMP:0 Other:0>
```

**Not**: sniff fonksiyonunun offline parametresi ile pcap dosyaları okuyabildiğimizden yukarıda bahsetmiştik.

### rdpcap fonksiyonu

Pcap dosyasını okumaya yarar.

İlk olarak pcap dosyasını alır.İkinci parametre olarak da **count **parametresi ile kaç adet paket alınacağını alır.

```
>>> rdpcap("test.pcap",count=1)
<test.pcap: TCP:1 UDP:0 ICMP:0 Other:0>
```





# Kaynakça

[https://blogs.sans.org/pen-testing/files/2016/04/ScapyCheatSheet\_v0.2.pdf](https://blogs.sans.org/pen-testing/files/2016/04/ScapyCheatSheet_v0.2.pdf)

[https://thepacketgeek.com/scapy-p-06-sending-and-receiving-with-scapy/](https://thepacketgeek.com/scapy-p-06-sending-and-receiving-with-scapy/)

