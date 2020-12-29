# Jarkom_Modul5_LaporanResmi_B01

1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi
SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan
MASQUERADE.

SURABAYA
```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.74.10
```

2. Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server
yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

SURABAYA
```
iptables -A FORWARD -d 10.151.83.16/29 -i eth0 -p tcp --dport 22 -j DROP
```

3. Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP
dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari
mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

SURABAYA, MALANG, MOJOKERTO
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
```

4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin
sampai Jumat.

MALANG
```
iptables -A INPUT -s 192.168.1.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays
Mon,Tue,Wed,Thu,Fri -j ACCEPT
```

5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap
harinya.

MALANG
```
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 17:00 --timestop 00:00 -j ACCEPT
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 00:00 --timestop 07:00 -j ACCEPT
```

6. Bibah ingin SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

SURABAYA

Probolinggo: 
```
iptables -A PREROUTING -t nat -p tcp -d 10.151.83.18 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.0.11:80
```
Madiun: 
```
iptables -A PREROUTING -t nat -p tcp -d 10.151.83.18 -j DNAT --to-destination 192.168.0.10:80
```

7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap
UML yang memiliki aturan drop.

SURABAYA, MALANG, MOJOKERTO
```
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING #no3
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "DROP: " --log-level info
iptables -A LOGGING -j DROP
```
