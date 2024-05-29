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
- [Firewall](#Firewall)
  - [Konfigürasyon](#Konfigürasyon)
  - [ASA FW Managment Portundan Bağlama](#ASA-FW-Managment-Portundan-Bağlama)
- [NAT](#NAT)
  - [STATIK NAT](STATIK-NAT)
  - [DINAMIK NAT](DINAMIK-NAT)
  - [NAT PAT](NAT-PAT)
- [SDN](#SDN)
  
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
OSPF yaptığımız zaman router ve L3 switch'lere statik olarak yol göstermeye gerek duymayız(ip route...)
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
### Firewall

- 2 çeşit firewall vardır; Standart Firewall ve NextGen Firewall.
- Üzerinden geçen trafiği inceler, kurallara göre geçişine izin verir ya da yasaklar.
- L3 Gateway olarak kullanılabilir, ağ içindeki vlnarın bazı yerlere erişmelerini sağlar. Örnek: Satış biriminin Destek birimi belgelerine ulaşması gibi.
- Firewall 3 parçaya böler; iç, dış ve DMZ bacağıdır. DMZ bacağını dışarıya servis verilen durumlarda kullanılır.
- Mail sunucumuzu DMZ'ye koyabiliriz.
- NextGen Firewall ile iç ağdan belirli sitelere girmelerini engelleyebiliriz. Örnek: Ağdaki PC'ler youtube'a giremesin.
- İnternete çıkarken PC'lerin default gateway'i FW ayarlıyoruz.
- IP Kameralar için default gateway'e girmeye gerek yok ama IP Kamera'lara kim bakmış diye kontrol ediceksek o zaman Firewall'ı IP Kamera'nın default gateway olarak ayarlamamız gerekiyor.
- IPS: Dışardan içeriye sıkma işlemi olduğunda FW bunları drop edebilir.
- DLP: Veri sızma önleme, daha önce işaretlenmiş dosyaların FW üzerinden dışarı çıkışı olduğunda FW bunları yakalar.
- FW L7'ye kadar işlem yapabilir.
- FW'a yazılmış kurallardan en üstte bulunan kuraldan geçiyorsa alttaki kurallar dikkate alınmaz.
- FW ile internete çıkarken, cihazların default gateway'lerini FW IP adresini gireriz.

### Konfigürasyon 

Step1: Core cihaza vlanları oluşturuyoruz, eğer core cihazdaki vlanlarda ip var iptal ediyoruz. İptal ettikten sonra ```#show vlan```diyince vlanları görmemiz gerekiyor.

```no int vlan 5,10``` komutu ile iptal ediyoruz

Step2: Firewall'da channel-port oluşturuyoruz.

```
int channel-group 1
ip address 10.1.1.20 (vlan 5 IP'si tüm vlanlar için yapıyoruz bunu)
```

Step3: Firewall'ın portlarını channel-group'a alıyoruz

```
int gi1/1
channel-group mode on
int gi1/2
channel-group mode on
```

Step4: Channel-group'a isim veriyoruz

```
int channel-group 1
nameif  pc1 (pc1, inside, outside vs olabilir)
```

Step5: PC0 (Vlan5) 10.1.1.20'yi (Vlan5 default gateway) pingliyoruz

### ASA FW Managment Portundan Bağlama

Step1: Kabloyu FW'un portundan bağlıyoruz
```
int man 1/1
ip address 20.1.1.2
```

Step2: PC'mize giriyoruz ve default gateway'i "20.1.1.2" giriyoruz
Step3: IP adresine 20.1.1.1 giriyoruz
Step4: SSH ile ASA FW'ye bağlanmak için ```AAA authentication ssh concole local```komutunu giriyoruz ve artık SSH ile FW'ye bağlanabiliriz

> Troubleshooting

- Mesela bir problemimiz var ama FW'den mi kaynaklanıyor bilemiyoruz. En üste any'den gelenler any'ye gitsin kuralı yazarız eğer çalışıyorsa sorun FW'den kaynaklıdır ama eğer çalışmıyorsa FW'den kaynaklı değildir.
### NAT

"Nat" kısaltması "Network Address Translation" (Ağ Adres Çevirisi) anlamına gelir. Bu, bir bilgisayar ağında bulunan cihazların yerel IP adreslerini (Local IP) genellikle internete erişim sağlamak için kullanılan küresel IP adreslere dönüştürme işlemidir. Nat, ağdaki cihazların dış dünyaya erişebilmesini sağlar, ancak aynı zamanda iç ağ yapılarını dış dünya tarafından görünmez hale getirir, bu da ağ güvenliğini artırır.

### STATIK NAT

Statik NAT (Static Network Address Translation), bir iç ağdaki bir özel IP adresini (genellikle yerel bir ağda kullanılan IP adresi) sabit bir genel IP adresine dönüştüren bir NAT türüdür. Statik NAT, her zaman belirli bir özel IP adresini belirli bir genel IP adresine eşler. Statik NAT, genellikle ağdaki sunucuların hizmetlerini dış dünyaya sunmak için kullanılır. Örneğin, bir web sunucusu veya e-posta sunucusu gibi. Statik NAT, iç ağdaki bir sunucunun dış IP adresiyle sürekli olarak ilişkilendirilmesini sağlar, böylece dış kullanıcılar bu sunuculara erişebilir. Statik NAT kurulduğunda, bir iç özel IP adresi ile bir dizi genel (dış) IP adresi eşleştirilir. Bu eşleşme, ağ yöneticisi tarafından yapılandırılır ve sabit kalır. Bu nedenle, Statik NAT, belirli servislere (web sunucusu, e-posta sunucusu vb.) erişim sağlamak için ideal bir çözümdür ve IP adresleri sabit kalmalıdır.

```
#ip nat inside source static <DönüştürülecekKaynakIP> <DönüşücekIP>
#int gig0/0
#ip nat inside (gig0/0 portunun iç ağda olduğunu belirtiyoruz)
#int se0/0/0
#ip nat outside (se0/0/0 portunun dış ağ olduğunu belirtiyoruz)
```
NAT Yaptığımız cihaza paket gelirken source IP adresi 172.16.4.100'ken NAT yapılmış cihazdan çıkış yapınca source IP adresi 80.210.12.3 olur. Statik NAT yapında daha sonra NAT tablosunu sıfırlayarak temizleyemeyiz.

### DINAMIK NAT

Dinamik NAT, iç ağdaki cihazların dış dünyaya erişim sağlamasına olanak tanırken, aynı zamanda iç ağdaki her cihazın sabit bir genel IP adresine ihtiyacı olmadığı durumlarda kullanışlıdır. Dinamik NAT işlemi genellikle bir NAT havuzunu kullanır. Bu havuz, dış IP adresleri içerir ve bu IP adresleri dış dünya ile iletişim kurmak isteyen iç cihazlar için dinamik olarak atanır. Bir iç cihaz dış dünyaya bir istek gönderdiğinde, NAT cihazı bir dış IP adresini dinamik olarak atanır ve bu cihazın iletişimini sağlar. İletişim sona erdiğinde, bu dış IP adresi tekrar havuza geri döner ve başka bir cihaza atanabilir. Dinamik NAT, birçok iç cihazın aynı anda dış dünya ile iletişim kurmasına izin verirken, aynı anda gereksiz miktarda genel IP adresi kullanımını önler. Bu, IP adreslerinin verimli kullanımını sağlar. Dinamik NAT ayrıca, iç ağdaki cihazların dış IP adresleriyle ilişkilendirilmesini dinamik olarak yönettiği için, ağ yöneticileri için yönetimi kolaylaştırır.

```
#access-list 1 permit 172.16.0.0 0.0.255.255 (172.16.x.x ağından gelen IP'leri dönüştür)
#ip nat pool KAMPUS_POOL 80.210.12.3 80.210.12.6 (Gelen IP'leri bu aralıktaki IP'lerle eşleştir)
#netmask 255.255.255.248 (total 4 IP adresi olduğu için /29 oluyor ve .248 olur)
#ip nat inside source list 1 pool KAMPUS_POOL
```

```
#access-list 1 permit 172.16.0.0 0.0.255.255 
#ip nat inside source list 1 interface s0/0/0
```

### NAT PAT

NAT PAT (Network Address Translation Port Address Translation), birden çok özel (yerel) IP adresini tek bir genel (küresel) IP adresiyle dış dünyaya erişim sağlamak için kullanan bir NAT (Network Address Translation) türüdür. Ayrıca Overloading olarak da adlandırılır. 

PAT, IP adresleri ve port numaraları üzerinde dönüşüm yaparak çalışır. Özel IP adresleri, dış IP adresiyle birlikte kullanılan farklı port numaralarıyla birleştirilir. Bu sayede, aynı genel IP adresini paylaşan birden çok iç cihazın aynı anda dış ağa erişim sağlaması mümkün olur.

Örneğin, bir evde veya küçük bir işletme ağında, birden fazla cihazın aynı anda internete erişim sağlaması gerekebilir. PAT, bu cihazların her birini farklı port numaralarıyla birleştirerek, tek bir küresel IP adresini kullanarak tüm cihazların internete erişimini sağlar.

1024-65534 arasında port numarası alır, aynı anda yaklaşık 64bin cihaz internete çıkabilir. Belirli cihaz grubuna belirli port numarasını yazabiliyoruz. 

```
#access-list 1 permit 172.16.0.0 0.0.255.255
#ip nat inside source list 1 interface se0/0/0 overload
```

### SDN 

1. SDN Nedir?

SDN, Software-Defined Networking'in (Yazılım Tanımlı Ağ) kısaltmasıdır. Basitçe söylemek gerekirse, bu teknolojiyle ağları daha esnek ve verimli bir şekilde yönetebiliriz. Geleneksel ağlar, her cihazın ayrı ayrı yönetildiği bir yapıya sahiptir. Ancak SDN'de, ağın kontrolü merkezi bir noktada toplanır ve ağ cihazlarına ne yapması gerektiği söylenir. Bu, ağ yönetimini daha basit ve güçlü bir hale getirir.

2. Nasıl Çalışır?

SDN'de, ağ trafiğini yöneten bir kontrol noktası vardır. Bu kontrol noktası, ağdaki her şeyi izler ve yönetir. Örneğin, bir bilgisayar internete bağlandığında, kontrol noktası bunu fark eder ve bilgisayarın verilerini doğru yere yönlendirir. Bu, ağın daha akıllı ve daha hızlı çalışmasını sağlar.

3. Neden Önemlidir?

SDN, ağların daha esnek, güçlü ve ölçeklenebilir olmasını sağlar. Bu da daha iyi bir internet deneyimi ve daha az ağ sorunu demektir. Ayrıca, SDN sayesinde ağ yönetimi daha basit hale gelir ve işletmeler için maliyetleri azaltır.

4. Nasıl Uygulanır?

SDN'yi uygulamak için, bir SDN kontrol noktası ve uyumlu ağ cihazlarına ihtiyacınız vardır. Kontrol noktası genellikle bir yazılım veya donanım olarak sunulur. Daha sonra, ağ cihazlarını kontrol noktasına bağlarsınız ve ağ politikalarını ayarlarsınız. Son olarak, REST API'ler aracılığıyla ağ hizmetlerini yönetmek için gerekli yazılımları oluşturabilir veya kullanabilirsiniz.

```
Örnek: Şirket Kampüs Ağı İçin SDN Kullanımı

Senaryo:
Bir şirketin kampüs ağı, binlerce çalışanı, birden fazla bölümü ve bir dizi farklı cihazı kapsamaktadır. Bu ağ, ofis binalarını, veri merkezlerini ve farklı konumlardaki şubeleri içerebilir.

SDN Kontrol Noktası:
Şirketin kampüs ağı için bir SDN kontrol noktası, merkezi veri merkezinde bulunabilir. Bu kontrol noktası, tüm ağı izler ve yönetir. Ayrıca, farklı şubeler ve binalardaki alt SDN kontrol noktalarıyla iletişim kurabilir.

Bölüm ve Departman İzolasyonu:
SDN, farklı departmanların ve bölümlerin ağ trafiğini izole etmeye yardımcı olabilir. Örneğin, finans departmanının duyarlı verileri, diğer bölümlerin erişiminden daha sıkı bir şekilde izole edilebilir.

Misafir Ağı Yönetimi:
Şirket kampüsünde misafirler için ayrı bir ağ segmenti oluşturulabilir. SDN, bu misafir ağını otomatik olarak yönetebilir ve misafirlerin internet erişimini sağlayabilirken şirket içi kaynaklara erişimini sınırlayabilir.

Kalite Yönetimi ve Hizmet Düzeyi (QoS):
Özellikle sesli ve video konferans gibi önemli uygulamalar için bant genişliği tahsisini optimize etmek önemlidir. SDN, bu tür uygulamaların öncelikli trafiği olduğunu belirleyerek QoS politikalarını dinamik olarak uygulayabilir.

Hızlı Hizmet Dağıtımı ve Esneklik:
Yeni bir departman oluşturulduğunda veya yeni bir hizmet devreye alındığında, SDN bu değişikliklere hızlı bir şekilde uyum sağlayabilir. Yeni gereksinimlere göre ağ politikaları ve yapılandırmaları kolayca ayarlanabilir.

Güvenlik ve Tehtit Tespiti:
SDN, ağdaki anormal trafiği izleyerek ve analiz ederek tehditleri tespit edebilir. Örneğin, kötü amaçlı yazılımların veya ağ saldırılarının belirtilerini tespit edebilir ve buna göre önlemler alabilir.

Bu örnek, bir şirketin kampüs ağı için SDN'nin nasıl kullanılabileceğini göstermektedir. SDN, karmaşık bir ağ yapısını daha yönetilebilir hale getirebilir, ağ performansını artırabilir ve güvenliği geliştirebilir. 

OpenFlow; yazılım tanımlı ağların (SDN) uygulanmasını sağlayan bir iletişim protokolüdür. Bu protokol, ağdaki veri yollarını ve yönlendirme kararlarını merkezi bir kontrol cihazından alır ve yönetir. OpenFlow, geleneksel ağ donanımının ağ yönetimi ve kontrolünü merkezi bir yazılıma taşıyan bir yaklaşımı temsil eder.
```

İşlevsel olarak, OpenFlow, ağ cihazlarının (genellikle switch'lerin) işlevlerini ikiye ayırır:

Data Plane (Veri Yolu): Bu, gerçek veri iletiminin gerçekleştiği fiziksel ağ cihazlarını (switch'ler, router'lar) ifade eder. Veri Yolu, ağdaki paketlerin yönlendirilmesinden ve iletilmesinden sorumludur.

Control Plane (Kontrol Yolu): Bu, ağdaki akış tablosunu yöneten ve veri yolunu kontrol eden merkezi bir kontrol cihazını ifade eder. Kontrol Yolu, OpenFlow kontrolörü tarafından temsil edilir ve ağdaki trafiği izler, analiz eder ve gerektiğinde akışları yönlendirir.

OpenFlow protokolü, Kontrol Yolu ile Veri Yolu arasındaki iletişimi sağlar. Kontrol cihazı, ağdaki akışları tanımlar, ağ trafiğini analiz eder, politikaları uygular ve ağdaki değişiklikleri yönetir. Veri Yolu ise, Kontrol Yolu tarafından sağlanan yönlendirmeleri uygular ve ağ trafiğini yönlendirir.

OpenFlow protokolü, ağdaki esneklik, ölçeklenebilirlik ve merkezi yönetim gibi SDN'nin temel prensiplerini sağlar. Bu sayede, ağ yönetimi ve kontrolü daha basit ve daha etkili hale gelir. Ayrıca, OpenFlow'un açık standardı, farklı üreticilerin cihazlarının birlikte çalışmasını sağlar, böylece tedarikçi bağımsız SDN çözümleri geliştirilmesine imkan tanır.
