# Time To Live

Bir paketi bilgisayarımızdan, uzaktaki bir bilgisayara gönderdik. Ağda dolaşa dolaşa hedef makinayı bulacak. Bir defada karşı bilgisayarı bulamayacağı için o HOP senin bu HOP benim "faruk eczanesini" sora sora ilerleyecek. 


![IP Paketi](https://user-images.githubusercontent.com/261946/146690569-2fa8e1cb-ed57-46b9-a100-0fe4e82f3b9c.png)

Eğer ki 4 HOP arasında aşağıdaki gibi dönüp dolaşarak hedefe ulaşamaz ve yönlendiricilerin (router) işlemci, bellek, bant genişlikleri gibi kaynaklarını boşa harcarsa diye TTL adında bir değer IP paketi içinde tanımlıdır.

![image](https://user-images.githubusercontent.com/261946/146690510-463a1126-82cc-40dc-b49e-877fbf3592c5.png)

Örneğin TTL değeri olarak 25 ayarladık (referans videodaki örnektir) ve her HOP geçerken bu değerden 1 eksildi ve 4 beceriksiz yönlendiricinin arasında kalıp döndü durdu. Her HOP geçtiğinde 1 eksilerek; 24, 23, 22...2,1 diye sıfıra gelecek. 0'ı gören HOP paketi düşürerek bu kaynak sarfiyatını sonlandıracak. 

![image](https://user-images.githubusercontent.com/261946/146690447-2bd77f45-80da-4236-a468-f836e08d6c88.png)

# MAX HOP

TTL ile aynı mantıkta bu kez her HOP geçtiğinde 1 arttırıyoruz. Eğer geçilebilecek en çok HOP noktasını geçmişsek paket yine Hakkın rahmetine kavuşturulur.


# tracert Uygulaması

- TRACERT tanılama yardımcı programı, hedefe İnternet Kontrol Mesaj Protokolü (ICMP) yankı paketleri göndererek bir hedefe giden yolu belirler. 
- Bu paketlerde TRACERT, değişen IP Yaşam Süresi (TTL) değerleri kullanır. Yol boyunca her yönlendiricinin paketi iletmeden önce paketin TTL'sini en az 1 azaltması gerektiğinden, TTL etkin bir şekilde bir atlama sayacıdır. 
- Bir paket üzerindeki TTL sıfıra (0) ulaştığında, yönlendirici kaynak bilgisayara bir ICMP "Zaman Aşıldı" mesajı gönderir. 
- TRACERT, TTL'si 1 olan ilk yankı paketini gönderir ve hedef yanıt verene veya maksimum TTL'ye ulaşılana kadar sonraki her iletimde TTL'yi 1 artırır. 
- Ara yönlendiricilerin geri gönderdiği ICMP "Süre Aşıldı" mesajları rotayı gösterir. 
- Ancak bazı yönlendiricilerin, süresi dolmuş TTL'leri olan paketleri sessizce bıraktığını ve bu paketlerin TRACERT tarafından görülmediğini unutmayın. 
- TRACERT, ICMP "Zaman Aşıldı" mesajlarını döndüren ara yönlendiricilerin sıralı bir listesini yazdırır. 
- `-d` seçeneğini tracert komutuyla kullanmak, TRACERT'e her IP adresinde bir DNS araması yapmamasını söyler, böylece TRACERT yönlendiricilerin yakın arabiriminin IP adresini bildirir.

```bash
tracert -d -h maximum_hops -j host-list -w timeout target_host
```

-d               : Specifies to not resolve addresses to host names

-h maximum_hops  : Specifies the maximum number of hops to search for the target

-j host-list     : Specifies loose source route along the host-list

-w timeout       : Waits the number of milliseconds specified by timeout for each reply

target_host      : Specifies the name or IP address of the target host


![image](https://user-images.githubusercontent.com/261946/146691202-6893eab1-55bf-49f3-b636-e21b00700330.png)

![image](https://user-images.githubusercontent.com/261946/146691385-97b9070b-3037-4ca1-8958-078b0ffe9c6a.png)

## tracert vs ping 
PING karşıdaki makinanın aktif ve bağlantı kurulup kurulamayacağını söyler. Tracert ise hoplar arasında geçişi gösterir. Yukarıdaki açıklamada ICMP paketinin tracert komutuyla ilişkili olan kısmını okuyabiliriz. İkisinin çıktısını, özellikle ICMP'nin TTL değerlerini okuyabilmek adına güzel bir gösterim:

![image](https://user-images.githubusercontent.com/261946/146693408-45e6c29c-fd6e-43ed-8921-8583b4ddb3c2.png)

Eğer TTL süresini girmezsek TTL=54 içinde hedefe varabiliyor.
Ancak `ping -i 10 www.opensuse.org` komutuyla gittiğimizde hedefe varacak kadar TTL değeri kalmadığından hedefin bir önceki HOP noktasında paket düşürülüyor.
Şimdi `-ì 7` değeri için bakalım:

![image](https://user-images.githubusercontent.com/261946/146693515-8f094d5a-f44a-4c7a-ad4b-7ef94e42b9ce.png)



#### Kaynaklar
- [Lim Jet Wee](https://www.youtube.com/watch?v=KEQcWP6bXds)
- [ping vs tracert](https://www.youtube.com/watch?v=g7emt39_OP8&list=PLrHVSJmDPvloic8M6wi3VhtE-fhoSngd6&index=105)
- [Tracert](https://support.microsoft.com/en-us/topic/how-to-use-tracert-to-troubleshoot-tcp-ip-problems-in-windows-e643d72b-2f4f-cdd6-09a0-fd2989c7ca8e)
- [Ağ dersleri verdiği bu çalma listesi de güzel](https://www.youtube.com/watch?v=U2HwVlROQ1I&list=PLrHVSJmDPvloic8M6wi3VhtE-fhoSngd6)
- [vTrace Uygulamasının demosu](https://www.youtube.com/watch?v=9Du5VPMoeO8&list=PLrHVSJmDPvloic8M6wi3VhtE-fhoSngd6&index=16)
