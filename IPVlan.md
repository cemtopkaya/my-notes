# IPVlan 



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
