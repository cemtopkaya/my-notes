
# IPVlan 

Önce IPVlan nedir, ne işe yarar öğrenelim.
Tabii ki, ipvlan (IP Virtual LAN), Docker'ın desteklediği bir ağ modudur ve Linux kernel'inin özelliklerini kullanarak konteynerlere özgü IP adreslerine sahip olmalarını sağlar. Ipvlan, birden çok IP adresine sahip bir ağ arabirimini oluşturur ve her bir IP adresi bir konteynerle ilişkilendirilebilir.

İşte ipvlan ile ilgili bazı temel kavramlar:

1. **Ipvlan Modları:**
   - **L2 Mode (Layer 2 Mode):** Konteynerler, host'a doğrudan bağlı gibi davranır, aynı fiziksel ağda bulunan diğer cihazlarla aynı Layer 2 ağında gibi iletişim kurabilirler.
   - **L3 Mode (Layer 3 Mode):** Konteynerler, host'un IP adresi kullanılarak iletişim kurarlar, ancak host üzerindeki ağ trafiği hala Layer 2 üzerinden yönlendirilir.

2. **Konteynerlerin Kendi IP Adresine Sahip Olması:**
   - Ipvlan, her bir konteynerin kendi IP adresine sahip olmasını sağlar. Bu, konteynerlerin host ağındaki diğer cihazlarla daha doğrudan iletişim kurmalarına olanak tanır.

3. **Performans ve İzolasyon:**
   - Ipvlan, host üzerindeki Linux kernel özelliklerini kullanarak çalıştığından, performans açısından etkili bir çözümdür. Ayrıca, konteynerler arasında izolasyon sağlar.

4. **Docker Ipvlan Kullanımı:**
   - Ipvlan modunu kullanmak için bir Docker konteyneri oluştururken `--network ipvlan` seçeneğini kullanabilirsiniz. 

