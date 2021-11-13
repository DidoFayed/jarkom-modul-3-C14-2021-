# Jarkom-Modul-3-C14-2021
Praktikum Jaringan Komputer Modul 3 - DHCP & Proxy Server
### Nama Anggota Kelompok:
1. 05111940000059 	Dido Fabian Fayed <br>
2. 05111940000074	Nur Ahmad Khatim <br>
3. 05111940000162	Ramadhan Arif Hardijansyah

## Soal 1
Membuat sebuah peta topology GNS3, dengan kriteria:
- **EniesLobby** akan dijadikan sebagai DNS Server.
- **Jipangu** sebagai DHCP Server.
- **Water7** akan dijadikan Proxy Server.

### Cara Pengerjaan
Dilakukan modifikasi terhadap topologi yang telah dibuat sebelumnya serta ditambahkan node baru, yaitu **Jipangu**, **Tottoland**, **Skypie**. Topologi ini terdiri dari tiga node ethernet switch, dengan:
- Switch1 terhubung dengan node client **Loguetown** dan **Alabasta**.
- Switch2 terhubung dengan node server **EniesLobby**, **Water7**, dan **Jipangu**.
- Switch3 terhubung dengan node client **Tottoland** dan **Skypie**.

![ssmodul3](https://github.com/DidoFayed/jarkom-modul-3-C14-2021-/blob/main/ssmodul3/1_1_Topology.png)

Lalu, dilakukan pengisian settingan network untuk masing-masing node dengan fitur `Edit network configuration` sebagai berikut.

- Foosha
	```
	auto eth0
	iface eth0 inet dhcp

	auto eth1
	iface eth1 inet static
		address 10.21.1.1
		netmask 255.255.255.0

	auto eth2
	iface eth2 inet static
		address 10.21.2.1
		netmask 255.255.255.0

	auto eth3
	iface eth3 inet static
		address 10.21.3.1
		netmask 255.255.255.0
	```
  
- Loguetown
	```
	auto eth0
	iface eth0 inet static
		address 10.21.1.2
		netmask 255.255.255.0
		gateway 10.21.1.1
	```
  
- Alabasta
	```
	auto eth0
	iface eth0 inet static
		address 10.21.1.3
		netmask 255.255.255.0
		gateway 10.21.1.1
	```
  
- EniesLobby
	```
	auto eth0
	iface eth0 inet static
		address 10.21.2.2
		netmask 255.255.255.0
		gateway 10.21.2.1
	```
  
- Water7
	```
	auto eth0
	iface eth0 inet static
		address 10.21.2.3
		netmask 255.255.255.0
		gateway 10.21.2.1
	```

- Jipangu
	```
	auto eth0
	iface eth0 inet static
		address 10.21.2.4
		netmask 255.255.255.0
		gateway 10.21.2.1
	```

- Skypie
	```
	auto eth0
	iface eth0 inet static
		address 10.21.3.2
		netmask 255.255.255.0
		gateway 10.21.3.1
	```

- TottoLand
	```
	auto eth0
	iface eth0 inet static
		address 10.21.3.3
		netmask 255.255.255.0
		gateway 10.21.3.1
	```
  
Restart semua node dan menjalankan command berikut pada router `Foosha`.
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.21.0.0/16
```
Dilanjutkan command berikut pada `Foosha` untuk melihat IP DNS:
```
cat /etc/resolv.conf
```
Muncul nameserver `192.168.122.1` yang akan digunakan pada konfigurasi selanjutnya. Agar node-node selain `Foosha` dapat mengakses internet, dijalankan command berikut dan menggunakan IP DNS tadi.
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
Testing semua node sudah terkoneksi dengan internet dengan `ping` ke google.com.

Berikutnya, diperlukan untuk menginstal tiga aplikasi pada tiga node server sebagai berikut:
1. Update package list pada **Jipangu**, **EniesLobby**, dan **Water7**.
	```
	apt-get update
	```

2. Install **isc-dhcp-server** pada DHCP Server **Jipangu**.
	```
	apt-get install isc-dhcp-server
	```

3. Install **bind9** pada server DNS Server **EniesLobby**.
	```
	apt-get install bind9
	```

4. Install **squid3** pada Proxy Server **Water7**.
	```
	apt-get install squid
	```


## Soal 2
Menjadikan **Foosha** sebagai DHCP Relay.

### Cara Pengerjaan
Install **isc-dhcp-relay** pada DHCP Relay **Foosha**.
```
apt-get install isc-dhcp-relay
```
Isikan IP Server dengan IP Jipangu sebagai DHCP Server yaitu `10.21.2.4` yang relay DHCP meneruskan request ke Server tersebut. Serta isikan interfaces dengan `eth1 eth2 eth3`. Kosongkan untuk selanjutnya.
 
![ssmodul3](https://github.com/DidoFayed/jarkom-modul-3-C14-2021-/blob/main/ssmodul3/1_4_a_DHCPRelaySetup.png) 

 
## Soal 3 
Semua client harus menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari `10.21.1.20` sampai `10.21.1.99` dan `10.21.1.150` sampai `10.21.1.169`.

### Cara Pengerjaan 
Edit file konfigurasi interface `/etc/default/isc-dhcp-server` pada DHCP Server Jipangu. Isikan dengan `eth1 eth2 eth3`.

![ssmodul3](https://github.com/DidoFayed/jarkom-modul-3-C14-2021-/blob/main/ssmodul3/3_1_InterfacesJipangu.png) 
 
Lalu edit file konfigurasi DHCP pada `/etc/dhcp/dhcpd.conf`. Tambahkan script berikut. 

``` 
subnet 10.21.2.4 netmask 255.255.255.0 { 
    range 10.21.1.20 10.21.1.99;
	range 10.21.1.99 10.21.1.169; 
    option routers 10.21.2.1; 
    option broadcast-address 10.21.2.255;
    option domain-name-servers 20.42.4.2;
    default-lease-time 600;
    max-lease-time 7200;
}
```

Lakukan restart service isc-dhcp-server.
```
service isc-dhcp-server restart
```
Namun, ternyata terdapat error sebagai berikut:

![ssmodul3](https://github.com/DidoFayed/jarkom-modul-3-C14-2021-/blob/main/ssmodul3/3_3_ErrorDHCPRestart.png)

## Soal 4
Client yang melalui Switch3 mendapatkan range IP dari 10.21.3.30 - 10.21.3.50.

### Cara Pengerjaan


## Kendala yang Dialami Selama Pengerjaan 
Kendala yang Dialami Selama Pengerjaan 
1. Beberapa teman yang menggunakan sistem operasi selain Linux mengalami kesulitan dalam mengerjakan, terdapat error ketika menjalankan GNS3, dan lain-lain. 
2. Kessulitan untuk mengimport file OVA pada virtualbox. Terdapat berbgai unsolved error pada saat create host manager
4. Terdapat error ketika mencoba untuk me-restart service isc-dhcp-server setelah mengedit file konfigurasi DHCP pada `/etc/dhcp/dhcpd.conf`, sehingga client yang melalui Switch1 tidak bisa mendapatkan konfigurasi IP dari DHCP Server.
