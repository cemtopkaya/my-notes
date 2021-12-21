## Router == Gateway
Yönlendirici anlamındaki `router`'ın ilk zamanlardaki ve bu alandaki cihazları işaret eden ortak adı (jenerik) `gateway`'dir.

## Default Router
Ağdaki cihazın dışarıya çıkmak için açtığı ilk kapı. Evden dışarı çıkacaksanız aklınıza ilk gelen dış kapıya yönelmektir. Ancak yangın durumunda aklınıza "yangın merdivenine açılan kapı" gelecektir. Yani özel bir durum için farklı bir geçit tanımlanması haricinden varsayılan geçidinize `default gateway` diyoruz.

## Rotaları Yazdırmak
`route PRINT` komutunu konsolda yazdığımızda aşağıdaki gibi liste yazdırılacaktır. Bazı satırlarını hemen altına ayrıntılandıralım.
Öncelikle bilgisayarım `192.168.1.34` IP adresine sahip ve ona bu adresi ADSL modemim (`192.168.1.1`) veriyor.

```bash
$ route PRINT
....
IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0      192.168.1.1     192.168.1.34     35
# 0.0.0.0 Netmask adresiyle hedeflenen IP adresine 0.0.0.0 yani CIDR gösterimiyle 0.0.0.0/0 adresine gitmek için
# 192.168.1.34 ağ arayüzünden erişilebilen 192.168.1.1 geçidine gideceğiz. 
# Yani ADSL Modeme (192.168.1.1) ulaşmak için 192.168.1.34 IP'sinin tanımlı olduğu ağ cihazına uğrayacağım. 
#
# Başka bir okumayla; herhangi bir adrese (0.0.0.0/0) gitmek için ADSL modemimin MAC adresini kullanacağım.
# Yani X.X.X.X IP adresi için Data katmanında MAC adresi olarak 192.168.1.1 ağ geçidinin MAC adresini yazacağım.
#
# Örnek AKIŞ'ım ICMP paketiyle olsun: ping 85.86.87.88
# Hedef IP adresi için rota tablomdaki ilk satırı (0.0.0.0/0) kullanabiliyorum.
# ARP tablomda 192.168.1.1 makinasının MAC adresi yoksa önce bir ARP mesajı oluşturup ADSL Modeme doğru 192.168.1.34 ağ 
# arayüzünden göndereceğim:
#
# Network Katmanı PAKETİ:
# Source IP      Destination IP     TTL    Other           Protocol
# 192.168.1.34   85.86.87.88        128    Diğer bilgiler  ICMP
#
# Ama Data Katmanı PAKETİ'nde hedef IP'nin MAC adresi ADSL MODEM olmalı, ancak ARP Tablosunda bu bilgi YOK!
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# ???????????????    D0-C6-37-C4-90-BE  IPv4               [192.168.1.34   85.86.87.88        128    Diğer bilgiler  ICMP]
#
# O zaman önce ARP paketi göndereceğiz:
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# FF:FF:FF:FF:FF     D0-C6-37-C4-90-BE  ARP                Who has 192.168.1.1?
#
# Gelen ARP REPLY mesajı aşağıdaki gibi olup bu kez ICMP paketi gönderilecek: 
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# D0-C6-37-C4-90-BE  CC:DE:12:F3:A1     ARP                CC:DE:12:F3:A1
#
# Gelen mesajın içinden 192.168.1.1 IP'li ADSL modemin MAC adresi alınıp ICMP paketinde DESTINATION MAC kısmına yazılır.
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# CC:DE:12:F3:A1     D0-C6-37-C4-90-BE  IPv4               [192.168.1.34   85.86.87.88        128    Diğer bilgiler  ICMP]
#
# Paketi gönderdiğimiz ADSL modem, FRAME başlığını atarak IP paketinde yazılı hedef IP'ye (85.86.87.88) bakar.
# Hedef adresin (85.86.87.88) kendi rota tablosunda hangi ağ arayüzü üstünden gönderilmesi gerektiğini çözümler.
# Yeniden hedef makinanın MAC adresini çözümlemek için ARP paketi hazırlar:
# - Frame header'ın Source MAC kısmına kendi MAC adresini yazar, 
# - DESTINATION MAC için BROADCAST MAC adresini (FF:FF:FF:FF:FF) yazar,
# - Layer3 Protokol olarak ARP yazar,
# - ARP mesajının içeriğine "Who has 85.86.87.88?" yazar ve gönderir.
#
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# FF:FF:FF:FF:FF     CC:DE:12:F3:A1     ARP                Who has 85.86.87.88?
#
# Gelen ARP REPLY içinden
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# CC:DE:12:F3:A1     FA:EA:DA:CA       ARP                 FA:EA:DA:CA
#
# - ARP cevabındaki MAC adresini payload içinden çekerek (örneğin FA:EA:DA:CA olsun) ICMP mesajının Frame header'ına yazar,
# - IP paketinde TTL'yi 1 azaltarak 127 yapar,
# - ICMP mesajını içeren IP paketini hedefe gönderir,
# - Gateway Network katmanındaki IP paketinin Header bilgisindeki kaynak ve hedef IP adreslerini bilinçli istemedikçe DEĞİŞMEZ!
# - Böylece cevap mesajına yazılacak hedef IP adresi en baştaki 192.168.1.34 olarak tekrar yola çıkar
#
# Destination MAC    Source MAC         Layer3 Protocol    Payload
# FA:EA:DA:CA        CC:DE:12:F3:A1     IPv4               [192.168.1.34   85.86.87.88        127    Diğer bilgiler  ICMP]
#
# Gelen PING REPLY mesajları aynı yolla en baştaki bilgisayara ulaştırır.

          
       23.23.23.0    255.255.255.0         On-link       23.23.23.23    281
# Bilgisayarımda bir loopback kartı tanımlı ve adresini 23.23.23.23 vermişim. Ve bu kartın dahil olduğu ağdaki bilgisayarlara
# erişmek için bu 23.23.23.23 IP adresli ağ kartına gitmeliyim.
# İlk kez gördüğümüz On-Link ifadesi 
# - arada kimse yok, 
# - doğrudan kartın bağlı olduğu ağ benim bilgisayarım, 
# - benim bilgisayar 23.23.23.0-255 bloğuna doğrudan ARP mesajları gönderebilir
# anlamlarıdadır.
#
# Yine 0.0.0.0/0 adresi nasıl uçsuz bucaksız bir ağ tanımı yapıyorsa bu satırdaki 255.255.255.0 maskesi ise o kadar kısıtlayarak
# sadece 23.23.23.0 ile 23.23.23.255 arasındaki bilisayarları işaret ediyor.

      23.23.23.23  255.255.255.255         On-link       23.23.23.23    281
# Bu kez tek bir IP adresini işaretlediğimiz 255.255.255.255 maskesini görüyoruz. Sadece 23.23.23.23 IP'li bilgisayara erişmek 
# için maskemizi 255.255.255.255 diye belirtiyoruz. CIDR olarak yazarsak 23.23.23.23/32 olacak adrese 23.23.23.23 IP'li  
# ağ kartımız üstünden erişebiliyoruz ve yine arada bir başka bir geçit olmadığını On-Link ile ifade ediyor.

     23.23.23.255  255.255.255.255         On-link       23.23.23.23    281
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      169.254.0.0      255.255.0.0         On-link   169.254.204.176    281
      169.254.0.0      255.255.0.0         On-link   169.254.171.238    291
      169.254.0.0      255.255.0.0         On-link    169.254.49.201    281
   169.254.49.201  255.255.255.255         On-link    169.254.49.201    281
  169.254.171.238  255.255.255.255         On-link   169.254.171.238    291
  169.254.204.176  255.255.255.255         On-link   169.254.204.176    281
  169.254.255.255  255.255.255.255         On-link   169.254.204.176    281
  169.254.255.255  255.255.255.255         On-link   169.254.171.238    291
  169.254.255.255  255.255.255.255         On-link    169.254.49.201    281
  
# Alt satırda bir ağ arayüzümüz var ve IP adresi 172.17.192.1'dir. Bu adres DOCKER veya K8s sayesinde oluşturturulmuş.
# Yukarıdakilerden farkı CIDR bilgisi 172.17.192.0/20 olacak (https://www.ripe.net/about-us/press-centre/IPv4CIDRChart_2015.pdf).
     172.17.192.0    255.255.240.0         On-link      172.17.192.1    271
     172.17.192.1  255.255.255.255         On-link      172.17.192.1    271
   172.17.207.255  255.255.255.255         On-link      172.17.192.1    271
   
     172.21.112.0    255.255.240.0         On-link      172.21.112.1    271
     172.21.112.1  255.255.255.255         On-link      172.21.112.1    271
   172.21.127.255  255.255.255.255         On-link      172.21.112.1    271

# Bu kez internet üstündeki bir bilgisayara erişmek için statik bir rota tanımlanmış. Burada diyor ki; "176.236.170.162/32 
# makinasına gitmek istersen, 192.168.1.34 arayüzünün üstünden 192.168.1.1 ağ geçidine yönlenmelisin".
  176.236.170.162  255.255.255.255      192.168.1.1     192.168.1.34     35
  
      192.168.1.0    255.255.255.0         On-link      192.168.1.34    291
      192.168.1.1  255.255.255.255         On-link      192.168.1.34     35
     192.168.1.34  255.255.255.255         On-link      192.168.1.34    291
    192.168.1.255  255.255.255.255         On-link      192.168.1.34    291
     192.168.33.0    255.255.255.0         On-link      192.168.33.1    281
     192.168.33.1  255.255.255.255         On-link      192.168.33.1    281
   192.168.33.255  255.255.255.255         On-link      192.168.33.1    281
     192.168.68.0    255.255.255.0         On-link      192.168.68.1    291
     192.168.68.1  255.255.255.255         On-link      192.168.68.1    291
   192.168.68.255  255.255.255.255         On-link      192.168.68.1    291
     192.168.99.0    255.255.255.0         On-link      192.168.99.1    281
     192.168.99.1  255.255.255.255         On-link      192.168.99.1    281
   192.168.99.255  255.255.255.255         On-link      192.168.99.1    281
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link      192.168.33.1    281
        224.0.0.0        240.0.0.0         On-link      192.168.99.1    281
        224.0.0.0        240.0.0.0         On-link      192.168.1.34    291
        224.0.0.0        240.0.0.0         On-link   169.254.204.176    281
        224.0.0.0        240.0.0.0         On-link      192.168.68.1    291
        224.0.0.0        240.0.0.0         On-link       23.23.23.23    281
        224.0.0.0        240.0.0.0         On-link   169.254.171.238    291
        224.0.0.0        240.0.0.0         On-link    169.254.49.201    281
        224.0.0.0        240.0.0.0         On-link      172.17.192.1    271
        224.0.0.0        240.0.0.0         On-link      172.21.112.1    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link      192.168.33.1    281
  255.255.255.255  255.255.255.255         On-link      192.168.99.1    281
  255.255.255.255  255.255.255.255         On-link      192.168.1.34    291
  255.255.255.255  255.255.255.255         On-link   169.254.204.176    281
  255.255.255.255  255.255.255.255         On-link      192.168.68.1    291
  255.255.255.255  255.255.255.255         On-link       23.23.23.23    281
  255.255.255.255  255.255.255.255         On-link   169.254.171.238    291
  255.255.255.255  255.255.255.255         On-link    169.254.49.201    281
  255.255.255.255  255.255.255.255         On-link      172.17.192.1    271
  255.255.255.255  255.255.255.255         On-link      172.21.112.1    271
```  


