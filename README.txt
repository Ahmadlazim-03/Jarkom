BASIC CONFIGURATION 

1. /ip dhcp-client add interface=ether1 use-peer-dns=no add-default-route=yes disabled=no
2. /ip address add address=192.168.10.1/24 interface=ether6
3. /ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
4. /ip pool add name=dhcp_pool_ether6 ranges=192.168.10.2-192.168.10.254
5. /ip dhcp-server add name=dhcp_ether6 interface=ether6 address-pool=dhcp_pool_ether6 lease-time=10m disabled=no
6. /ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=8.8.8.8,1.1.1.1
7. /ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes

BRIDGE

/interface bridge
add name=bridge-lan

/interface wireless
set [ find default-name=wlan1 ] ssid=MikroTik

/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik

/ip pool
add name=dhcp_pool0 ranges=192.168.1.2-192.168.1.254
add name=dhcp_pool1 ranges=192.168.3.2-192.168.3.254

/ip dhcp-server
add address-pool=dhcp_pool0 disabled=no interface=ether2 name=dhcp1
add address-pool=dhcp_pool1 disabled=no interface=bridge-lan name=dhcp2

/interface bridge port
add bridge=bridge-lan interface=ether3
add bridge=bridge-lan interface=ether4

/ip address
add address=192.168.1.1/24 interface=ether2 network=192.168.1.0
add address=192.168.3.1/24 interface=bridge-lan network=192.168.3.0

/ip dhcp-client
add disabled=no interface=ether1

/ip dhcp-server network
add address=192.168.1.0/24 gateway=192.168.1.1
add address=192.168.3.0/24 gateway=192.168.3.1

/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether1

/system clock
set time-zone-name=Asia/Jakarta

WIFI AND LIMIT

/ip pool add name=pool-lan ranges=192.168.100.10-192.168.100.100
/ip dhcp-server add name=dhcp-lan interface=ether2 address-pool=pool-lan disabled=no
/ip dhcp-server network add address=192.168.100.0/24 gateway=192.168.100.1 dns-server=8.8.8.8
/ip pool add name=pool-wifi ranges=192.168.200.10-192.168.200.254
/ip dhcp-server add name=dhcp-wifi interface=wlan1 address-pool=pool-wifi disabled=no
/ip dhcp-server network add address=192.168.200.0/24 gateway=192.168.200.1 dns-server=8.8.8.8
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/ip hotspot profile add name=hotspot-profile interface=wlan1 address-pool=pool-wifi dns-name=hotspot.local hotspot-address=192.168.200.1
/ip hotspot add name=hotspot1 interface=wlan1 address-pool=pool-wifi profile=hotspot-profile
/ip hotspot user add name=pengguna1 password=12345 profile=default
/ip hotspot user profile set default rate-limit=""
/ip hotspot user profile add name=tamu rate-limit=5120k/5120k
/ip hotspot user add name=tamu1 password=tamu123 profile=tamu
open "http://192.168.200.1"
🔧 1. Buat Hotspot di Interface wlan1
Jika belum, lakukan ini:

Masuk ke WebFig → IP > Hotspot > Hotspot Setup

Pilih interface: wlan1

Address for local: 192.168.200.1

Masukkan DNS: 8.8.8.8

Masukkan IP Pool: 192.168.200.10-192.168.200.254

SSL Certificate: none

SMTP: kosongkan (0.0.0.0)

Masukkan DNS name hotspot: hotspot.local (bebas, nanti user login lewat ini)

Selesai.
👤 2. Buat User Hotspot
🔹 Pengguna terdaftar (no limit)
/ip hotspot user add name=pengguna1 password=12345 profile=unlimited
Buat profil unlimited:
/ip hotspot user profile add name=unlimited shared-users=1 rate-limit=""
🔹 Tamu (limit 5 Mbps)
/ip hotspot user profile add name=guest rate-limit=5M/5M shared-users=1
/ip hotspot user add name=tamu1 password=tamu123 profile=guest