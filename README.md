# Cisco Packet Tracer

- [Sıfırdan Switch Router Konf](#Sıfırdan-Switch-Router-Konf)
- [Switch Enable Resetleme](#Switch-Enable-Resetleme)
- [Show Komutları](#Show-Komutları)
- [Switchlere İsim Vermek](#Switchlere-İsim-Vermek)
- [Router Aktif Etme](#Router-Aktif-Etme)
- [Router'a Yol Öğretme](#Router'a-Yol-Öğretme)
- [Vlan Oluşturma](#Vlan-Oluşturma)
- [Cost Ayarlaması Yapmak](#Cost-Ayarlaması-Yapmak)
- [Router Sıfırlama](#Router-Sıfırlama)
- [OSPF](#OSPF)
- [EtherChannel](#EtherChannel)
- [Vlan'ı Trunk Yapmak](#Vlan'ı-Trunk-Yapmak)
- [Subınterface](#Subınterface)
- [HSRP](#HSRP)
- [GLBP](#GLBP)
- [SDN](#SDN)
- [Tier 3 Demo Yapı](#Tier-3-Demo-Yapı)

## Sıfırdan Switch Router Konf

* Yeni bir router ya da switch'i konfigürasyon yaparken mutlaka takip etmemiz gereken adımlar vardır.
* ```#show memory``` ile cihazın donanımsal özellikleri görebiliriz, cihazda bir performans sorunu olursa buraya bakabiliriz.
* ```#config register 0x2142``` komutu ile cihaz konf'larını sıfırlarız. Cihaz açıldıktan sonra konfigürasyon yapmadan önce tekrar silinmemesi için ```#config register 0x2102``` komutunu girmemiz gerekir.
* ```#show run``` ile yapılan konfigürasyonları görebiliriz.

  * 1- Cihazımızı açıyoruz ve ilk "would you like to..." ile başlayan soruya no diyoruz 

  * 2- ```#show memory``` ile cihazın donanımsal özellikleri görebiliriz, cihazda bir performans sorunu olursa buraya bakabiliriz.
 
  * 3- Cihazımıza username tanımlamalıyız;
    ```
    # aaa new model
    #username ahmet secret 123 !ahmet kullanıcısı kriptolanmış-şifre 123
    #username ahmet password 123 !ahmet kullanıcısı şifre 123 
    #username ahmet privillage <0-15> ! bir yetki seviyesi verir 0-min 15-max
    #service password-encryption !password ile girilen şifreyi kriptolar
    #write memory
    ```
  * 4- SSH için ayarlama yapıyoruz;
    ```
    #ip ssh version 2
    #ip domain-name verify.com.tr
    #crypto key generaye rsa
    :1024
  * 5- Vlan1 Ip ekliyoruz;
    ```
    #int Vlan1
    #ip address 10.6.3.100 255.255.255.0
    #no shutdown
    ```
  * 6- Line vty ayarlıyoruz;
    ```
    #line vty 0 4
    #login local
    #transport input ssh
    ```
## Switch Enable Resetleme

* Önceden konfigürasyon yapılmış switch'in enable şifresi unutulduğunda switch'i resetlemek için yapmamız gereken işlemler vardır.

1- Switch'i power'dan çekerek kapatıyoruz.

2- Switch üzerinde bulunan "mode" tuşuna basarken, switch'in fişini takıyoruz ve switch'e bağlı PC'mizin ekranında "USB Console init, flash init" görünce parmağımızını çekiyoruz.

3- Komutları giriyoruz.
```
>flash-init
>dir flash:
>rename flash:config.text flash:config.old
>boot
```

4- Çıkan ekrana "no" diyoruz;
```
>rename flash: config.old flash:config.text
>copyflash:config.text system:running-config
```

### Show Komutları

```
show vlan: ile vlanları ve bağlı portları görebiliriz
show ip route: Router tablosunu gösterir
show ip ospf int brief: Cost bilgisi verir
show history: Geçmiş komutları listeler
show interface: Arayüzleri gösterir
```

### Switchlere İsim Vermek

```
conf t
hostname SW2 (switch'in ismini SW2 olarak değiştirir)
```
### Router Aktif Etme

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

### Router'a Yol Öğretme

Örneğin R1 ve R2 iki adet router'ımız var. R1'in iç ağındaki PC1'in R2'de bulunan PC2'yle iletişime geçebilmesi için yolu öğretmemiz gerekiyor
```
R1'e giriyoruz
ip route 192.168.2.0 (PC2'nin local ağı) 255.255.255.0 10.0.0.2 (R2'nin R1'e bakan bacağı)
R2'ye giriyoruz
ip route 192.168.1.0 (PC1'in local ağı) 255.255.255.0 10.0.0.1 (R1'nin R2'ye bakan bacağı)
```
- Komutun başına "no" ekleyerek iptal edebiliriz

### Vlan Oluşturma
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

### Cost Ayarlaması Yapmak

Cost'u az olan yol daha uzun olsa bile o yoldan paket iletimi sağlanır

```
int se0/0/0
ip ospf cost 100 (o portun cost'unu 100 olarak ayarlıyor)
```

### Router Sıfırlama 
```
write memory (confları kaydediyoruz)
copy run start
reload
```

### OSPF
OSPF yaptığımız zaman router'lara statik olarak yol göstermeye gerek duymayız(ip route...)
```
en
conf t
router ospf 1
network 192.168.2.0 0.0.0.255 area 0 (192.168.2.0 ile bir ağ var demektir eğer başka iç ağlar varsa onlarıda tek tek tanımlamamız lazım)
```

### EtherChannel
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

### Vlan'ı Trunk Yapmak

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

### Subınterface

2 local ağ bir router üzerinden geçiceği zaman, 2 adet default gateway'e ihtiyaç duyar. Bu durumlarda subınterface yaparız ve vlan ağlarına özel default gateway oluştururuz.

Router'a giriyoruz
```
conf t
int Gi1/0.20 192.168.4.1 255.255.255.0 (Gi1/0 portunun Vlan 20 için default gateway'i 192.168.4.1 olarak ayarlanır)
encapsulation dot1q
end
```

### HSRP

### GLBP

### SDN

SDN, yazılım tanımlı ağ demektir. Control layer ve data layer birbirinden ayrılmıştır. 

### Tier 3 Demo Yapı

1- Cihazları yerleştiriyoruz. L3 switch ile Router farkı, L3 switch'lerde interface için bir vlan oluşturup o vlana bir IP ataması yaparız ve interface'i o vlana atarız. Router'larda ise interface bacağıan IP ataması yaparız

2- Access switchlere vlan'ları oluşturuyoruz.
```
int f0/2 (PC'nin portları)
switchport access vlan 5 (Vlan'a göre değişir)
int vlan 5
name A
end
```
3- Distribution switch'lerin üzerinden hangi vlanlar geçicekse onları yazıyoruz. (Karşılıklı portları trunk yapmamız gerekiyor)
```
int g1/0/1
switchport mode trunk
switchport trunk allowed vlan 1,5,10 (o porttan sadece vlan1,vlan5 ve vlan10 geçsin)
```
4- Core switchler arasında etherchannel yapıyoruz. Bu sayede core switchler birbirinin ayakta olup olmadığını kontrol edebiliriz. C1 cihazının fa0/2 ve fa0/3 için yapıyoruz ve daha sonra C2 cihazı içinde yapmamız gerekiyor. 
```
conf t
int port-channel 1
switchport mode trunk
int fa0/3
channel-group  1 mode on
int fa0/2
channel-group 1 mode on
```
5- Core cihazlarımızda HSRP yapıyoruz. İki Core switch'imizde de bu işlemi yapıyoruz ve bu işlemi her vlan için yapmamız gerekiyor. Dist. ile Core1 arasında bir sorun olursa otomatik olarak Core2 yolu kullanılacak.
```
Core1 cihazımıza giriyoruz
int vlan 5
ip address 10.1.2.2 255.255.255.0 C1'imizin vlan5 için IP'si
standby 1 ip 10.1.2.1 Aktif olan Core kendi "10.1.2.2"'yi değil bu IP'yi kullanıcak 

Core2'ye giriyoruz
int vlan 5
ip address 10.1.2.3 255.255.255.0 C2'imizin vlan5 için IP'si
standby 1 ip 10.1.2.1
int vlan 10
ip address 10.1.3.3 255.255.255.0
standby 1 ip 10.1.3.1
```

Yapıda kullanılan ve geçişi olan vlanları Dist. ve Core switch'de de oluşturmamız gerekiyor.

6- Spannig-Tree oluşumunu engellemek için loop'daki tüm switchlere girip olan vlanlar için "pvst" aktif ediyoruz;
```
spanning-tree mode pvst
spanning-tree vlan 1,10,20,30
show spanning-tree summary
```