## Örnek 

### SENARYO 1: Router Table üstünde yönlendirme yoksa paket düşer
10.0.0.10'dan 192.168.10.8'e ICMP paketi gidecek. Paket hazırlanır ve 0.10 makinasının varsayılan ağ geçidine gönderilir.

![image](https://user-images.githubusercontent.com/261946/146869736-41869e8c-2242-4875-bda6-d9b0eb67ec20.png)

ROUTER A'ya gelen paketin hedef IP adresi 192.168.10.8, yönlendirme tablosunda aranır ve bulunamaz. Paket düşürülerek senaryo sonlanır!

![image](https://user-images.githubusercontent.com/261946/146784852-b9568843-80b0-4577-a9b2-042fadaa0683.png)

Burada bir açıklama daha yapalım. Eğer 10.0.0.10 makinamızın konsolundan ping mesajını 192.168.10.8'e atarsak ve `ROUTER A`'da tanımlı bir yönlendirme yoksa alacağımız cevap **`Destination host unreachable`** olacaktır

```bash
C:\Windows\System32>ping 192.168.10.8
Pinging 192.168.10.8 with 32 bytes of data:
Reply from 10.0.0.1: Destination host unreachable. 
Reply from 10.0.0.1: Destination host unreachable. 
Reply from 10.0.0.1: Destination host unreachable. 
Reply from 10.0.0.1: Destination host unreachable.

Ping statistics for 192.168.10.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

Ancak aşağıdaki adımda `192.168.10.x` bloğuna gitmek istenirse `ROUTER B`'ye gidilsin diye bir rotayı `ROUTER A`'ya tanımladığımızda alacağımız mesaj **`Request timed out.`** olacaktır. Çünkü bu kez `ROUTER A`'dan `ROUTER B`'ye bir hedef mevcut ancak `ROUTER B`'ye gelen cevabın hedefi `10.0.0.10` adresine yani bize olacağı için ve bu yönde (`192.160.10.x` -> `10.0.0.x` ) bir  rota tanımlı olmadığı için mesajlar `ROUTER B` tarafından düşürülür. `ROUTER A`'nın isteklerine karşılık gelecek cevap mesajları dönemediği için **İSTEK ZAMAN AŞIMINA** uğruyor.

```bash
C:\Windows\System32>ping 192.168.10.8
Pinging 192.168.10.8 with 32 bytes of data: 
Request timed out. 
Request timed out. 
Request timed out. 
Request timed out.

Ping statistics for 192.168.10.8:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
    
C:\Windows\System32>
```


---

### SENARYO 2: Router Tablosunda yönlendirme bulunursa paket hedefe gider

#### Router A için 192.168.10.8'e gidilebilecek rota tanımlayalım

![image](https://user-images.githubusercontent.com/261946/146785517-58c74647-1445-46bd-a7f8-09ea1a4654ba.png)

A yönlendiricisi için diyoruz ki;
> Sana 192.168.10.x IP'sine gitmek isteyen bir paket gelirse, B yönlendiricisinin MAC adresini frame başlığına yaz ve gönder, B yönlendiricisi nereye göndermesi gerekiyorsa o bilir.

![image](https://user-images.githubusercontent.com/261946/146866223-b0cad6ca-5489-4af8-aee3-553a78772613.png)

Bu şekilde bir rota girdiğimizde tablomuza **SSSS**tatik rota eklemiş oluyoruz:

![image](https://user-images.githubusercontent.com/261946/146866667-e1d8c454-6a15-4c4e-926e-0d40fe2f4968.png)

Rota bilgisi girerken kodlar ile rotanın türünü belirtiyoruz:
| Kod | Anlamı |
| --- | --- |
| **C** | **connected** |
| **S** | **static** |
| R  | RIP |
| M  | mobile |
| B  | BGP |
| D  | EIGRP |
| EX | EIGRP external |
| O  | OSPF |
| IA | OSPF inter area |
| N1 | OSPF NSSA external type 1 |
| N2 | OSPF NSSA external type 2 |
| E1 | OSPF external type 1 |
| E2 | OSPF external type 2 
| i  | IS-IS |
| su | IS-IS summary |
| L1 | IS-IS level-1 |
| L2 | IS-IS level-2 |
| ia | IS-IS inter area |
| *  | candidate default |
| U  | per-user static route |
| o  | ODR |
| P  | periodic downloaded static route |

Şimdi tekrar akışı görelim:

#### İstemci -> Router A

İstemciden giden paketin IP başlığına kaynak ve hedef bilgilerini yazıp doğrudan varsayılan ağ geçidine göndereceğiz:

![image](https://user-images.githubusercontent.com/261946/146866978-b4ed8052-50dc-4ee9-8adb-501b56d368aa.png)

---

#### Router A -> Router B

A Yönlendiricisine gelen paketin hedef IP adresine (192.168.10.8) uygun bir yönlendirme var mı diye bakılacak ve bulunacak (`S  192.168.10.0/24 via 172.16.0.2`). Artık gideceği hedef `Router B` olacağından onun MAC adresini öğrenmek için ARP paketi gönderilecek

![image](https://user-images.githubusercontent.com/261946/146867459-fdfe63fa-8f24-44a5-813d-132406348b5f.png)
![image](https://user-images.githubusercontent.com/261946/146867533-bc737046-56c3-49af-9efc-32235a14aa64.png)

| DESTINATION MAC | SOURCE MAC | LAYER3 PROTOCOL | PAYLOAD |
| --- | --- | --- | --- |
|FF:FF:FF:FF:FF| 0C:29:FC:70:A5 | ARP | Who has 172.16.0.2? |

ARP REPLY gelince içinden `ROUTER B`'nin MAC adresi (diyelimki `BB:BB:BB:BB:BB:00` geldi) FRAME içinde `DESTINATION MAC` kısmına yazılacak ve gönderilecek:

| DESTINATION MAC | SOURCE MAC | LAYER3 PROTOCOL | PAYLOAD |
| --- | --- | --- | --- |
| BB:BB:BB:BB:00 | 0C:29:FC:70:A5 | IPv4 | \[ IPv4 paketi: SRC IP:`10.0.0.10`, DST IP:`192.168.10.8`, TTL:`127`, OTHER, PROTOCOL: `ICMP` ] |

![image](https://user-images.githubusercontent.com/261946/146867260-6790012e-9eec-4cc5-880b-1b856b3eec5d.png)

---

#### Router B -> Sunucu

`ROUTER B` gelen paketin hedef IP adresini yönlendirme tablosunda `192.168.10.x` bloğuna gönderilebilir olarak bulur. FRAME Header'a yazacağı `DESTINATION MAC` bilgisine ne yazacağını bilemediği için ARP mesajına `Who has 192.168.10.8?` yazarak `DESTINATION MAC: FF:FF:FF:FF:FF` ile BROADCAST yapar. 
Gelen cevaptan hedef makinanın MAC adresini alarak IPv4 Layer3 protokolüyle gidecek olan gerçek mesajın `DESTINATION MAC` bilgisini doldurarak hedefe ulaştırır.

![image](https://user-images.githubusercontent.com/261946/146871068-65c67795-6c17-44fc-9df3-9400a3d19b97.png)

![image](https://user-images.githubusercontent.com/261946/146871110-c1759e90-21d9-49e5-a1be-a8917407f391.png)

---

### SENARYO 3: Sunucudan cevabın ROUTER B'ye gönderilmesi

#### Sunucu -> ROUTER B
Sunucu gelen mesaja cevap olarak sıfırdan `ICMP REPLY` mesajı oluşturur ve TTL değerini `128` olarak yeniler. Gelen mesajın `FRAME HEADER`'ında yer alan `SOURCE MAC ADDRESS` değerini bu kez giden mesajın `DESTINATION MAC ADDRESS` kısmına yazar ve gönderir.

![image](https://user-images.githubusercontent.com/261946/146871375-6f5d0e69-94c8-4f83-b4bf-d1750e834cdc.png)

---

#### ROUTER B -> Bitbucket

Gelen mesajın hedef adresi `10.0.0.10` ve yönlendirme tablosunda olmadığı için paket düşürülür ve senaryo biter.

![image](https://user-images.githubusercontent.com/261946/146871464-e0be3cc8-79e3-4052-87cf-a748e53311a3.png)

---

#### ROUTER B'ye 10.0.0.x bloğuna gidecek paketler için ROTA EKLEYELİM

`ROUTER B`'nin `F0/0` bacağından `ROUTER A`'nın `F0/1` bacağına hedef IP bloğu `10.0.0.x` için `S 10.0.0.0/24 via 172.16.0.1` rotasını `ROUTER B`'nin tablosuna ekliyoruz.
Artık paketler düşmeyecek !

![image](https://user-images.githubusercontent.com/261946/146871912-06ed1a6c-9663-41ba-8945-bdade0464c9f.png)

Sunucunun mesajı tekrar `ROUTER B`'ye gönderdiğini düşünelim ve akışa bir kez daha bakalım. Mesajı alan `ROUTER B` yönlendirme tablosunda `10.0.0.x` bloğuna gidecek kurala bakarak `172.16.0.1` IP adresine göndermesi gerektiğini görecek. O halde `172.16.0.1`'in MAC adresini öğrenmek için önce ARP mesajı gönderecek, gelen ARP REPLY içinden MAC adreini ICMP REPLY mesajının FRAME HEADER'ına yazarak gönderecek.

![image](https://user-images.githubusercontent.com/261946/146872463-4c8053f9-5217-467b-9513-2273f7a84bf3.png)

---

#### ROUTER A -> İstemci

Artık mesaj `ROUTER A`'da. Tablosunda `10.0.0.x` bloğunun doğrudan (`On-link`) `F0/0` bacağına bağlı olduğunu görüp `10.0.0.10` makinasının MAC adresini öğrenmek için ARP  mesajını tüm ağa yayacak ve gelen mesaj bizim zavallı istemcimizden olacağı için onun MAC adresiyle paketin `FRAME HEADER`'ını süsleyerek gönderecek.

![image](https://user-images.githubusercontent.com/261946/146873243-7193a32a-3ff0-467e-a242-c4900e3beb6a.png)

Artık ICMP REPLY mesajı istemcide:
![image](https://user-images.githubusercontent.com/261946/146873295-19b0aed6-9dc0-4884-a17c-c6768532e979.png)

Sonuç olarak iki yönlendiricinin tablosuna birbirlerinin rota bilgilerini eklemiş olarak akışı sağlayabildik:

![image](https://user-images.githubusercontent.com/261946/146873581-b1d3be04-b6e3-4571-96f6-caa0a9d642bd.png)
