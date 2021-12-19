# Time To Live

Bir paketi bilgisayarımızdan, uzaktaki bir bilgisayara gönderdik. Ağda dolaşa dolaşa hedef makinayı bulacak. Bir defada karşı bilgisayarı bulamayacağı için o HOP senin bu HOP benim "faruk eczanesini" sora sora ilerleyecek. 


![IP Paketi](https://user-images.githubusercontent.com/261946/146690569-2fa8e1cb-ed57-46b9-a100-0fe4e82f3b9c.png)

Eğer ki 4 HOP arasında aşağıdaki gibi dönüp dolaşarak hedefe ulaşamaz ve yönlendiricilerin (router) işlemci, bellek, bant genişlikleri gibi kaynaklarını boşa harcarsa diye TTL adında bir değer IP paketi içinde tanımlıdır.

![image](https://user-images.githubusercontent.com/261946/146690510-463a1126-82cc-40dc-b49e-877fbf3592c5.png)

Örneğin TTL değeri olarak 25 ayarladık (referans videodaki örnektir) ve her HOP geçerken bu değerden 1 eksildi ve 4 beceriksiz yönlendiricinin arasında kalıp döndü durdu. Her HOP geçtiğinde 1 eksilerek; 24, 23, 22...2,1 diye sıfıra gelecek. 0'ı gören HOP paketi düşürerek bu kaynak sarfiyatını sonlandıracak. 

![image](https://user-images.githubusercontent.com/261946/146690447-2bd77f45-80da-4236-a468-f836e08d6c88.png)

# MAX HOP

TTL ile aynı mantıkta bu kez her HOP geçtiğinde 1 arttırıyoruz. Eğer geçilebilecek en çok HOP noktasını geçmişsek paket yine Hakkın rahmetine kavuşturulur.

Kaynak: [Lim Jet Wee](https://www.youtube.com/watch?v=KEQcWP6bXds)

[Ağ dersleri verdiği bu çalma listesi de güzel](https://www.youtube.com/watch?v=U2HwVlROQ1I&list=PLrHVSJmDPvloic8M6wi3VhtE-fhoSngd6)
