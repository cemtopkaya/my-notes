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
