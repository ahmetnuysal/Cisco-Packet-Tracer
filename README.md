# Cisco Packet Tracer

- PC'lere IP'yi statik olarak arayüzden verebiliriz
- Komutun başına "no" ekleyerek iptal edebiliriz

#### Show Komutları

```
show vlan: ile vlanları ve bağlı portları görebiliriz
show ip route: Router tablosunu gösterir
show ip ospf int brief: Cost bilgisi verir
show history: Geçmiş komutları listeler
show interface: Arayüzleri gösterir
```

#### Switchlere İsim Vermek

```
conf t
hostname SW2 (switch'in ismini SW2 olarak değiştirir)
```
#### Router Aktif Etme

- Router içine giriyoruz
- Conf
- GigEth0/0 tıklayıp IP giriyoruz ve on yapıyoruz
- GigEth0/1 tıklayıp IP giriyoruz ve on yapıyoruz

```
en
conf t
int gi0/0
ip address 10.0.0.2 255.255.255.0
no shutdown
int gi0/1
ip address 192.168.2.1 255.255.255.0
no shutdown
```

- Router'da seri port kullanabilmek için Router 1941 ekliyoruz
- Physical
- Router'ı kapatıp HWIC-2T ekliyoruz

#### Router'a Yol Öğretme

Örneğin R1 ve R2 iki adet router'ımız var. R1'in iç ağındaki PC1'in R2'de bulunan PC2'yle iletişime geçebilmesi için yolu öğretmemiz gerekiyor
```
R1'e giriyoruz
ip route 192.168.2.0 (PC2'nin local ağı) 255.255.255.0 10.0.0.2 (R2'nin R1'e bakan bacağı)
R2'ye giriyoruz
ip route 192.168.1.0 (PC1'in local ağı) 255.255.255.0 10.0.0.1 (R1'nin R2'ye bakan bacağı)
```

#### Vlan Oluşturma
```
Switch'e giriyoruz
vlan 10 (vlan 10 oluşturur)
name Ankara (vlan 10 adına Ankara verir)
```
```
show vlan: ile vlanları ve bağlı portları görebiliriz
```
```
conf t
int ra fa0/1-10
switchport access vlan 10 (fa0/1 ile fa0/10 arasındaki portları vlan 10'a atar)
```

#### Cost Ayarlaması Yapmak

Cost'u az olan yol daha uzun olsa bile o yoldan paket iletimi sağlanır

```
int se0/0/0
ip ospf cost 100 (o portun cost'unu 100 olarak ayarlıyor)
```

#### Router Sıfırlama 
```
write memory (confları kaydediyoruz)
copy run start
reload
```

#### OSPF
OSPF yaptığımız zaman router'lara statik olarak yol göstermeye gerek duymayız(ip route...)
```
en
conf t
router ospf 1
network 192.168.2.0 0.0.0.255 area 0 (192.168.2.0 ile bir ağ var demektir eğer başka iç ağlar varsa onlarıda tek tek tanımlamamız lazım)
```

#### EtherChannel
Ether Channel birden fazla fiziksel portu mantıksal olarak tek bir porta indirme işlemine denir. Özellikle bant genişliğini arttırarak yüksek hızlı bağlantılar sağlamak için kullanırız. Böylelikle birden fazla portu birleştirerek darboğaz sorununu önlemiş oluruz. Örneğin bizim 2 ader Switch'imiz var SW2 ve SW3. Etherchannel olması için karşılıklı 2'şer portun trunk olarak ayarlanması lazım.

SW2'ye giriyoruz
```
en
conf t
int port-channel 1
switchport mode trunk
int fa0/3
channel-group 1 mode on
int fa0/4
channel-group 1 mode on
```
SW3'e giriyoruz
```
en
conf t
int port-channel 1
switchport mode trunk
int fa0/1
channel-group 1 mode on
int fa0/2
channel-group 1 mode on
```

#### Vlan'ı Trunk Yapmak

Switch'in bir portundan birden çok VLAN geçicekse o portu trunk moda almamız gerekir.

Switch'e giriyoruz
```
conf t
vlan 10 (vlan 10 oluşturur)
name Ankara (adını Ankara yapar)
vlan 20 (vlan 20 oluşturur)
name Istanbul (adını Istanbul yapar)
end
int fa0/1
switchport mode trunk (fa0/1 modunu trunk yapar)
```

#### Subınterface

2 local ağ bir router üzerinden geçiceği zaman, 2 adet default gateway'e ihtiyaç duyar. Bu durumlarda subınterface yaparız ve vlan ağlarına özel default gateway oluştururuz.

Router'a giriyoruz
```
conf t
int Gi1/0.20 192.168.4.1 255.255.255.0 (Gi1/0 portunun Vlan 20 için default gateway'i 192.168.4.1 olarak ayarlanır)
encapsulation dot1q
end
```
  
