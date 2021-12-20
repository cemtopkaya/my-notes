# ARP - Addres Resolution Protocol

IP Adresi -> MAC Adresi 

![image](https://user-images.githubusercontent.com/261946/146691943-60ec9f9d-b6ab-4cf1-a734-ff6fa7371f79.png)

**Data Link** katmanıyla, **Network Link** katmanı arasında çalışır ARP mesajları. Kaynak ve hedefin IP adreslerini ağ katmanında yazılırken, MAC adresi veri katmanında yazılır. Eğer IP'nin karşılığı olan MAC adresi bilinmiyorsa `FF:FF:FF:FF:FF:FF` adresi yazılır ve bu sayede ağdaki tüm bilgisayarlara "`10.10.12.121` IP'li hedef adres sen isen bana MAC adresini ARP protokolüyle gönderdiğim bu isteği cevaplayarak bildir" deriz.

ARP isteği ile IP Adresinin MAC adresine çözümlemesi yapılır. Aşağıdaki istek çıktısı ve cevabında hedef IP'nin MAC adresi bilinmezken (00:00:00:00:00:00) gelen cevabın içeriğinde MAC bilgisini görebiliyoruz (00:09:0f:0c:57:d2)

![image](https://user-images.githubusercontent.com/261946/146691732-f0942551-3aee-469c-a7e3-41eb591c35e2.png)

![image](https://user-images.githubusercontent.com/261946/146691782-d645ff1e-010c-47ad-a763-8bc7a48eb2b2.png)

Not: ARP tablosu ile MAC tablosu %100 farklı tablolardır.

## ARP Tablosu
Bizim ADSL'ye bağlı olduğumuz ağ arayüzümüzün IP adresi `192.168.1.34`. Bu adresin kapısı ise `192.168.1.1`. 
O halde kapımızın ARP tablosuna `# arp -a | grep 192.168.1.34 -A9 -B1` komutuyla bakabiliriz (`-A9` ile bulunan satırın 9 satır üstü, `-B1` ile 1 satır altını getiririz):

![image](https://user-images.githubusercontent.com/261946/146694933-95229653-38ee-415a-8c5f-b4bd31d9f205.png)

Kablosuz ağımızın GATEWAY'i yani ADSL modemimiz bizim ARP mesajlarımıza cevap verenlerden olduğu için **LAYER 2 broadcast mesajlarımızı (`FF:FF:FF:FF:FF:FF`)** kabul ettiği için listenin en üstüne yazıyoruz.

```bash
  Internet Address      Physical Address      Type
  192.168.1.1           fc-f5-28-07-11-c4     dynamic
```

Listenin ikinci elemanı **LAYER 3 Broadcast adresi (`192.168.1.255`)** ve onun **LAYER 2 Broadcast adresi (`FF:FF:FF:FF:FF:FF`)**:
```bash
  192.168.1.255         ff-ff-ff-ff-ff-ff     static
```

Listenin üçüncü elemanı ise Microsoft'un IGMP (Internet Group Messaging Protocol) ile ağdaki cihazları bulmak için kullandığı Multi-cast adresi:
```bash
  224.0.0.22            01-00-5e-00-00-16     static
```

Diğer multicast adreslerimizden [224.0.0.251 - Multicast DNS](https://www.rfc-editor.org/rfc/rfc6762.html), [224.0.0.251 - Link-Local Multicast Name Resolution (LLMNR)](https://www.rfc-editor.org/rfc/rfc4795.html) 
``` bash
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
```

Sondan ikinci adresimiz `239.255.255.250` IP adresi  UpNp/SSDP multi cast adresidir.
Bu adres, çeşitli satıcılar tarafından UPnP (Evrensel Tak ve Çalıştır)/SSDP (Basit Hizmet Keşif Protokolü) için bir VLAN'daki cihazların yeteneklerinin reklamını yapmak (veya keşfetmek) için kullanılır. MAC OS, Microsoft Windows, IOS ve diğer işletim sistemleri ve uygulamaları bu protokolü kullanır. Her istemci cihazı, yeteneklerini diğer cihazlara tanıtmak için büyük olasılıkla bu protokolü kullanacağından, UPnP/SSDP trafik hacmi, ağa bağlanan yeni cihaz ve hizmetlerin miktarıyla orantılı olarak bir ağdaki trafiğin taşması nedeniyle hızla artabilir.
```bash
  239.255.255.250       01-00-5e-7f-ff-fa     static
```

Yani genel anlamda: IP çok noktaya yayın için 224 ile 239 arasında bir sayı ile başlayan adresler kullanılır. IP çok noktaya yayın, aynı içeriği birden çok hedefe verimli bir şekilde göndermek için bir teknolojidir. Diğer şeylerin yanı sıra finansal bilgileri ve video akışlarını dağıtmak için yaygın olarak kullanılır.

## ARP Tablosuna ICMP Paketiyle Yeni Satır Eklemek
Aynı ağdaki cep telefonuma PING atarak onun MAC adresi çözümlemesinin yapılması için ARP mesajlarının gidip gelmesini sağlıyorum. Ardından ARP cache tabloma `192.168.1.36` adresiyle cep telefonumun eklendiğimi görebilirim:

![image](https://user-images.githubusercontent.com/261946/146695375-a477fd58-c05e-4dfc-b5db-5695042267ac.png)

 ## ARP Tablosunu Tamamen Temizlemek
 `arp -d *` ile tüm tabloyu temizliyoruz, tabloyu görüntüleyip, cep telefonuna ping atıp tekrar tabloyu görüntülüyoruz:
 ```bash
 $ arp -d *
 $ arp -a | grep 192.168.1.34 -A9 -B
 $ ping 192.168.1.36 -n 1 
 $ arp -a | grep 192.168.1.34 -A9 -B
 ```
 
 ![image](https://user-images.githubusercontent.com/261946/146695498-d55e888d-5113-4005-867e-9854296fb0e1.png)


#### Kaynaklar
- [Lim Jet Wee](https://www.youtube.com/watch?v=pBj-7ez1RW0&list=PLrHVSJmDPvloic8M6wi3VhtE-fhoSngd6&index=27)
- [MAC adresinin temelleri](https://www.youtube.com/watch?v=FkiTOMn-XGw)
- [IP vs MAC Address](https://www.youtube.com/watch?v=LMbZWSVplHU)
- [Routing IPv4 and IPv6 - Address Resolution Protocol](https://app.pluralsight.com/course-player?clipId=022627cc-feca-40e1-80e3-787bee15b00e)
- [UpNp/SSDP](https://extremeportal.force.com/ExtrArticleDetail?an=000091058)
