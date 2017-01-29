# Scapy'ye başlangıç

Scapy'nin etkileşimli kabuğu, bir terminal oturumunda çalışır.Paket gönderirken kök kullanıcı izni gerektiğinden, burada **sudo** kullanmamız gerekir.

```
$ sudo scapy
Welcome to Scapy (2.0.1-dev)
>>>
```

Windowsta da benzer şekilde administrator yetkileriyle cmd'ye girip, ilgili dizine gidip aşağıdaki gibi yazarak Scapy etkileşimli kabuğunu başlatabilirsiniz.

```
C:\>scapy
INFO: No IPv6 support in kernel
WARNING: No route found for IPv6 destination :: (no default route?)
Welcome to Scapy (2.0.1-dev)
>>>
```

Eğer tüm isteğe bağlı paketler -_indirme ve yükleme kısmında bahsedilenler_- yüklenmemişse, Scapy, sizi bu konu hakkında bilgilendirecektir.

```
INFO: Can't import python gnuplot wrapper . Won't be able to plot.
INFO: Can't import PyX. Won't be able to use psdump() or pdfdump().
```

Ancak bu sadece bilgilendirmedir.Paket gönderme ve almada herhangi bir sorun yaratmayacaktır.

# Etkileşimli satır

Hadi bir paket tanımlayalım.

```
>>> paket = IP(ttl=10)
>>> paket
<IP ttl=10 |>
>>> paket.src
'127.0.0.1'
>>> paket.dst="192.168.2.1"
>>> paket
<IP ttl=10 dst=192.168.2.1 |>
>>> paket.src
'192.168.2.47'
>>> del(paket.ttl)
>>> paket
<IP dst=192.168.2.1 |>
>>> paket.ttl
64
```

Buradaki açıklanması gereken terimler:

ttl \(_time-to-live_\) : Paketin yaşam süresidir.

src \(_source_\) : Paketin kaynağını belirtir.

dst \(_destination_\): Paketin hedefini belirtir.

# Katmanlamak \(Stacking layers\)

