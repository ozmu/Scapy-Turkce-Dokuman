# Scapy nedir ?

Scapy, ağ paketlerini tarama, araştırma, özel TCP/IP paketleri oluşturma, TCP/IP paket manipülasyonu, paketleri yakalama,istek ve cevapları eşleştirme ve çok daha fazlasını yapmada işinize yarayan, aynı zamanda modül olarak ta kullanılabilen bir python programıdır.Dolayısıyla bu anlatım boyunca, python\(_3_\) bildiğiniz varsayılacaktır.Nmap,packit,hping gibi bu işe özel port tarama ve paket üretme araçlarından farkı, bu araçların belirli bir amacı gerçekleştirmek için yazılmış olması, Scapy'nin ise paket yönetimi üzerinde adeta bir oyun hamuru gibi esnek bir yapı sunmasıdır.

![](http://www.secdev.org/projects/scapy/doc/_images/testing-taxonomy.png)

Basitçe düşünürsek, Scapy başlıca iki şey yapar: paket gönderme ve cevap alma.Siz bir paket kümesi tanımlarsınız, Scapy onları gönderir, cevapları alır ve isteklerle cevapları karşılaştırır.Sonuç olarak eşleşen paketler ve eşleşmeyen paketlerin listesini döndürür.Bu özelliği, -_Nmap veya hping'e göre tek bir cevaba \(açık/kapalı/filtreli\) indirgemesinden ziyade tüm paketi göstermesi_-büyük bir avantajdır.

# Scapy'yi özel kılan nedir ?

İlk olarak, diğer networking araçlarında, aracı yazan kişinin hayal ettiklerinin dışına çıkamazsınız.Bu araçlar belirli bir hedef için oluşturulmuştur ve ondan çok fazla bir sapma olmaz.Örneğin, bir ARP önbellek zehirleyen \([_ARP Cache Poisoning_](http://blog.btrisk.com/2016/01/arp-poisoning-nedir-nasil-yapilir.html)\) araç, 802.1q kapsüllemeye\([_double 802.1q encapsulation_](https://networklessons.com/switching/802-1q-encapsulation-explained/)\) izin vermeyecektir.

İkinci olarak, bu tarz araçlar çözme\(_decoding_\) ve durumu yorumlamayı karıştırırlar.Makineler çözmede iyidir fakat durumu yorumlamak size\(_insanlara_\) kalmış bir olgudur.Çoğu program, bunu taklit etmeye çalışır.Örneğin, "**SYN-ACK aldım**" yerine "**Bu port açık**" döndürürler.Bu durum bazen kullanışlı bazen kullanışlı değildir.Yeni başlayanlar için işe yarar gözükebilir ama arka planda neler olduğunu anlamaya çalışanlar için gereksiz ve kullanışsızdır.

Üçüncü olarak, sadece çözen programlar bile aldıkları tüm bilgiyi size vermez.Örneğin, size ethernet girintisi\(_ethernet padding_\) bilgisi veren bir araç biliyor musunuz ?

Scapy, bu sorunların üstesinden gelmeye çalışıyor.İstediğiniz paketleri tam olarak oluşturmanızı sağlar.İstediğiniz herhangi bir alana istediğiniz herhangi bir değeri koymakta özgürsünüz, istediğiniz gibi yığabilirsiniz.

Araştırmadan sonra, Scapy mutlaka yorumlanmadan sadece çözülmüş paketler verecektir.Bu, bir kez araştırıp verilen çıktıyı birçok kez yorumlayabileceğiniz anlamına gelir.

# Scapy'nin Kavramları

### Hızlı Paket Dizayn

Çoğu araç, kabuktan çalıştırılan program paradigmasına bağlıdır.Böyle olduğundan, sonuçta bir paket üretebilmek için korkunç bir söz dizimiyle karşılaşırız.Üstelik, aracın yazarının yazdığı seçeneklere bağlı olarak kullanmak zorunda kalırız.Örneğin, bir port tarama durumunda, kullandığınız araca IP'yi verdiğinizde, taramayı biraz değiştirseniz bile, sonuçta araç kodlandığı gibi çalışacaktır.

Scapy, bu duruma çözüm olarak her tür paketin güçlü ve hızlı tanımlanmasını sağlayan bir DSL\([_Domain Specific Language_](https://eksisozluk.com/entry/7173411)\) önermektedir.En büyük avantajı, DSL söz dizimi ve yorumlayıcısı yerine Python söz dizimini ve yorumlayıcısını kullanmasıdır.Bu sayede, kullananların baştan yeni bir dil öğrenmesinden ziyade Python gibi güçlü bir dilin bünyesinde kullanmaları sağlanmış olur.

Scapy, bir paketi veya paket kümesini katmanlar hâlinde tanımlamanızı sağlar.Scapy'de önceden belirlenmiş bir senaryo \(_ör. port tarama_\) veya şablon kullanmanız mecburi değildir.Bu, her paket göndermek istediğinizde yeni bir araç yazma derdinden kurtarır.C'de bir paket tanımlamak için ortalama 60 satır gerekirken, Scapy'de yalnızca bir satırda göndermek istediğiniz paketi tanımlayabilirsiniz.Kısacası, ağ tarama araçlarının %90'ı, Scapy'de 2 satırda tekrar yazılabilir.

### Bir kez tara, bir çok kez yorumla

Ağ keşfi, blackbox test yöntemidir.Bir ağı keşfederken, bir çok uyaran gönderilir fakat bazılarına cevap verilir.Doğru uyaranlar seçilirse, arzu edilen bilgi cevaplardan elde edilir.

![](http://www.secdev.org/projects/scapy/doc/_images/scapy-concept.png)

###

### Scapy çözer, çözüleni yorumlamaz

Sıklıkla kullanılan araçlardaki ortak problem, alınan cevabı çözüp yorumlamalarıdır.Örneğin bazı tarayıcılar, **ICMP hedef ulaşılamaz**\(_ICMP destination unreachable_\) paketi aldığında, bunu direk yazmak yerine **filtreli TCP portu** raporlamaya eğimlidir.Bu bazen doğrudur.Ancak, paketin güvenlik duvarı tarafından filtrelenmesinden ziyade paketin ulaşacağı sunucunun ayakta olmadığı bir senaryoda bizi yanıltacaktır.





# Kaynakça

[http://blog.lifeoverip.net/2010/08/23/scapy-ozellestirilebilen-tcpip-test-araci/](http://blog.lifeoverip.net/2010/08/23/scapy-ozellestirilebilen-tcpip-test-araci/)

[http://www.secdev.org/projects/scapy/doc/introduction.html](http://www.secdev.org/projects/scapy/doc/introduction.html)
