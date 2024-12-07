# Virtualisation Réseau : On se remet dans le bain.

![meow](https://media1.tenor.com/m/I5uR41-Gmn4AAAAd/cat-kitten.gif)

---

## 1. Most Simplest LAN.

Pour rappel : le tableau d'adressage.

|      Machines       | IP correspondate |
|:-------------------:|:----------------:|
|  `node1.tp1.efrei`  |  `10.1.1.1/24`   |
|  `node2.tp1.efrei`  |  `10.1.1.2/24`   |

---
### 1.1. Les adresses MAC.

On commence par déterminer les adressesMAC des deux machines.

> Machine 1:

```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 0.0.0.0/0
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10000
RHOST:PORT  : 127.0.0.1:10001
MTU:        : 1500
```
On a donc `00:50:79:66:68:00` pour la première machine.  
> Machine 2:

```

PC2> show ip

NAME        : PC2[1]
IP/MASK     : 0.0.0.0/0
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 10008
RHOST:PORT  : 127.0.0.1:10009
MTU:        : 1500

PC2>
```
Et `00:50:79:66:68:01` pour la seconde machine.

### 1.2. Les adresses IP.

Hop une IP pour PC1:

```
PC1> ip 10.1.1.1/24
Checking for duplicate address...
PC1 : 10.1.1.1 255.255.255.0

PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.1/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10000
RHOST:PORT  : 127.0.0
```
(On vérifie bien que l'addresse est bien appliquée avec `show ip`)

Et une IP pour PC2:

```
PC2> ip 10.1.1.2/24
Checking for duplicate address...
PC1 : 10.1.1.2 255.255.255.0

PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.2/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 10008
RHOST:PORT  : 127.0.0.1:10009
MTU:        : 1500
```

### 1.3. Le ping.

On vérifie que les deux machines se ping bien.

> De PC1 à PC2:
```
PC1> ping 10.1.1.2
84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.177 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.475 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=0.355 ms
```

> De PC2 à PC1:
```
PC2> ping 10.1.1.1
84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=0.369 ms
84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=0.370 ms
84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=0.367 ms
```

### 1.4. Wireshark.

Ok, time to wireshark !!

[Ping entre les deux machines](pcaps/Capture_pings.pcapng)  
On voit bien que les deux machines se ping bien.

Quel est le protocole utilisé pour le ping ? ICMP bien sûr.

### 1.5 Requêtes ARP.

On souhaite maintenant connaître les adresses MAC des deux machines, surtout d'une machine vers l'autre.  
On va donc faire un ping de PC1 vers PC2, et observer les requêtes ARP.  

après ping du PC1 vers PC2:

```
PC1> arp

00:50:79:66:68:01  10.1.1.2 expires in 117 seconds
```

On voit bien que l'adresse MAC de PC2 est `00:50:79:66:68:01`.

Ensuite, on ping du PC2 vers PC1 (pas obligatoire, mais pour le fun surtout):

```
PC2> arp

00:50:79:66:68:00  10.1.1.1 expires in 78 seconds
```

Et on voit bien que l'adresse MAC de PC1 est `00:50:79:66:68:00`.

---

## 2. Ajoutons un switch.

On est pas des sauvages... 2eme table d'adressage.

|      Machines       | IP correspondate |
|:-------------------:|:----------------:|
|  `node1.tp1.efrei`  |  `10.1.1.1/24`   |
|  `node2.tp1.efrei`  |  `10.1.1.2/24`   |
|  `node3.tp1.efrei`  |  `10.1.1.3/24`   |

### 2.1 Déterminer les adresses MAC + attribuer les IP.

On commence par déterminer les adresses MAC des trois machines.  
On en profite pour vérifier que les IP sont bien attribuées.
Encore une fois, on attribue les IP puis on utilise `show ip` pour obtenir les adresses MAC.

> Machine 1:

```
PC1> ip 10.1.1.1/24
Checking for duplicate address...
PC1 : 10.1.1.1 255.255.255.0

PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.1/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10006
RHOST:PORT  : 127.0.0.1:10007
MTU:        : 1500
```

> Machine 2:

```
PC2> ip 10.1.1.2/24
Checking for duplicate address...
PC1 : 10.1.1.2 255.255.255.0

PC2> ip show
Invalid address

PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.2/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 10008
RHOST:PORT  : 127.0.0.1:10009
MTU:        : 1500
```

> Machine 3:

```
PC3> ip 10.1.1.3/24
Checking for duplicate address...
PC3 : 10.1.1.3 255.255.255.0

PC3> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.3/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:02
LPORT       : 10009
RHOST:PORT  : 127.0.0.1:10009
MTU:        : 1500
```

### 2.2. Le ping.

C'est parti pour vérifier que les machines se ping bien.

On ping :
- de node1 à node2
- de node2 à node3
- de node1 à node3

> De node1 à node2:
```
PC1> ping 10.1.1.2  
84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.379 ms  
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.574 ms
```

> De node2 à node3:
```
PC2> ping 10.1.1.3
84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.393 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.375 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=0.362 ms
```

> De node1 à node3:
```
PC1> ping 10.1.1.3
84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.336 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.484 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=0.459 ms
```

---

![Fear...](fear.png)

---

### 3. Serveur DHCP.