"**/"** operatörü, iki katman arasında birleşim operatörü olarak kullanılır.Bunu yaparken, alt katman, üst katmana göre varsayılan alanlarının bir veya daha fazlasını aşırı yükleyebilir\([_overloading_](https://en.wikipedia.org/wiki/Operator_overloading)\).Bir katar\(_string_\), bir ham katman olarak kullanılabilir.

```
>>> IP()
<IP |>
>>> IP()/TCP()
<IP frag=0 proto=tcp |<TCP |>>
>>> Ether()/IP()/TCP()
<Ether type=0x800 |<IP frag=0 proto=tcp |<TCP |>>>
>>> IP()/TCP()/"GET / HTTP/1.0\r\n\r\n"
<IP frag=0 proto=tcp |<TCP |<Raw load='GET / HTTP/1.0\r\n\r\n' |>>>
>>> Ether()/IP()/IP()/UDP()
<Ether type=0x800 |<IP frag=0 proto=ipencap |<IP frag=0 proto=udp |<UDP |>>>>
>>> IP(proto=55)/TCP()
<IP frag=0 proto=55 |<TCP |>>
```

![](http://www.secdev.org/projects/scapy/doc/_images/fieldsmanagement.png)

# Eğlenmeye Başlamak

Buraya kadar Scapy'nin [resmi sitesini](http://www.secdev.org/projects/scapy/doc) takip ederek içeriği oluşturdum. Aslında **oluşturdum** kelimesinden ziyade **çevirdim** demek daha mantıklı olacaktır.Çünkü önceki açıklamalarda çok az kendi yorumumu kattım.Ve farkettim ki gerçekten sitede fazlasıyla teorik anlatıyor.Üstelik bazı yerler eski kalmış -_sitede sorunsuz gözüken örnek bende hata verebiliyor_-.Bundan sonra benim anladığım kadarıyla size de anlatmaya çalışacağım -_dilim döndükçe_-.Burdan sonra fazlasıyla örnek ve açıklama -_hatta bazen ilgili sniff paketlerini de_- göreceksiniz.

Sonuçta giriş kısmında da bahsedildiği gibi Scapy bir domain-specific language.Yani tam olmasa da bir programlama dili.Onun için keşfedilecek tonlarca özelliği var.-_Eğer varsa_- Hatalarımı normal karşılarsanız sevinirim.

**Not**: Sizin iyi seviyede -_en azından okuyunca mantığını anlayabilecek_- Python ve Ağ kavramlarına hâkim olduğunuzu varsayıyorum.

# Genel komutlar

```
>>> ls() ---> Protokol/katmanları listeler
>>> lsc() ---> Komutları listeler
>>> conf ---> Konfigürasyonları gösterir
>>> help(sniff) --> Özel bir komut için yardım sayfası görüntüler.
```

### Ping göndermek

Hadi en basitten başlayalım.Adım adım ağın gateway'ine ping gönderelim.

İlk olarak, Scapy'ye girip "ip" adında bir değişken tanımlayalım.Ve bu değişkene, IP sınıfından oluşturduğumuz -_hedef olarak gateway'in ip adresini parametre olarak alan_- nesneyi atayalım.

```
>>> ip = IP(dst="192.168.2.1")
>>> ip
<IP dst=192.168.2.1 |>
```

Sonra ping göndermek ICMP protokolü üzerinden sağlandığı için, ICMP\(\) sınıfından da bir örneği "ping" adında bir değişkene atayalım. \(_Bunu yapmanıza gerek yok,direk **sr** fonksiyonuna parametre olarak da verebilirsiniz ama değişkene atamak protokol ayarlarını değiştirmede işinizi kolaylaştırır._\)

```
>>> ping = ICMP()
```

Şimdi **sr\(\)** fonksiyonunu kullanarak bu paketi yollayacağız.

**sr** fonksiyonu\(_send and receive_\) paket yollamayı ve verilen cevabı almayı sağlar.3. Katmanda çalışır.

```
>>> sr(ip/ping)
Begin emission:
Finished to send 1 packets.
*
Received 1 packets, got 1 answers, remaining 0 packets
(<Results: TCP:0 UDP:0 ICMP:1 Other:0>, <Unanswered: TCP:0 UDP:0 ICMP:0 Other:0>)
```

Ağı izlediğinizde normal bir ping paketi görürsünüz.Bunda hiçbir gariplik yok.

Bu basit işlem sizi etkilememiş olabilir.Fakat bu sizi vazgeçirmesin.Çünkü aşağıdaki özelliklerin ağzınızı açık bırakacağına eminim-_en azından bende öyle oldu :\)_-

### Ağda ayakta olmayan bir makine tarafından ping göndermek

Evet çok basit.Sadece üstteki **ip** değişkeninin özelliklerinden **src** yani kaynağa istediğimiz ip'yi atamamız.Örneğin, 192.168.2.199 ip adresinden yine gateway'e ping göndermek istiyoruz.Normalde böyle bir makine ağ üzerinde mevcut değil.Ama bunun adına gateway'e ping atabiliriz.

```
>>> ip.src = "192.168.2.199"
```

Şimdi **display\(\)** adlı özellik fonksiyonuyla değişkenin değerlerini öğrenelim.

```
>>> ip.display()
###[ IP ]###
version= 4
ihl= None
tos= 0x0
len= None
id= 1
flags=
frag= 0
ttl= 64
proto= hopopt
chksum= None
src= 192.168.2.199
dst= 192.168.2.1
\options\
```

Yukarıdaki gibi **ip** değişkenin **src **özelliğine görüldüğü gibi istediğimiz ip değerini atadık.Şimdi paketlerimiz hazır.Şimdi tek yapmamız gereken **sr** fonksiyonunu tekrar çalıştırıp paketi göndermek.

```
>>> sr(ip/ping)
Begin emission:
.Finished to send 1 packets.
.....^C
Received 6 packets, got 0 answers, remaining 1 packets
(<Results: TCP:0 UDP:0 ICMP:0 Other:0>, <Unanswered: TCP:0 UDP:0 ICMP:1 Other:0>)
```

Daha kolay izleyebilmek için Wireshark'a "**ip.src == 192.168.2.199 && icmp**"** **parametresini vermeniz yeterlidir.

Üstte **sr** fonksiyonunun yerine **send** fonksiyonunu da kullanabilirdik.Tek farkı **send**'in sadece paket göndermesi, **sr**'nin ise gönderip cevap beklemesidir.Kısaca ileride de bolca kullanacağımız için **sr** kullandık.İsterseniz **send** fonksiyonunu da aynı parametrelerle çalıştırabilirsiniz.

**Not**: Gördüğünüz gibi ping gidiyor ama geri cevap gelmediği için noktalar hâlinde çıktı gösteriyor.Bitirmek için Ctrl+C yapmak zorunda kaldım.

Ağı izlediğinizde, paketin tam istediğiniz gibi yol izlediğini görürsünüz.

İlkinde hemen çıktı almasını ve sonlanmasını, ikincisinde ise hiç bitmemesinin nedenini anlamışsınızdır diye umuyorum.Fakat yine de anlatayım.İlkinde ping'i kendi ip'mizle gönderdik, bu ping paketi gateway'e gitti ve gateway bizim ip'ye beklediğimiz paketi yolladı.

Fakat ikincide, biz ayakta olmayan bir makine adına ping yolladık.Gateway bunu o makineden gelmiş sandı ve o makine ile iletişim kurmaya çalıştı.Ama bağlantıyı bizimle kurduğu için paketleri bize yolladı.Açıklamaya dikkat ederseniz -_Received 6 packets, got 0 answers_- 6 paket alındığını ve bunlardan hiçbirinin cevabı içermediğini söylüyor.Yani gateway'in yolladığı paket bize uygun değil.

### TCP protokolü üzerinden istenilen bayrağa sahip TCP paketi göndermek

Şimdi bir senaryo düşünün, bu senaryo gereği sizin istediğiniz bayrağa, hatta istediğiniz hedef ve kaynak portlarını sahip bir TCP paketini bir adrese göndermeniz gereksin.Senaryoda **53** portundan **80** portuna, **192.168.2.133** ip numaralı makineye **FIN** bayraklı bir TCP paketi ile bağlantı kurduğunuzu varsayalım.Bunu yapmanız Scapy'de fazlasıyla rahattır.

Önce IP sınıfından hedef ip'yi içeren bir nesne örnekleyelim ve değişkene atayalım:

```
>>> ip = IP(dst="192.168.2.133")
```

Sonra TCP sınıfından kaynak portu\(s_port_\), hedef portu\(_dport_\) ve TCP bayrakları\(_flags_\) içeren bir nesne örnekleyelim ve değişkene atayalım:

```
>>> tcp = TCP(sport=53,dport=80,flags="F")
```

Şimdi sadece gönderelim:

```
>>> send(ip/tcp)
.
Sent 1 packets.
```

Tamamdır.Sorunsuz bir şekilde isteğimizi gerçekleştirdik.İsterseniz aşağıdaki paketi inceleyebilirsiniz:

![](https://lh5.googleusercontent.comC-02DF1pQxOAo_usgISh13nHWeQY54jUr4TdmyN9MyZsQO_i_yhH1DuAF_GSYBz9ter7Y7lwKHbg854=w1366-h650)



#
