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
# Başka bir deyişle; herhangi bir adrese (0.0.0.0/0) gitmek için ADSL modemine uğramam gerekecek. 
# Modeme ulaşmak için 192.168.1.34 IP'sinin tanımlı olduğu ağ cihazına uğrayacağım. 
          
       23.23.23.0    255.255.255.0         On-link       23.23.23.23    281
# Bilgisayarımda bir loopback kartı tanımlı ve adresini 23.23.23.23 vermişim. Ve bu kartın dahil olduğu ağdaki bilgisayarlara
# erişmek için bu 23.23.23.23 IP adresli ağ kartına gitmeliyim.
# İlk kez gördüğümüz On-Link ifadesi "arada kimse yok, doğrudan kartın bağlı olduğu ağ" anlamındadır.
# Yine 0.0.0.0/0 adresi nasıl uçsuz bucaksız bir ağ tanımı yapıyorsa bu satırdaki 255.255.255.0 maskesi ise o kadar kısıtlayarak
# sadece 23.23.23.0 ile 23.23.23.255 arasındaki bilisayarları işaret ediyor.

      23.23.23.23  255.255.255.255         On-link       23.23.23.23    281
# Bu kez tek bir IP adresini işaretlediğimiz 255.255.255.255 maskesini görüyoruz. Sadece 23.23.23.23 IP'li bilgisayara erişmek 
# için maskemizi 255.255.255.255 diye belirtiyoruz. CIDR olarak yazarsak 23.23.23.23/32 olacak adrese 23.23.23.23 IP'li ağ kartımız 
# üstünden erişebiliyoruz ve yine arada bir başka bir geçit olmadığını On-Link ile ifade ediyor.

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

# Bu kez internet üstündeki bir bilgisayara erişmek için statik bir rota tanımlanmış. Burada diyor ki; "176.236.170.162/32 makinasına 
# gitmek istersen, 192.168.1.34 arayüzünün üstünden 192.168.1.1 ağ geçidine yönlenmelisin".
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
