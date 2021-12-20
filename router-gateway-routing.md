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

Başka bir senaryoya bakalım:

10.0.0.0/24 Ağında `10.0.0.10` IP'li bilgisayarımızdan `192.168.10.8` makinasına ICMP paketi gönderelim.
Eğer statik rota tanımlıysa 192.168.10.8 IP adresinin hangi girdiyle bulunabileceğine bakacaktır.

Örneğin A yönlendiricisinin rota tablosu şöyle olsun:
```text
C   F0/0                  10.0.0.0/24
C   F0/1                  172.16.0.0/30
S   192.168.10.0/24  via  172.16.0.2
```

ilk kayıt 10.0.0.0/24 bloğu olduğu için es geçecek, ikinci kayıt da öyle, 3. kayıtta 192.168.10.8 IP adresinin bloğunu bulacağı için hedef olarak ROUTER B'nin MAC adresini yazar ve yönlendirir. ROUTER B'ye gelen paket aynı şekilde rota tablosunda baklır.

ROUTER B'nin rota tablosu şöyle olsun:
```text
C   F0/0                  172.16.0.0/30
C   F0/1                  192.168.10.0/24
```

B Yönlendiricisi ikinci kayıtta 192.168.10.0 bloğunda hedef IP'nin olduğunu görüp üzerindeki F0/1 ethernet arayüzüne yönlendirir ancak bu kez BROADCAST MAC adresini hedef MAC alanına yazarak ARP mesajı yayınlayarak 192.168.10.8 IP'li bilgisayarın "benim ben" diye cevaplamasını isteyecek. Gelen MAC bilgisini FRAME HEADER'ına yazarak IP paketini (ICMP mesajı içeren) gönderecek.

Ağdaki cihazların IP ve MAC adresleri:

| İSTEMCİ      | ROUTER A             | ROUTER B              | SUNUCU        |
| ------------ | -------------------- | --------------------- | ------------- |
|              |  F0/0: 172.16.0.1/30 | F0/0: 172.16.0.2/30   |               | 
|              |  AA:AA:00..          | BB:BB:00..            |               |
| 10.0.0.10/24 |  F0/1: 10.0.0.1/24   | F0/1: 192.168.10.1/24 | 192.168.10.8  | 
| 10:10:..     |  AA:AA:11..          | BB:BB:11..            | 88:88:..      |



Peki, ya statik rota yoksa?
İşte o zaman sürekli ARP paketleri yayarak hedefe götürecek HOP'ların MAC adreslerini toplayacak ve ortalık ARP mesajlarından geçilmeyecek.
