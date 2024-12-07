# Virtualisation Réseau : On se remet dans le bain.

![meow](https://media1.tenor.com/m/I5uR41-Gmn4AAAAd/cat-kitten.gif)

<u>

## 1. Most Simplest LAN.
</u>

Pour rappel : le tableau d'adressage.

|      Machines       | IP correspondate |
|:-------------------:|:----------------:|
|  `node1.tp1.efrei`  |  `10.1.1.1/24`   |
|  `node2.tp1.efrei`  |  `10.1.1.2/24`   |

---
<u>

### 1.1. Les adresses MAC.
</u>

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

<u>

### 1.2. Les adresses IP.
</u>

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
<u>

### 1.3. Le ping.
</u>

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
<u>

### 1.4. Wireshark.
</u>

Ok, time to wireshark !!

[Ping entre les deux machines](pcaps/Capture_pings.pcapng)   

On voit bien que les deux machines se ping avec succès.

Quel est le protocole utilisé pour le ping ? **ICMP** bien sûr.

<u>

### 1.5 Requêtes ARP.
</u>

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

<u>

## 2. Ajoutons un switch.
</u>

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

<u>

### 2.2. Le ping.
</u>

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

<u>

## 3. Serveur DHCP.
</u>

On va maintenant ajouter un serveur DHCP pour attribuer les IP automatiquement.   
Et accessoirement faire joujou avec les options.

<u>

### 3.1. Legit.
</u>

Hop la table d'adressage

|      Machines       | IP correspondate |
|:-------------------:|:----------------:|
|  `node1.tp1.efrei`  |      `N/A`       |
| `node2.tp1.efrei`  |        `N/A`     |
|  `node3.tp1.efrei`  |      `N/A`       |
|  `dhcp.tp1.efrei`   | `10.1.1.253/24`  |

<u>

#### 3.1.1. Configuration du serveur DHCP.
</u>

On commence par fournir un accès internet au serveur DHCP.

```shell
\[root@localhost \~\]# curl ifconfig.me 84.235.235.54  
\[root@localhost \~\]# ping kali.download  
PING kali.download (104.17.254.239) 56(84) bytes of data.  
64 bytes from 104.17.254.239 (104.17.254.239): icmp_seq=1 ttl=57 time=24.9 ms  
64 bytes from 104.17.254.239 (104.17.254.239): icmp_seq=2 ttl=57 time=24.2 ms  
64 bytes from 104.17.254.239 (104.17.254.239): icmp_seq=3 ttl=57 time=24.4 ms  
^C  
\--- kali.download ping statistics ---  
3 packets transmitted, 3 received, 0% packet loss, time 2005ms  
rtt min/avg/max/mdev = 24.224/24.482/24.863/0.274 ms  
\[root@localhost \~\]#  
```

Ça fonctionne bien, on peut ping un domaine random, on passe à la suite.  

Grâce à [Ce super tutoriel](https://gitlab.com/it4lik/b2-network-virt/-/blob/main/memo/rocky_network.md) On définit une addresse statique pour notre serveur.

> Configuration IP:

```shell
[root@localhost \~\]# cat /etc/sysconfig/network-scripts/ifcfg-en0s3
DEVICE=enp0s3
NAME=Meow

ONBOOT=YES
BOOTPROTO=static

IPADDR=10.1.1.253
NETMASK=255.255.255.0
[root@localhost \~\]#
```

Ensuite, on installe le serveur DHCP.  
L'installation est relativement simple, il suffit d'exécuter les commandes suivantes:

```shell
[root@localhost \~\]# dnf-y install dhcp-server
[root@localhost \~\]# vim /etc/dhcp/dhcpd.conf
```

Dans /etc/dhcp/dhcpd.conf, on ajoute les lignes suivantes:

```
authoritative;
subnet 10.1.1.0 netmask 255.255.255.0 {
	range 10.1.1.10 10.1.1.50;
	option broadcast-address 10.1.1.1;
	option routers 10.1.1.1;
}
```

On démarre le service DHCP:

```shell
systemctl enable --now dhcpd
```

Il est possible que le serveur miaule, et ne veuille pas démarrer !  
Pour moi, reboot la VM a fonctionné.

<u>

#### 3.1.2. Récupérer une IP depuis un client.
</u>

On va maintenant attribuer une IP à une machine cliente.  
On utilise la commande `dhcp` pour obtenir une IP automatiquement.  

Ensuite, on vérifie que l'IP est bien attribuée avec `show ip`.  

> Machine 1:
```
C1> dhcp  
DORA IP 10.1.1.10/24 GW 10.1.1.1  
  
PC1> show ip  
  
NAME        : PC1\[1\]  
IP/MASK     : 10.1.1.10/24  
GATEWAY     : 10.1.1.1  
DNS         :  
DHCP SERVER : 10.1.1.253  
DHCP LEASE  : 42892, 42898/21449/37535  
MAC         : 00:50:79:66:68:02  
LPORT       : 10008  
RHOST:PORT  : 127.0.0.1:10009  
MTU:        : 1500  
  
PC1>  
```

Bien, Machine 2:

> Machine 2:
```
PC2> dhcp
DORA IP 10.1.1.11/24 GW 10.1.1.1

PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.11/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 43089, 43191/21595/37792
MAC         : 00:50:79:66:68:01
LPORT       : 10010
RHOST:PORT  : 127.0.0.1:10011
MTU:        : 1500

PC2>
```

Parfait, Machine 3:

> Machine 3:
```
PC3> dhcp
DDORA IP 10.1.1.12/24 GW 10.1.1.1

PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.1.1.12/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 43152, 43200/21600/37800
MAC         : 00:50:79:66:68:00
LPORT       : 10012
RHOST:PORT  : 127.0.0.1:10013
MTU:        : 1500

PC3>
```

<u>

#### 3.1.3. Wireshark:
</u>

On va maintenant observer les échanges DORA entre le serveur DHCP et une machine cliente.  
On va donc capturer Dora...  
<img src="dora.jpg" width="40"/>

[Capture DHCP](pcaps/capture_dora.pcapng)

<u>

### 3.2. DHCP spoofing.
</u>

On va maintenant tenter de faire du DHCP spoofing.  
On va donc essayer de récupérer les requêtes DHCP d'une machine cliente, et de lui attribuer une IP différente.

On part d'une autre rocky Linux, autant faire simple.

<u>

#### 3.2.1 Configuration du serveur
</u>

Avant toute chose, on pense à setup une ip statique pour notre serveur DHCP.  
On l'a fait plus haut, mais on le refait pour cette machine.  

Personellement, j'ai mis `10.1.1.251` pour ce serveur là.

Ensuite on installe dnsmasq.

```bash
dnf install -y dnsmasq
```

On configure ensuite le fichier `/etc/dnsmasq.conf` pour qu'il ressemble à ça:

```
port=0
dhcp-range=10.1.1.210,10.1.1.250,255.255.255.0,12h
dhcp-authoritative
interface=enp0s3
```

Une fois cela fait, on démarre le service dnsmasq.

```bash
systemctl enable --now dnsmasq
```

Là, ça ne fonctionne pas, pour une raison X ou Y.  
Il faut penser à désactiver IPtables...

```bash
sudo dnf remove iptables
```

On redémarre la machine par précaution, et on relance le service.  
On fait un test rapide avec un nouveau VPCS:

```
C3> dhcp
DDORA IP 10.1.1.245/24 GW 10.1.1.251

PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.1.1.245/24
GATEWAY     : 10.1.1.251
DNS         :
DHCP SERVER : 10.1.1.251
DHCP LEASE  : 43049, 43200/21600/37800
MAC         : 00:50:79:66:68:00
LPORT       : 10012
RHOST:PORT  : 127.0.0.1:10013
MTU:        : 1500
```

`10.1.1.245`, ça a l'air bon.

<u>

#### 3.2.2. Spoofing & Race.
</u>

On va maintenant tenter de faire du DHCP spoofing.  
On va connecter les deux serveurs DHCP sur le même switch, et on va lancer quelques VPCS pour voir si on peut récupérer des ip différentes.  

> Machine 3:
```
PC3> dhcp
DORA IP 10.1.1.12/24 GW 10.1.1.1

PC3> 
```
> Machine 4:
```
PC4> dhcp
DDORA IP 10.1.1.248/24 GW 10.1.1.251

PC4>
```
> Machine 5:
```
PC5> dhcp
DDORA IP 10.1.1.249/24 GW 10.1.1.251

PC5>
```

On va dire 2/3 récupèrent leur ip du serveur "malveillant". La classe.

On va maintenant faire une "Race" en observant le serveur légitime et en enregistrant avec wireshark.

[Capture DHCP Spoofing](pcaps/dhcp_race.pcapng)

Contrairement à ce qu'on a vu juste avant, le serveur illégitime ne récupère qu'une seule requête DHCP, et le serveur légitime récupère le reste... On pourrait lui "casser les jambes" pour permettre au serveur illégitime de récupérer plus de traffic....

---

BONUS ??? (À voir si j'ai le temps)

---

![pas mal non ?](pasmal.jpg)
