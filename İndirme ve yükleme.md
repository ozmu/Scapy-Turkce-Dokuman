# Genel Bakış

1. Python indirin.

2. Scapy'yi indirin ve kurun.

3. libpcap ve libdnet ve onların Python paketlerini indirin. \(_Linux hârici işletim sistemleri için_\)

4. Özel durumlar için ek paketler yükleyin. \(_isteğe bağlı_\)

5. Scapy'yi kök kullanıcı\(_root_\) izinleriyle çalıştırın.


Scapy'nin versiyonu, kullandığınız platforma bağlıdır ve toplamda 2 farklı versiyonu vardır:

**Scapy v1.x**: Yalnızca bir dosyadan oluşur ve Python 2.4 üzerinde çalışır, bu nedenle kurulum daha kolaydır.

**Scapy v2.x**: Şu anki versiyonu birkaç özellik ekler\(_ör. IPv6_\).Paketlenmiş birkaç dosyadan oluşur ve Python 2.5 gerektirir.

# Scapy v2.x yükleme

**Not**: Aşağıdaki adımlar Unix tarzı\(_Linux,BSD,Mac OS X_\) işletim sistemlerine uygun olarak yazılmıştır.Windows kurulum için [şu link](http://www.secdev.org/projects/scapy/doc/installation.html#windows)'e bakabilirsiniz.

**Not2**: "\#" ilgili satırı açıklamak için kullanılan yorum satırlarını belirtir.

Son versiyonu "geçici" dizinine \(_tmp_\) indirmek ve standart şekilde yüklemek için aşağıdaki adımları uygulayın:

```
$ cd /tmp
$ wget scapy.net #Eğer index.html olarak indiriyorsa, index'te sadece zip dosyası
$ unzip scapy-latest.zip #olduğu için 'unzip index.html' yapınca da dosyaları çıkaracaktır.
$ cd scapy-2.*
$ sudo python setup.py install
```

Alternatif olarak, zip dosyasını direk çalıştırabilirsiniz:

```
$ chmod +x scapy-latest.zip
$ sudo ./scapy-latest.zip
```

veya:

```
$ sudo sh scapy-latest.zip
```

veya:

```
$ mv scapy-latest.zip /usr/local/bin/scapy
$ sudo scapy
```

**Not**: Bir zip dosyasını çalıştırılabilir yapmak, zip başlığından önce bazı baytlar eklemeyle gerçekleşir.Çoğu zip programı bunu yapar fakat hepsi yapacak diye bir durum söz konusu değildir.

# En son sürümden haberdâr olmak

Eğer her zaman, son eklenen özellikler ve hata çözümleri ile son versiyonu edinmek istiyorsanız Scapy'nin Mercurial deposunu kullanmanız gerekir.

1.[Mercurial](/www.selenic.com/mercurial/) sürüm kontrol sistemini yükleyin. Debian/Ubuntu'da:

1. ```
$ sudo apt-get install mercurial
```


veya OpenBSD'de:

1. ```
$ pkg_add mercurial
```


2.Scapy'nin deposundaki kopya ile kıyaslayın:

1. ```
$ hg clone http://hg.secdev.org/scapy
```


3.Scapy'yi standart yoldan kurun:

1. ```
$ cd scapy
$ sudo python setup.py install
```


Daha sonra her zaman son sürüme güncelleyebilirsiniz:

```
$ hg pull
$ hg update
$ sudo python setup.py install
```

# Scapy v1.2 yükleme

Scapy v1, yalnızca bir dosyadan oluştuğu için yüklenmeYa da ssittir.Sadece indirin ve Python yorumlayıcınız ile çalıştırın.

```
$ wget http://hg.secdev.org/scapy/raw-file/v1.2.0.2/scapy.py
$ sudo python scapy.py
```

# Özel istekler için isteğe bağlı yazılımlar

Bazı ek özellikler için daha fazla yazılım yüklemeniz gerekir.Örneğin bir grafik çıkartmak\(_plotting_\) istediğinizi düşünün.Bunun için gerekli olan **plot\(\)** fonksiyonu, [GnuPlot](http://www.gnuplot.info/) ve [NumPy](http://www.numpy.org/) paketlerine/modüllerine ihtiyaç duyar.

```
>>> p=sniff(count=50)
>>> p.plot(lambda x:len(x))
```

Ya da 2 Boyutlu grafiklerde kullanılan **psdump\(\)** ve **pdfdump\(\) **fonksiyonları, [PyX](http://pyx.sourceforge.net/) paketine ihtiyaç duyar.

```
>>> p=IP()/ICMP()
>>> p.pdfdump("test.pdf")
```

Diyagram oluşturmada kullanılan **conversations\(\) **fonksiyonu, [Grapviz](http://www.graphviz.org/) ve [ImageMagick](http://www.imagemagick.org/script/index.php) paketlerine ihtiyaç duyar.

```
>>> p=readpcap("myfile.pcap")
>>> p.conversations(type="jpg", target="> test.jpg")
```

Üç boyutlu grafiklerde kullanılan **trace3D\(\)** fonksiyonu [VPython](http://www.vpython.org/) paketine ihtiyaç duyar.

```
>>> a,u=traceroute(["www.python.org", "google.com","slashdot.org"])
>>> a.trace3D()
```

WEP çözümünde\(_WEP decryption_\) kullanılan **unwep\(\)** fonksiyonu [PyCripto](https://www.dlitz.net/software/pycrypto/)'ya ihtiyaç duyar.

```
>>> enc=rdpcap("weplab-64bit-AA-managed.pcap")
>>> enc.show()
>>> enc[0]
>>> conf.wepkey="AA\x00\x00\x00"
>>> dec=Dot11PacketList(enc).toEthernet()
>>> dec.show()
>>> dec[0]
```

Parmak izi toplamaya\(_fingerprinting_\) yarayan **nmap\_fp\(\)** fonksiyonu [Nmap](https://nmap.org/)'e ihtiyaç duyar.

```
>>> load_module("nmap")
>>> nmap_fp("192.168.0.1")
Begin emission:
Finished to send 8 packets.
Received 19 packets, got 4 answers, remaining 4 packets
(0.88749999999999996, ['Draytek Vigor 2000 ISDN router'])
```

VOIP için kullanılan **voip\_play\(\)** fonksiyonu [SoX](http://sox.sourceforge.net/)'a ihtiyaç duyar.
