# 3 - La Partie 3.

---

<u>
Sommaire</u>

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