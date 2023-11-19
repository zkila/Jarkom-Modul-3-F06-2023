# Jarkom-Modul-3-F06-2023

Kelompok F06:
- Arkana Bilal Imani / 5025211034
- ~~Azhar Abiyu Rasendriya H. / 5025211177~~

## Resource

- [Grimoire](https://docs.google.com/document/d/1cqIqL9wcrqEp37a-ASJLjwVFFkVMJhspBVT8EcWa904/edit?usp=sharing)
- [GNS3 Project](https://itsacid-my.sharepoint.com/:u:/g/personal/5025211034_student_its_ac_id/ERAXro4f4E5OgVTNV6d5ULABgQTvK-XPtwHNyA_HQeTe7w?e=I3KqbE)

## Daftar Isi


- [Daftar Isi](#daftar-isi)
- [Topologi](#topologi)
- [Konfig Node](#konfig)
- [No.0 - DNS](#soal-0)
- [No.1-5 - DHCP](#soal-1-5)
- [No.6 - PHP Worker](#soal-6)
- [No.7 - Weighted Round Robin](#soal-7)
- [No.8 - Algoritma Load Balancer Lain](#soal-8)
- [No.9 - Jumlah Worker](#soal-9)
- [No.10 - Autentikasi](#soal-10)
- [No.11 - Proxy Pass](#soal-11)
- [No.12 - IP Akses](#soal-12)
- [No.13 - Database](#soal-13)
- [No.14 - Deploy Laravel](#soal-14)
- [No.15-17 - Endpoint](#soal-15-17)
- [No.18 - Proxy Bind](#soal-18)
- [No.19 - PHP-FPM](#soal-19)
- [No.20 - Least Connection](#soal-20)

- [Kendala](#kendala)

 
### Topologi

![Alt text](topolog3.png)

### Konfig

- Aura (Router / DHCP Relay)
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.224.1.0
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.224.2.0
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.224.3.0
	netmask 255.255.255.0

auto eth4
iface eth4 inet static
	address 192.224.4.0
	netmask 255.255.255.0
```
- Switch 1
  - Himmel (DHCP Server)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.1.1
    	netmask 255.255.255.0
    	gateway 192.224.1.0
    ```
  - Heiter (DNS Server)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.1.2
    	netmask 255.255.255.0
    	gateway 192.224.1.0
    ```
- Switch 2
  - Denken (Database Server)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.2.1
    	netmask 255.255.255.0
    	gateway 192.224.2.0
    ```
  - Eisen (Load Balancer)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.2.2
    	netmask 255.255.255.0
    	gateway 192.224.2.0
    ```
- Switch 3
  - Lawine (Worker PHP 1)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.3.1
    	netmask 255.255.255.0
    	gateway 192.224.3.0
    ```
  - Linie (Worker PHP 2)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.3.2
    	netmask 255.255.255.0
    	gateway 192.224.3.0
    ```
  - Luger (Worker PHP 3)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.3.3
    	netmask 255.255.255.0
    	gateway 192.224.3.0
    ```
  - Revolte (Client)
    ```
    auto eth0
    iface eth0 inet dhcp
    ```
  - Reichter (Client)
    ```
    auto eth0
    iface eth0 inet dhcp
    ```
- Switch 4
  - Frieren (Worker Laravel 1)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.4.1
    	netmask 255.255.255.0
    	gateway 192.224.4.0
    ```
  - Flamme (Worker Laravel 2)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.4.2
    	netmask 255.255.255.0
    	gateway 192.224.4.0
    ```
  - Fern (Worker Laravel 3)
    ```
    auto eth0
    iface eth0 inet static
    	address 192.224.4.3
    	netmask 255.255.255.0
    	gateway 192.224.4.0
    ```
  - Sein (Client)
    ```
    auto eth0
    iface eth0 inet dhcp
    ```
  - Stark (Client)
    ```
    auto eth0
    iface eth0 inet dhcp
    ```
## Soal-0
- [Daftar Isi](#daftar-isi)

Setelah mengalahkan Demon King, perjalanan berlanjut. Kali ini, kalian diminta untuk melakukan register domain berupa `riegel.canyon.yyy.com` untuk worker Laravel dan `granz.channel.yyy.com` untuk worker PHP mengarah pada worker yang memiliki IP [prefix IP].x.1.  

Yang diperlukan untuk menyelesaikan soal ini adalah melakukan command `iptables` di Aura sebagai berikut:
```shell
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.224.0.0/16
```
Lalu install DNS di Heiter karena Heiter adalah DNS server di topologi ini dengan skrip berikut:
```shell
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
service bind9 start
```
Setelah terinstall, tambahkan konfigurasi untuk `riegel.canyon.yyy.com` dan `granz.channel.yyy.com` yang mengarah ke `192.224.x.1` dengan skrip berikut:
```shell
cho 'zone "canyon.f06.com" {
        type master;
        file "/etc/bind/jarkom/canyon.f06.com";
};' > /etc/bind/named.conf.local

mkdir /etc/bind/jarkom

cp /etc/bind/db.local /etc/bind/jarkom/canyon.f06.com

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     canyon.f06.com. root.canyon.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      canyon.f06.com.
@       IN      A       192.224.2.2 ; IP Eisen
riegel  IN      A       192.224.4.1 ; IP Frieren
' > /etc/bind/jarkom/canyon.f06.com


echo 'zone "granz.channel.f06.com" {
        type master;
        file "/etc/bind/jarkom/granz.channel.f06.com";
};' >> /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/jarkom/granz.channel.f06.com

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     granz.channel.f06.com. root.granz.channel.f06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      granz.channel.f06.com.
@       IN      A       192.224.3.1 ; IP Lawine
www     IN      CNAME   granz.channel.f06.com.
' > /etc/bind/jarkom/granz.channel.f06.com

echo '
options {
        directory "/var/cache/bind";

         forwarders {
                192.168.122.1;
         };

        //dnssec-validation auto;
        allow-query{any;};
        listen-on-v6 { any; };
};
' > /etc/bind/named.conf.options

service bind9 restart
```
Untuk domain `riegel.canyon.yyy.com`, riegel dijadikan subdomain dari `canyon.yyy.com` memisahkan cara mengakses worker Frieren secara langsung dengan mengakses lewat load balancer Eisen dengan default domain tersebut. Dalam konfigurasi DNS ini juga ada forwarder yang mengarah ke IP `192.168.122.1` yang menghubungkan DNS ini ke internet untuk keperluan install dan update pada soal soal selanjutnya.  

Setelah konfigurasi, DNS server dapata dites dengan skrip berikut di node manapun (kecuali client karena DHCP belum dikonfigurasi).
```shell
echo nameserver 192.224.1.2 > /etc/resolv.conf
ping riegel.canyon.f06.com -c 3
ping granz.channel.f06.com -c 3
```
  
Berikut adalah hasil tesnya: 
![Alt text](images/nomer0a.png)

## Soal-1-5
- [Daftar Isi](#daftar-isi)

Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.  

Kemudian, karena masih banyak spell yang harus dikumpulkan, bantulah para petualang untuk memenuhi kriteria berikut.:  
- Semua CLIENT harus menggunakan konfigurasi dari DHCP Server.
- Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80 
- Client yang melalui Switch4 mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168 
- Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut
- Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit

Yang diperlukan untuk melakukan konfigurasi DHCP adalah menginstall DHCP di Himmel karena Himmel sudah ditentukan sebagai DHCP server dengan skrip berikut:
```shell
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y
apt-get --reinstall install rsyslog

service rsyslog start

dhcpd --version
```
Service rsyslog digunakan untuk membuat log file karena segala error di konfigurasi DHCP akan masuk ke situ.  

Setelah melakukan proses instalasi tersebut, DHCP bisa dikonfigurasikan dengan skrip berikut:
```shell
echo '
subnet 192.224.1.0 netmask 255.255.255.0{
}

subnet 192.224.2.0 netmask 255.255.255.0{
}

subnet 192.224.3.0 netmask 255.255.255.0 {
    range 192.224.3.16 192.224.3.32;
    range 192.224.3.64 192.224.3.80;
    option routers 192.224.3.0;
    option broadcast-address 192.224.3.255;
    option domain-name-servers 192.224.1.2;
    default-lease-time 180;
    max-lease-time 5760;
}
subnet 192.224.4.0 netmask 255.255.255.0 {
    range 192.224.4.12 192.224.4.20;
    range 192.224.4.160 192.224.4.168;
    option routers 192.224.4.0;
    option broadcast-address 192.224.4.255;
    option domain-name-servers 192.224.1.2;
    default-lease-time 780;
    max-lease-time 5760;
}' > /etc/dhcp/dhcpd.conf

echo '
INTERFACESv4="eth0"
' > /etc/default/isc-dhcp-server

service isc-dhcp-server restart
```
Di konfigurasi tersebut, subnet 192.224.3.0 diset dengan `range 192.224.3.16 - 192.224.3.32` dan `range 192.224.3.64 - 192.224.3.80` untuk memenuhi soal nomor 2.
Untuk subnet 192.224.4.0, diberikan `range 192.224.4.12 - 192.224.20` dan `range 192.224.4.160 - 192.224.4.168` untuk memenuhi soal nomor 3.  

Untuk nomor 4, karena `option domain-name-servers` diset untuk mengarah ke DNS server, yaitu Heiter di IP `192.224.1.2`, client (Sein, Stark, Revolte, Reichter) secara otomatis menggunakan DNS server tersebut. Karena DNS server di Heiter sudah dikonfigurasikan dengan forwarder, maka client dapat mengakses web server yang sudah di list di DNS (Riegel dan Granz) dan tersambung ke internet juga melalui IP di forwarder.  

Berikut adalah bukti sudah bisa terhubung dengan internet:
![Alt text](images/nomer1a.png)

Untuk nomor 5, untuk mengatur lama waktu IP dipinjamkan ke client secara default dan maksimal, dapat melalui `default-lease-time` dan `max-lease-time` yang dikonfigurasikan sesuai dengan permintaan soal. Karena angka yang dimasukkan ke dalam parameter tersebut bersifat detik, maka angka menit harus dikali 60 terlebih dahulu.  
  
Setelah dilakukan konfigurasi di Himmel sebagai DHCP server, diperlukan juga konfigurasi di Aura sebagai DHCP relay dengan skrip berikut:  
```shell
apt-get update
apt-get install isc-dhcp-relay -y

echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
service isc-dhcp-relay start
service isc-dhcp-relay restart
```
Saat menginstall DHCP relay, akan diprompt untuk konfigurasi IP DHCP server, interface yang akan tersambung, dan modifier lain.  
- IP yang digunakan untuk DHCP server adalah IP Himmel di `192.224.1.1`.  
- Interface yang akan tersambung dengan DHCP relay adalah `eth1, eth2, eth3 dan eth4` sesuai dengan konfigurasi network di aura untuk menyambungkan semua subnet karena konfigurasi DHCP server di Himmel juga mendaftarkan setiap subnet. Maka dari itu list interface tersebut harus sama dengan konfigurasi relay supaya sistem DHCP ini dapat bekerja.  
- Modifier lain bisa dibiarkan kosong.  
  
Berikut adalah prompt konfigurasinya:
  
![Alt text](images/nomer1b.png)
  
Untuk melakukan tes DHCP sudah bekerja atau belum, yang dilakukan adalah stop dan start client, lalu membuka web console bagi client tersebut untuk melihat bahwa client sudah dipinjamkan IP sesuai dengan range yang sudah ditentukan. Proses peminjaman ini seharusnya sudah terlihat saat web console dibuka. Apabila tidak terlihat, maka bisa dilihat menggunakan command `ip a`.  
Berikut adalah contoh gambarnya:  
  
![Alt text](images/nomer1c.png)
![Alt text](images/nomer1d.png)

## Soal 6
- [Daftar Isi](#daftar-isi)

Pada masing-masing worker PHP, lakukan konfigurasi virtual host untuk website [berikut](https://drive.google.com/file/d/1ViSkRq7SmwZgdK64eRbr5Fm1EGCTPrU1/view?usp=sharing) dengan menggunakan php 7.3.  

Yang perlu dilakukan pertama-tama adalah menginstall semua hal yang dibutuhkan untuk deploy website tersebut, bisa menggunakan skrip berikut:
```shell
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install php7.3 -y
php -v
apt-get install nginx -y
nginx -v
service nginx start
apt-get install php-fpm -y
service php7.3-fpm start
service php7.3-fpm restart
apt-get install wget unzip -y
apt-get install htop -y

# wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1ViSkRq7SmwZgdK64eRbr5Fm1EGCTPrU1' -O granz.channel.f06.com.zip
unzip granz.channel.f06.com.zip 
mkdir /var/www/granz
mv -v modul-3/* /var/www/granz/
rm -rf modul-3/
```  
Selain PHP versi 7.3 dan Nginx yang diinstall, beberapa service lain seperti php7.3-fpm, wget, unzip, dan htop yang diperlukan untuk nomor nomor selanjutnya juga sekalian diinstall di skrip tersebut. 

Service wget dan unzip digunakan untuk mendownload file yang diperlukan untuk websitenya seperti index.php dan lain lain, dan setelah di download, file .zip tersebut diunzip dengan unzip dan dipindahkan ke folder `/var/www/granz`.  
  
Setelah proses install selesai, virtual host untuk website granz bisa dikonfigurasikan dengan skrip berikut:  
```shell
rm /var/www/html/index.html
rm -rf /etc/nginx/sites-enabled/default

touch /etc/nginx/sites-available/granz
echo '
server {

        listen 80;

        root /var/www/granz;

        index index.php index.html index.htm;
        server_name granz.channel.f06.com;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
        }
        location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/granz_error.log;
        access_log /var/log/nginx/granz_access.log;
}
' > /etc/nginx/sites-available/granz

ln -s /etc/nginx/sites-available/granz  /etc/nginx/sites-enabled
service nginx restart
nginx -t
```
Untuk menghindari terhubung ke default page Nginx, file index.html dan konfigurasi virtual host default dihapus.  

Lalu file granz dibuat sebagai konfigurasi virtual host dengan root mengarah ke /var/www/granz dan file tersebut akan dibuatkan link ke folder /etc/nginx/sites-enabled untuk mengaktifkan konfigurasi virtual hostnya. Hal ini dilakukan tiga kali di worker PHP (Lawine, Linie, Luger).  

Setelah itu, perlu dilakukan konfigurasi load balancer juga untuk menghandle pembagian kerja worker PHP tersebut, dengan menggunakan skrip berikut:  
```shell
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install nginx -y
apt-get install apache2-utils -y
apt-get install htop -y

service nginx start
service nginx status
```
Skrip tersebut juga menginstall apache2-utils dan htop yang akan berguna untuk nomor nomor selanjutnya.  
```shell

touch /etc/nginx/sites-available/lb-eisen

echo '
upstream myweb  {
	server 192.224.3.1; 
	server 192.224.3.2; 
  server 192.224.3.3; 
}

server {
	listen 80;
	server_name granz.channel.f06.com;

	location / {
	proxy_pass http://myweb;
	}
}' > /etc/nginx/sites-available/lb-eisen

ln -s /etc/nginx/sites-available/lb-eisen /etc/nginx/sites-enabled

service nginx start
service nginx restart
rm /var/www/html/index.html
rm -rf /etc/nginx/sites-enabled/default

nginx -t
```
Hal yang pertama dilakukan adalah membuat file konfigurasi di folder /etc/nginx/sites-available bernama lb-eisen. lalu dimasukkan konfigurasi tersebut dengan semua IP worker PHP (Lawine, Linie, dan Luger) menjadi worker. server_name juga diset menjadi granz.channel.f06.com agar saat mengetes menggunakan lynx atau ab, tidak perlu menggunakan IP Eisen.  

Berikut adalah hasil lynx load balancer dengan worker PHP: 
![Alt text](images/nomer6a.png)

![Alt text](images/nomer6b.png)

![Alt text](images/nomer6c.png)

## Soal-7
- [Daftar Isi](#daftar-isi)

Kepala suku dari Bredt Region memberikan resource server sebagai berikut:
- Lawine, 4GB, 2vCPU, dan 80 GB SSD.
- Linie, 2GB, 2vCPU, dan 50 GB SSD.
- Lugner 1GB, 1vCPU, dan 25 GB SSD.

Aturlah agar Eisen dapat bekerja dengan maksimal, lalu lakukan testing dengan 1000 request dan 100 request/second. 

Untuk melakukan testing pada worker PHP dan load balancer di Eisen, konfigurasi load balancer perlu ditambahkan weight terlebih dahulu seperti pada konfigurasi berikut:
```shell
# Config 1
	server 192.224.3.1 weight=4; 
	server 192.224.3.2 weight=2; 
  server 192.224.3.3 weight=1;
# Config 2
	server 192.224.3.1 weight=80; 
	server 192.224.3.2 weight=50; 
  server 192.224.3.3 weight=25;
# Config 3
	server 192.224.3.1 weight=2; 
	server 192.224.3.2 weight=2; 
  server 192.224.3.3 weight=1;
```
Algoritma yang digunakan berubah menjadi weighted round robin karena terdapat parameter weight untuk setiap worker di konfigurasi load balancer di Eisen.

Untuk melakukan testing, bisa menggunakan command berikut:
```shell
ab -n 1000 -c 100 -g out.data http://granz.channel.f06.com/
```
Berikut adalah hasil testing dengan urutan sesuai dengan config diatas:

![Alt text](images/nomer7a.png)

![Alt text](images/nomer7b.png)

![Alt text](images/nomer7c.png)

## Soal-8
- [Daftar Isi](#daftar-isi)

Karena diminta untuk menuliskan grimoire, buatlah analisis hasil testing dengan 200 request dan 10 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut:
- Nama Algoritma Load Balancer
- Report hasil testing pada Apache Benchmark
- Grafik request per second untuk masing masing algoritma. 
- Analisis 

Untuk menyelesaikan soal ini, yang diganti bukan weight untuk algoritma round robin lagi, tetapi tipe algoritma yang dipakai di load balancer di Eisen.

Berikut adalah algoritma yang akan dipakai untuk testing:

```shell
# Round Robin

# Least Connection
least_conn;
# IP Hash
ip_hash;
# Generic Hash
hash $request_uri consistent;
```
Dapat dilihat bahwa round robin tidak memiliki kode, karena algoritma round robin secara default digunakan di konfigurasi Nginx.

Menggunakan command ab berikut, dilakukan testing dengan urutan sesuai dengan konfig diatas:

```shell
ab -n 200 -c 10 -g out.data http://granz.channel.f06.com/
```

Berikut adalah hasil ab:

![Alt text](images/nomer8a.png)

![Alt text](images/nomer8b.png)

![Alt text](images/nomer8c.png)

![Alt text](images/nomer8d.png)

## Soal-9
- [Daftar Isi](#daftar-isi)

Dengan menggunakan algoritma Round Robin, lakukan testing dengan menggunakan 3 worker, 2 worker, dan 1 worker sebanyak 100 request dengan 10 request/second, kemudian tambahkan grafiknya pada grimoire.

Untuk menyelesaikan soal ini, yang perlu dilakukan adalah mengetes dengan ab jika ada 3 worker yang bekerja, 2 worker yang bekerja dan 1 worker yang bekerja.

Hal ini bisa dilakukan dengan memberhentikan nginx di worker dengan command:
```shell
service nginx stop
```

Berikut adalah command ab yang dipakai dengan hasil ab tersebut:
```shell
ab -n 100 -c 10 -g out.data http://granz.channel.f06.com/
```
![Alt text](images/nomer9a.png)

![Alt text](images/nomer9b.png)

![Alt text](images/nomer9c.png)

## Soal-10
- [Daftar Isi](#daftar-isi)

Selanjutnya coba tambahkan konfigurasi autentikasi di LB dengan dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/.

Untuk menyelesaikan soal ini, diperlukan untuk membuat file passwd dengan command `htpasswd`. Selain itu juga diperlukan untuk mengubah file virtual host untuk membutuhkan autentikasi untuk diakses. Semua itu bisa dilakukan dengan skrip berikut:

```shell
htpasswd -c /etc/nginx/rahasisakita netics

echo '
upstream myweb  {
	server 192.224.3.1 weight=4; 
	server 192.224.3.2 weight=2; 
    server 192.224.3.3 weight=1; 
}

server {
	listen 80;
	server_name granz.channel.f06.com;

        location / {
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/rahasisakita;
        proxy_pass http://myweb;
        }

        location ~ /\.ht {
            deny all;
        }
error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}' > /etc/nginx/sites-available/lb-eisen

service nginx restart
```

Saat diprompt untuk membuat password, langsung saja masukkan `ajkf06`.

![Alt text](images/nomer10a.png)

Lalu bisa di tes di client menggunakan command lynx berikut:
```lynx 192.224.2.2```

![Alt text](images/nomer10b.png)

![Alt text](images/nomer10c.png)

Setelah terkonfirmasi bahwa granz membutuhkan password, langsung saja tes dengan ab dengan command berikut:
```shell
ab -A netics:ajkf06 -n 1000 -c 100 -g out.data http://192.224.2.2/
ab -n 1000 -c 100 -g out.data http://192.224.2.2/
```

Didapatkan hasil sebagai berikut:

![Alt text](images/nomer10d.png)

![Alt text](images/nomer10e.png)
Bisa dilihat bahwa ab yang tidak menggunakan autentikasi, semua requestnya menjadi request Non-2xx karena tidak bisa mengakses konten websitenya dan dihentikan di proses autentikasi.

## Soal-11
- [Daftar Isi](#daftar-isi)

Lalu buat untuk setiap request yang mengandung /its akan di proxy passing menuju halaman https://www.its.ac.id. Hint: (proxy_pass).

Untuk menyelesaikan soal ini, langsung saja gunakan skrip ini untuk mengubah konfigurasi load balancer di Eisen.
```shell
echo '# Default menggunakan Round Robin
upstream myweb  {
	server 192.224.3.1 weight=4; 
	server 192.224.3.2 weight=2; 
    server 192.224.3.3 weight=1; 
}

server {
	listen 80;
	server_name granz.channel.f06.com;

        location / {
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/rahasisakita;
        proxy_pass http://myweb;
        }

        location ~ /\.ht {
            deny all;
        }

        location /its {
        proxy_pass https://www.its.ac.id;
        }

error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}' > /etc/nginx/sites-available/lb-eisen

service nginx restart
```

Yang ditambahkan adalah `location` block baru yang melihat apakah ada requests dengan tambahan `/its`dan jika ada, maka akan diredirect dengan `proxy_pass` ke https://www.its.ac.id.

Berikut adalah command lynx yang digunakan untuk mengetes dan hasilnya:
```shell
lynx 192.224.2.2/its
```


![Alt text](images/nomer11a.png)

![Alt text](images/nomer11b.png)

## Soal-12
- [Daftar Isi](#daftar-isi)

Selanjutnya LB ini hanya boleh diakses oleh client dengan IP [Prefix IP].3.69, [Prefix IP].3.70, [Prefix IP].4.167, dan [Prefix IP].4.168.  Hint: (fixed in dulu clinetnya).

Untuk menyelesaikan soal ini, yang perlu diubah hanyalah konfigurasi load balancer di Eisen, yang bisa dilakukan dengan skrip berikut:

```shell
echo '
upstream myweb  {
	server 192.224.3.1 weight=4; 
	server 192.224.3.2 weight=2; 
        server 192.224.3.3 weight=1; 
}

server {
	listen 80;
	server_name granz.channel.f06.com;

        location / {
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/rahasisakita;
        proxy_pass http://myweb;
        allow 192.224.3.69;
        allow 192.224.3.70;
        allow 192.224.4.167;
        allow 192.224.4.168;
        deny all;
        }

        location ~ /\.ht {
        deny all;
        }

        location /its {
        proxy_pass https://www.its.ac.id;
        }

error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}' > /etc/nginx/sites-available/lb-eisen

service nginx
```
Penambahannya adalah beberapa `allow` yang menginjinkan client dengan IP tertentu yang ditentukan soal untuk mengakses dan `deny all` yang melarang client dengan IP lain untuk mengakses granz.

Untuk melakukan tes, salah satu client bisa diubah untuk memiliki IP yang diijinkan untuk mengakses dengan mengganti `network configuration` di `GNS3`.

![Alt text](images/nomer12a.png)

Berikut adalah hasilnya saat mencoba menggunakan lynx dengan command berikut:
```shell
lynx 192.224.2.2
```

- Gagal:

![Alt text](images/nomer12b.png)

- Sukses: 

![Alt text](images/nomer12c.png)

## Intro Laravel
Karena para petualang kehabisan uang, mereka kembali bekerja untuk mengatur riegel.canyon.yyy.com.
## Soal-13
- [Daftar Isi](#daftar-isi)

Semua data yang diperlukan, diatur pada Denken dan harus dapat diakses oleh Frieren, Flamme, dan Fern.

Untuk menyelesaikan soal ini, diperlukan konfigurasi Database server yang akan menggunakan MariaDB. Bisa dilakukan dengan skrip berikut:
```shell
apt-get update
apt-get install mariadb-server -y
service mysql start

echo '
[client-server]

# Import all .cnf files from configuration directory
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/

[mysqld]
skip-networking=0
skip-bind-address
' > /etc/mysql/my.cnf

service mysql restart
```
Potongan kode yang dimasukkan kedalam `my.cnf` berguna untuk menginzinkan worker lain untuk mengakses database menggunakan MariaDB client.

Setelah proses install selesai, bisa lanjut ke step step berikut untuk melakukan setup database.
- Pertama adalah menggunakan `mysql -u root -p` untuk masuk ke console SQL MariaDB.
- Kedua adalah membuat user baru dan database baru dengan beberapa kode berikut:
```sql
CREATE USER 'kelompokf06'@'%' IDENTIFIED BY '';
CREATE USER 'kelompokf06'@'localhost' IDENTIFIED BY '';
CREATE DATABASE dbkelompokf06;
GRANT ALL PRIVILEGES ON *.* TO 'kelompokf06'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'kelompokf06'@'localhost';
FLUSH PRIVILEGES;
```
- Ketiga adalah mengetes apakah db tersebut terbuat dengan `SHOW DATABASES;`
- Hasilnya adalah sebagai berikut:

![Alt text](images/nomer13a.png)
- Setelah itu bisa menggunakan `\q` untuk keluar dari console SQL.

Untuk menginstall MariaDB client di semua worker digunakan skrip berikut:
```shell
apt-get update
apt-get install mariadb-client -y


# to test if connected
mariadb --host=192.224.2.1 --port=3306 --user=kelompokf06 --password
```
Dan dapat menggunakan command `SHOW DATABASES;` tadi juga untuk memastikan sudah terhubung.

![Alt text](images/nomer13b.png)

## Soal-14
- [Daftar Isi](#daftar-isi)

Frieren, Flamme, dan Fern memiliki Riegel Channel sesuai dengan quest guide berikut. Jangan lupa melakukan instalasi PHP8.0 dan Composer.

Untuk menyelesaikan soal ini, diperlukan untuk menginstall dan mengkonfigurasikan banyak hal sesuai dengan modul Reverse Proxy. Dapat dilakukan dengan skrip berikut:
```shell
apt-get update
apt-get install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'

apt-get update
apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y
php --version

wget https://getcomposer.org/download/2.0.13/composer.phar
chmod +x composer.phar
mv composer.phar /usr/bin/composer

apt-get install nginx -y
apt-get install git -y

cd /var/www/
git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git
cd laravel-praktikum-jarkom/
composer install
composer update

cp .env.example .env
echo '
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=192.224.2.1
DB_PORT=3306
DB_DATABASE=dbkelompokf06
DB_USERNAME=kelompokf06
DB_PASSWORD=

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MEMCACHED_HOST=127.0.0.1

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https
PUSHER_APP_CLUSTER=mt1

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
' > /var/www/laravel-praktikum-jarkom/.env

# 2 Command dibawah hanya perlu dilakukan di 1 worker
#php artisan migrate:fresh
#php artisan db:seed --class=AiringsTableSeeder
# 2 Command diatas hanya perlu dilakukan di 1 worker

php artisan jwt:secret
php artisan key:generate
```
Apabila skrip tersebut dijalankan harusnya terlihat seperti berikut:

![Alt text](images/nomer14a.png)

![Alt text](images/nomer14b.png)

Setelah melakukan proses install Laravel tersebut, bisa dijalankan skrip berikut untuk menambahkan virtual host dengan port yang diubah sesuai dengan ketentuan soal (Frieren:8001 Flamme:8002 Fern:8003).
```shell
rm /var/www/html/index.html
rm -rf /etc/nginx/sites-enabled/default

echo '
server {

    listen 800x;

    root /var/www/laravel-praktikum-jarkom/public;

    index index.php index.html index.htm;
    server_name _;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }
    location ~ /\.ht {
            deny all;
    }

    error_log /var/log/nginx/implementasi_error.log;
    access_log /var/log/nginx/implementasi_access.log;
}'> /etc/nginx/sites-available/riegel


ln -s /etc/nginx/sites-available/riegel /etc/nginx/sites-enabled/

chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage

service php8.0-fpm start

service nginx restart
```

Setelah itu langsung bisa dites menggunakan command lynx berikut di client dengan hasil seperti berikut:
```shell
lynx riegel.canyon.f06.com:8001
```
![Alt text](images/nomer14c.png)

![Alt text](images/nomer14d.png)

## Soal-15-17
- [Daftar Isi](#daftar-isi)

Riegel Channel memiliki beberapa endpoint yang harus ditesting sebanyak 100 request dengan 10 request/second. Tambahkan response dan hasil testing pada grimoire.

- POST /auth/register 
- POST /auth/login 
- GET /me 

Untuk 3 soal ini, diperlukan konfigurasi load balancer yang menghandle 3 worker laravel. Hal itu bisa dilakukan dengan menjalankan skrip berikut:
```shell
echo '
upstream myweb  {
        server 192.224.3.1 weight=4; 
        server 192.224.3.2 weight=2; 
        server 192.224.3.3 weight=1; 
}
upstream laravel {
        server 192.224.4.1:8001;
        server 192.224.4.2:8002;
        server 192.224.4.3:8003;
}
server {
        listen 80;
        server_name granz.channel.f06.com;

        location / {
        auth_basic "Rahasia Uy";
        auth_basic_user_file /etc/nginx/rahasisakita;
        proxy_pass http://myweb;
        }

        location ~ /\.ht {
            deny all;
        }

        location /its {
        proxy_pass https://www.its.ac.id;
        }

error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}

server {
        listen 81;
        server_name canyon.f06.com;

        location / {
        proxy_pass http://laravel; 
        }
}
' > /etc/nginx/sites-available/lb-eisen

service nginx restart
```
Setelah itu, endpoint dapat dites dengan beberapa command `curl` berikut:
```shell
# test at worker (Frieren, Flamme, Fern)

curl -X POST "http://localhost:8001/api/auth/register" -d "username=kelompokf06&password=passwordf06"

curl -X POST "http://localhost:8001/api/auth/login" -d "username=kelompokf06&password=passwordf06"

curl -X GET "http://localhost:8001/api/me" -H "Authorization: Bearer (token here)"

# test at client (Sein)

curl -X POST "http://canyon.f06.com:81/api/auth/register" -d "username=kelompokf06&password=passwordf06"

curl -X POST "http://canyon.f06.com:81/api/auth/login" -d "username=kelompokf06&password=passwordf06"

curl -X GET "http://canyon.f06.com:81/api/me" -H "Authorization: Bearer (token here)"
```
Berikut adalah hasil register, login, dan /me:

![Alt text](images/nomer15a.png)

![Alt text](images/nomer15b.png)

![Alt text](images/nomer15c.png)

Setelah itu, untuk melakukan testing dengan ab, bisa digunakan beberapa command ab sebagai berikut:

```shell
# 15ab.sh
ab -n 100 -c 10 -p ~/postreg.json -T application/json http://canyon.f06.com:81/api/auth/register/
# 16ab.sh
ab -n 100 -c 10 -p ~/postreg.json -T application/json http://canyon.f06.com:81/api/auth/login/
# 17ab.sh
ab -n 100 -c 10 -H "Authorization: Bearer (token here)" -r -k "http://canyon.f06.com:81/api/me"
```
Command ab ini membutuhkan file `.json` yang dapat dengan mudah dibuat dengan skrip berikut:
```shell
touch postreg.json
echo '
{
    "username": "kelompokf06",
    "password": "passwordf06"
}
' > postreg.json
```

Berikut adalah hasil dari testing ab yang dilakukan:

![Alt text](images/nomer15d.png)

![Alt text](images/nomer15e.png)

![Alt text](images/nomer15f.png)



## Soal-18
- [Daftar Isi](#daftar-isi)

Untuk memastikan ketiganya bekerja sama secara adil untuk mengatur Riegel Channel maka implementasikan Proxy Bind pada Eisen untuk mengaitkan IP dari Frieren, Flamme, dan Fern. 

Untuk menyelesaikan soal ini, hanya perlu menambahkan sedikit konfigurasi tambahan pada load balancer di Eisen yang dapat dijalankan dengan skrip berikut:

```shell
echo '
upstream myweb  {
        server 192.224.3.1 weight=4; 
        server 192.224.3.2 weight=2; 
        server 192.224.3.3 weight=1; 
}
upstream laravel {
        server 192.224.4.1:8001;
        server 192.224.4.2:8002;
        server 192.224.4.3:8003;
}
server {
        listen 80;
        server_name granz.channel.f06.com;

        location / {
        auth_basic "Rahasia Uy";
        auth_basic_user_file /etc/nginx/rahasisakita;
        proxy_pass http://myweb;
        }

        location ~ /\.ht {
            deny all;
        }

        location /its {
        proxy_pass https://www.its.ac.id;
        }

error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}

server {
        listen 81;
        server_name canyon.f06.com;

        location / {
        proxy_pass http://laravel; 
        }

        location /frieren/{
        proxy_bind 192.224.2.2;
        proxy_pass http://192.224.4.1:8001/index.php;
        }

        location /flamme/{
        proxy_bind 192.224.2.2;
        proxy_pass http://192.224.4.2:8002/index.php;
        }

        location /fern/{
        proxy_bind 192.224.2.2;
        proxy_pass http://192.224.4.3:8003/index.php;
        }


}
' > /etc/nginx/sites-available/lb-eisen

service nginx restart
```
Lalu, langsung dapat dites dengan command lynx berikut:
```shell
lynx canyon.f06.com:81/frieren
```
Dan mendapatkan hasil berikut:

![Alt text](images/nomer18a.png)


## Soal-19
- [Daftar Isi](#daftar-isi)

Untuk meningkatkan performa dari Worker, coba implementasikan PHP-FPM pada Frieren, Flamme, dan Fern. Untuk testing kinerja naikkan 
- pm.max_children
- pm.start_servers
- pm.min_spare_servers
- pm.max_spare_servers

sebanyak tiga percobaan dan lakukan testing sebanyak 100 request dengan 10 request/second kemudian berikan hasil analisisnya pada Grimoire.

Untuk menyelesaikan soal ini, diperlukan pembuatan konfigurasi `pool` baru di PHP-FPM dan pengubahan `.sock` di `virtual host`. Dapat dijalankan dengan skrip berikut:

```shell
touch /etc/php/8.0/fpm/pool.d/riegel.conf
echo '
[riegel_canyon]
user = riegel_user
group = riegel_user
listen = /var/run/php8.0-fpm-riegel.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 75
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.process_idle_timeout = 10s
' > /etc/php/8.0/fpm/pool.d/riegel.conf

groupadd riegel_user
useradd -g riegel_user riegel_user

service php8.0-fpm restart

echo '
server {

    listen 800x;

    root /var/www/laravel-praktikum-jarkom/public;

    index index.php index.html index.htm;
    server_name _;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php8.0-fpm-riegel.sock;
    }
    location ~ /\.ht {
            deny all;
    }

    error_log /var/log/nginx/implementasi_error.log;
    access_log /var/log/nginx/implementasi_access.log;
}'> /etc/nginx/sites-available/riegel

systemctl restart 

cd /var/www/laravel-praktikum-jarkom
chmod -R 777 public
chmod -R 777 storage
```

Berikut adalah hasil testing menggunakan command ab `ab -n 100 -c 10 -g out.data http://canyon.f06.com:81/`:

```
pm = dynamic
pm.max_children = 75
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.process_idle_timeout = 10s
```

![Alt text](images/nomer19a.png)

```
pm = dynamic
pm.max_children = 10
pm.start_servers = 5
pm.min_spare_servers = 1
pm.max_spare_servers = 5
pm.process_idle_timeout = 10s
```

![Alt text](images/nomer19b.png)

```
pm = dynamic
pm.max_children = 100
pm.start_servers = 50
pm.min_spare_servers = 10
pm.max_spare_servers = 50 
pm.process_idle_timeout = 10s
```

![Alt text](images/nomer19c.png)

## Soal-20
- [Daftar Isi](#daftar-isi)

Nampaknya hanya menggunakan PHP-FPM tidak cukup untuk meningkatkan performa dari worker maka implementasikan Least-Conn pada Eisen. Untuk testing kinerja dari worker tersebut dilakukan sebanyak 100 request dengan 10 request/second. 

Untuk menyelesaikan soal ini, tinggal menambahkan `least_conn;` di konfigurasi load balancer di Eisen.

Berikut adalah hasil testing sebelum digunakan Least Connection:

![Alt text](images/nomer19c.png)

Berikut adalah hasil testing setelah digunakan Least Connection:

![Alt text](images/nomer20a.png)

## Kendala
- Azhar tidak ada kontribusi mengerjakan sama sekali.
- Azhar tidak ikut demo rekaman video.