Örneğin:
Aşağıdaki komut satırı çıktılarını tek tek inceleyerek örneğimize başlayalım.
![image](https://github.com/cemtopkaya/my-notes/assets/261946/19419a26-27aa-471d-92ae-db966e55f206)

#### 1. Ağ Kartlarının 1-2 Katmanlarındaki Bilgileri
`$ ip -a -br -c l` komutuyla ev sahibi bilgisayarın ağ kartları hakkında `Layer 1` (fiziki katman, elektrik sinyalleri seviyesi) ve `Layer  2` (Veri -Data- katmanında MAC adresi yer alır) seviyelerinde bilgileri çekebiliriz.

![image](https://github.com/cemtopkaya/my-notes/assets/261946/7d75b31e-0ade-413d-a7a4-4749accd482d)

![image](https://github.com/cemtopkaya/my-notes/assets/261946/5c2aa20d-7c16-4112-a3d6-b617851e370a)

#### 2. Docker Ağları
`$ docker network ls` komutuyla varsayılan ağları görüyoruz ( `none`, `bridge`, `host` ). Buraya bizim tanımlayacağımız `ipvlan` türünde bir ağ tanımı birazdan gelecek. Burada eklenmişi var :))) sırf fikir olsun diye son halini de görün:
![image](https://github.com/cemtopkaya/my-notes/assets/261946/4a9ec144-d659-4ddd-8194-5e3f91848e39)

#### 3. Docker Konteynerleri
`$ docker ps` komutuyla çalışan konteynerlerin olmadığını görüyoruz.
![image](https://github.com/cemtopkaya/my-notes/assets/261946/ad6c7626-e3a8-487c-be80-a6b634f116d1)


#### 4.  Ağ Kartlarının 3. Katmanındaki Bilgileri
`$ ip -a -br -c a` komutuyla ev sahibi bilgisayarın ağ kartlarının  `Layer 3` ağ (network) katmanı seviyelesinde bilgileri çekebiliriz.
![image](https://github.com/cemtopkaya/my-notes/assets/261946/fe7b4165-17ae-423d-be19-ce6cc452531b)

#### 5. ipvlan Sürücüsünü Kullanan Docker Ağı Yaratılıyor
Ipvlan, iki ana mod ile kullanılabilir: Layer 2 (L2) mode ve Layer 3 (L3) mode. Bu modlar, konteynerlerin host üzerinde nasıl göründüğünü ve nasıl ağ trafiği yönlendirildiğini belirler.

Önce `bridge` sürücülü varsayılan docker ağına bağlanarak oluşturulan bir konteynerin kendine özgü bir MAC adresiyle yaratıldığını görelim:
![image](https://github.com/cemtopkaya/my-notes/assets/261946/9449153c-509f-4639-ae8f-79dded4eea20)

Aynı konteyner oluşturma işini peş peşe 3 kez tekrar edince sırayla her konteyner için özgün bir MAC adresi oluşturulduğunu göreceksiniz:
![image](https://github.com/cemtopkaya/my-notes/assets/261946/9f0ebaff-3c22-4615-a093-bb6a5cc49be7)

Peki şimdi bu konteynerleri, ev sahibi bilgisayarın `eth0` ağ kartını `parent` alacak şekilde yaratacağımız bir `ipvlan` ağına bağlayarak oluşturalım:

- önce ipvlan sürücüsünü kullanan kendi IP bloğu olan bir ağ yaratalım
- sonra konteyneri bu ağa bağlanacak şekilde `--network <ağ adı>` bayrağıyla oluşturalım 

```bash
$ docker network create              \
            --driver ipvlan          \
            --subnet=10.20.20.0/24   \
            --gateway=10.20.20.1     \
            --opt mode=l2     \
            --opt parent=eth0 \
            yeni_ag_l2

$ docker run -d --network yeni_ag_l2 nicolaka/netshoot sleep 100
```
`Layer 2` Modunda bir ağ yarattık (`ipvlan_mode=l2`) ve parent olarak fiziki ağ kartını (`eth0`) gösterdik (`ipvlan_parent=eth0`). Konteyneri bu ağa bağlayacak şekilde oluşturduk ve konteynerin kendine ait özgün bir MAC adresi oluştuğunu gördük çünkü Data Layer (layer 2) hedeflenmişti.

```bash
$ docker network create --driver ipvlan --subnet=10.20.20.0/24 --gateway=10.20.20.1 --opt mode=l2 --opt parent=eth0 yeni_ag_l2
48af008b33df7b820cd32ec7b0d8a3e152cc43cc691319d77f429732bdf156bc

$ docker run -d --network yeni_ag_l2 nicolaka/netshoot sleep 100
95fa3045a843f91442ac5edffaf85553ad960d8776fab202c22aecbe3790c021

$ docker exec -it 95 ip -br -c -a link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0@if2         UNKNOWN        00:15:5d:10:e0:5c <BROADCAST,MULTICAST,UP,LOWER_UP>

$ ip -br -c -a link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             00:15:5d:10:e0:5c <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          DOWN           02:42:26:85:a7:a0 <NO-CARRIER,BROADCAST,MULTICAST,UP>

$ docker inspect -f '{{json .}}' yeni_ag_l2 | jq '{name: .Name, driver: .Driver, containers: .Containers}'
{
  "name": "yeni_ag_l2",
  "driver": "ipvlan",
  "containers": {
    "95fa3045a843f91442ac5edffaf85553ad960d8776fab202c22aecbe3790c021": {
      "Name": "elated_carver",
      "EndpointID": "a8e91f3dd5299829bbe846cd6119c09f415cafdc1cf9da03b85768536fe20fc3",
      "MacAddress": "",
      "IPv4Address": "10.20.20.2/24",
      "IPv6Address": ""
    }
  }
}
```

Şimdi Layer 3 modunda bir ağ ve bu ağa bağlı konteyner yaratalım:
```bash
$ docker network ls && echo ----- && docker ps && echo ----- && ip -br -c -a addr && echo ---- && ip -br -c link
NETWORK ID     NAME      DRIVER    SCOPE
fbd2ddef8aa6   bridge    bridge    local
74a450fd2704   host      host      local
5fd49e26190c   none      null      local
-----
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
-----
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.18.108.207/20 fe80::215:5dff:fe10:e05c/64
docker0          DOWN           172.17.0.1/16 fe80::42:26ff:fe85:a7a0/64
----
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             00:15:5d:10:e0:5c <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          DOWN           02:42:26:85:a7:a0 <NO-CARRIER,BROADCAST,MULTICAST,UP>
cemt@PC-CEM-TOPKAYA:~$
cemt@PC-CEM-TOPKAYA:~$ docker network create --driver ipvlan --subnet=10.30.30.0/24 --gateway=10.30.30.1 --opt mode=l3 --opt parent=eth0 yeni_ag_l3
f094d8abeb8cd5cdd94307441588ebf7e253bfc80771db66820dafa16dec0ccc
cemt@PC-CEM-TOPKAYA:~$
cemt@PC-CEM-TOPKAYA:~$ docker run -d --network yeni_ag_l3 nicolaka/netshoot sleep 100
0531c0be4cf66b5c3aa74e8091c8a2b5c2969de1b99133a2109ebf1b32989d05
cemt@PC-CEM-TOPKAYA:~$
cemt@PC-CEM-TOPKAYA:~$ docker exec -it 05 ip -br -c -a link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0@if2         UNKNOWN        00:15:5d:10:e0:5c <BROADCAST,MULTICAST,UP,LOWER_UP>
cemt@PC-CEM-TOPKAYA:~$
cemt@PC-CEM-TOPKAYA:~$ docker exec -it 05 ip -br -c -a addr
lo               UNKNOWN        127.0.0.1/8
eth0@if2         UNKNOWN        10.30.30.2/24

```

![image](https://github.com/cemtopkaya/my-notes/assets/261946/c2cde0b7-6fea-4e52-a34e-75af77a35d4d)


```bash
$ docker inspect -f '{{json .}}' bridge | jq '{name: .Name, driver: .Driver, containers: .Containers}'
```

1. **Layer 2 (L2) Mode:**
   - **Açıklama:** Layer 2 modu, her bir konteynerin kendine özgü bir MAC (Media Access Control) adresi ile aynı fiziki ağda bulunan diğer cihazlar gibi davranmasını sağlar. Bu modda, konteynerlerin host'a doğrudan bağlı gibi davranmaları söz konusudur.
   - **Kullanım:** `--ipvlan_mode=l2` veya `-o ipvlan_mode=l2` seçeneği ile belirlenir.

2. **Layer 3 (L3) Mode:**
   - **Açıklama:** Layer 3 modu, konteynerlerin host'un IP adresi kullanılarak iletişim kurmalarını sağlar, ancak host üzerindeki ağ trafiği hala Layer 2 üzerinden yönlendirilir. Bu modda, konteynerlerin host'un IP adresi ile aynı alt ağda olmaları önemlidir.
   - **Kullanım:** `--ipvlan_mode=l3` veya `-o ipvlan_mode=l3` seçeneği ile belirlenir.

Bu modlar, konteynerlerin ağdaki diğer cihazlarla nasıl etkileşimde bulunacağını belirler. L2 modu genellikle konteynerlerin daha geniş bir ağda yer almasını ve ağdaki diğer cihazlarla doğrudan iletişim kurmalarını sağlamak için kullanılırken, L3 modu genellikle host'un IP adresi üzerinden iletişim kurmalarını sağlamak için kullanılır.
Bir `ipvl
an` sürücüsünde ağ yaratacaksanız iki modda çalışlıyor `l2`, `l3`
```bash
docker network create --driver ipvlan  --opt mode=l2 --opt parent=eth0 ipvlan_network
```
 
![image](https://github.com/cemtopkaya/my-notes/assets/261946/26ca13c9-bd97-431f-9999-b019638b40f5)

```bash
docker run --network ipvlan --ip 192.168.1.2 -it ubuntu /bin/bash
```

     Bu komut, bir Ubuntu konteynerini ipvlan ağına bağlar ve konteynerin IP adresini belirler.

Ipvlan, özellikle her bir konteynerin kendi IP adresine sahip olması gerektiği durumlar için kullanışlıdır. Ancak, ipvlan modunu kullanmadan önce ağ yapılandırmanızı dikkatlice planlamanız önemlidir.

---

Kernel'e eklenmesiyle ilgili [IPVLAN Driver HOWTO](https://www.kernel.org/doc/Documentation/networking/ipvlan.txt).
L3 ve L2 de çalışabilen bir sürücü. L3 te Broadcast ve Multicast özelliklerini yitiriyor.

Örnek oluşturma kabuk komutu:
```shell
ip link add link <master> name <slave> type ipvlan [ mode MODE ] [ FLAGS ]

# L3 modu varsayılan ama MODE bilgisi şu değerleri alabilir: l3 (default) | l3s | l2
# FLAGS: bridge (default) | private | vepa

    e.g.
    (a) Following will create IPvlan link with eth0 as master in L3 bridge mode
          bash# ip link add link eth0 name ipvl0 type ipvlan

    (b) This command will create IPvlan link in L2 bridge mode.
          bash# ip link add link eth0 name ipvl0 type ipvlan mode l2 bridge
          
    (c) This command will create an IPvlan device in L2 private mode.
          bash# ip link add link eth0 name ipvlan type ipvlan mode l2 private
          
    (d) This command will create an IPvlan device in L2 vepa mode.
          bash# ip link add link eth0 name ipvlan type ipvlan mode l2 vepa
```

## IPVlan Örneği:

```
  +=============================================================+
  |  Host: host1                                                |
  |                                                             |
  |   +----------------------+      +----------------------+    |
  |   |   NS:ns0             |      |  NS:ns1              |    |
  |   |                      |      |                      |    |
  |   |                      |      |                      |    |
  |   |        ipvl0         |      |         ipvl1        |    |
  |   +----------#-----------+      +-----------#----------+    |
  |              #                              #               |
  |              ################################               |
  |                              # eth0                         |
  +==============================#==============================+
```

### Ağ İsim Uzayı Oluşturmak & Listelemek 
1. Ağ ad alanı (net namespace) oluşturacağız `ip netns add <isim uzayının adı>`
2. Ağ ad alanlarını listeleyeceğiz `ip netns list`

![image](https://user-images.githubusercontent.com/261946/147513333-875f41cd-81db-47cd-9d07-1a12ea27b53b.png)

### Cihazdaki Ağ Arayüzlerini & Bağlantı Durumlarını Listelemek
`ip link` Bize `Layer 2` ve `Layer 1` seviyesinde bilgi verecektir. Bu "ağ arayüzünün MAC adresi ne?" ve "elektrik geliyor mu?" sorularına `ip link` cevap verecek.

![image](https://user-images.githubusercontent.com/261946/147513502-1a34030d-f52b-4686-b7de-dfb24fffb0e9.png)

### IPVlan Türünde Sanal Ağ Arayüzü Oluşturmak
Çeşitli ağ arayüzü türleri mevcut ve şimdi `ipvlan` türünde bir tane oluşturalım. TYPE bilgisi gerekli ama MODE bilgisi vermezsek varsayılan `l3` yani Layer 3 olacak.
Bunun için `ip link add link < <arayüzün-adı> <type <ipvlan|macvlan|...>> [mode <l2|l3|l3s>]`

#### `ip link` İle Ağ Arayüz İşlemleri
Ağ arayüzleriyle (network interface) yani ağ kartlarıyla ilgili komut `ip link`. Yardım için tab tuşuna basabilirsiniz:

![ip_komutu ve tab](https://user-images.githubusercontent.com/261946/147514867-c73088f1-1b5c-4015-89bc-264a1d6b6766.gif)

#### `ip link add ...` İle Ağ Arayüzü Eklemek

![image](https://user-images.githubusercontent.com/261946/147513970-2f58bb3d-c206-4a28-91d0-d54902ee8c63.png)

Yeni ağ arayüzü oluşturmak için ikinci komut `add [link <aygıt-adı örn. eth0>] [arayüzün-adı-ne-istenirse] <type <türü: ipvlan|macvlan|...>> [mode <l2|l3|l3s>]`

```shell
ip link add link eth0 ipvl0 type ipvlan mode l2
```

![image](https://user-images.githubusercontent.com/261946/147514342-9653b4a1-acc6-4f3e-a37c-3da563749ad5.png)

Bir ağ arayüzü oluşturduğumuzda varsayılan olarak elektrik sinyali gelmediğini `state DOWN mode` ile görebiliyoruz. 

![image](https://user-images.githubusercontent.com/261946/147514547-2627b118-f973-40c7-8924-824aea9b9ec1.png)

#### `ip netns` İle Ağ İsim Alanı İşlemleri
Bir ağ isim alanı yaratalım:

![image](https://user-images.githubusercontent.com/261946/147514930-63fb3fd4-f33a-4959-916a-0d9a9a9183f1.png)

#### Ağ İsim Uzayına Ağ Kartını Bağlayalım
Ağ kartını -> ağ ad alanına `IP LINK SET` ile atıyoruz:

```shell
ip link set <ağ-kartının-adı> netns <isim-uzayının-adı>
```

![image](https://user-images.githubusercontent.com/261946/147515054-bc6f5a53-03f0-48ed-bb6b-ae37025e628c.png)

Sonuçta ağ kartı artık HOST'un üsütnden NETNS içine atandığı için HOST bilgisayar içinde `ip link` komutuyla göremiyoruz ;)

#### `ip netns exec ...` Ağ Ad Alanında Komut Çalıştırmak 
`ip netns exec <ağ-ad-alanının-ismi> <çalışacak komut>` ile ağ ad alanı içinde işleyecek komutları çalıştırabiliriz.

Aşağıdaki komut der ki; `bash` komutunu `ns0` isimli ağ ad alanında çalıştır ve sonraki tüm komutlarımı burada çalışıyor olacak şekilde işlet.
Bir nevi `docker exec -it <konteyner-adı> bash` komutu (ki aslında docker arkada `ip netns exec <konteyerner-adı> bash` komutunu çalıştırır).

```shell
ip netns exec ns0 bash
```

![image](https://user-images.githubusercontent.com/261946/147515461-391473f7-6cbe-4482-9b9e-a1e5cda70bb7.png)



### Toplu KoMutlar


1. İki ağ ad alanı yaratalım
```shell
$ ip netns add ns0
$ ip netns add ns1
```

2. Ağ alan adları için 2 IPVlan arayüzü yaratalım. 
```shell
$ ip link add link eth0 ipvl0 type ipvlan mode l2
$ ip link add link eth0 ipvl1 type ipvlan mode l2
```

3. Her ağ ad alanına bir ağ arayüzü atayalım.
```shell
$ ip link set dev ipvl0 netns ns0
$ ip link set dev ipvl1 netns ns1
```

4. Ağ arayüzlerini fiziki ağ kartımıza yönlendirelim.
  - Önce `ns0` adlı ağ isim alanımıza bağlanalım çünkü içinde komutlar çalıştıracağız
```shell
ip netns exec ns0 bash
```
  - Yarattığımız `ipvl0` isimli ağ arayüzü üçüncü adımda bu ağ ad alanına kurulmuştu şimdi enerjisini verelim (`up`)
```shell
$ ip link set dev ipvl0 up
```
  - Ağ ad alanında varsayılan olarak `x.x.x.x/32` (tek bir IP tanımlı -cidr: 32- olan ağın adı **`lo`**opback) ağ arayüzünü de ayaklandıralım
```shell
$ ip link set dev lo up
```
  - Loopback ağ arayüzüne IP adresi olarak 127.0.0.1 verelim 
```shell
$ ip -4 addr add 127.0.0.1 dev lo
```
  - `ipvl0` isimli ağ ad alanına da `$IPADDR` içeriğindeki IP adresini atayalım
```shell
$ ip -4 addr add $IPADDR dev ipvl0
```

  - Ve `ipvl0` ağ arayüzünün varsayılan çıkış kapısı olarak `$ROUTER` içindeki IP'yi gösterecek yönlendirmeyi yapalım
```shell
$ ip -4 route add default via $ROUTER dev ipvl0
```
