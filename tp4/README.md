# TP-Reseau - Routage Statique

# Préparation d'une VM "patron"

Après avoir crée une machine virtuelle "patron", il a fallut cloner ce patron afin d'en créer 3 VM modifiables.

# I. Mise en place du lab
Simple création de VMs !

# 1. Création des réseaux
2 cartes réseau ont du être crées, une ayant comme IP réseau : `10.1.0.0` et l'autre `10.2.0.0`, bien entendu, les 2 cartes ont le **DHCP** désactivé.

# 2. Création des VMs

Le clonage des VMs sur VirtualBox a du se faire comme ceci :
  * clone intégral
  * réinitialisation des adresses MAC : OUI
  
  
La machine client dispose d'une interface privé hôte ayant une route dans le **net1**


Le machine serveur dispose de la même chose, mais ayant une route dans le **net2**

Par contre, la machine routeur dispose d'une route dans le **net1** ET dans le **net2**


Voici le tableau récapitulatif :
  
| Machine       | net1           | net2  |
| ------------- |:-------------:| -----: |
| client      | 10.1.0.10 | X            |
| routeur     | 10.1.0.254|   10.1.0.254 |
| serveur     | X         |   10.2.0.10  |

Une fois ceci fait, on peut donc se connecter par SSH.

Tout fonctionne correctement, la machine routeur est accessible depuis `10.1.0.254` et `10.2.0.254` , c'est ce que l'on veut.

Ensuite, on définit les noms de domaine.

La machine client dispose de ce nom de domaine : `client1.tp4.b1`

La machine serveur : `serveur1.tp4.b1`

La machine routeur : `routeur1.tp4.b1`

Voici les fichiers hosts des machines afin de pas retaper l'IP à chaque fois ..

`sudo nano /etc/hosts` sur toutes les machines

Machine routeur :

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 routeur1 routeur1.tp4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.0.10 client1 client1.tp4
10.2.0.10 serveur1 serveur1.tp4
```

Machine client :

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 client1 client1.tp4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.2.0.10 serveur1 serveur1.tp4
10.1.0.254 routeur1 routeur1.tp4
```

Machine serveur :

```                                                     
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 serveur1 serveur1.tp4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.0.10 client1 client1.tp4
10.2.0.254 routeur1 routeur1.tp4
```

Pour que les machines puissent se ping, on va ajouter des routes.

On tape cette commande sur le client et le serveur :
`sudo nano /etc/sysconfig/network-scripts/route-enp0s8`

On y ajoute la ligne

`10.2.0.0/24 via 10.1.0.254 dev enp0s8` pour le client

`10.1.0.0/24 via 10.2.0.254 dev enp0s8` pour le serveur


C'est génial ! Notre client et notre serveur peuvent se ping !

```
[vm1@client1 ~]$ ping serveur1
PING serveur1 (10.2.0.10) 56(84) bytes of data.
64 bytes from serveur1 (10.2.0.10): icmp_seq=1 ttl=63 time=0.711 ms
64 bytes from serveur1 (10.2.0.10): icmp_seq=2 ttl=63 time=0.497 ms
64 bytes from serveur1 (10.2.0.10): icmp_seq=3 ttl=63 time=0.621 ms
64 bytes from serveur1 (10.2.0.10): icmp_seq=4 ttl=63 time=0.631 ms
^C
--- serveur1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.497/0.615/0.711/0.076 ms
```

```
[vm1@serveur1 ~]$ ping client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.727 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=0.677 ms
64 bytes from client1 (10.1.0.10): icmp_seq=3 ttl=63 time=0.666 ms
64 bytes from client1 (10.1.0.10): icmp_seq=4 ttl=63 time=0.631 ms
^C
--- client1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 0.631/0.675/0.727/0.038 ms
```

Bon, et vu que le client et le routeur sont sur le même réseau, le ping se fait tout simplement !
```
[vm1@client1 ~]$ ping routeur1
PING routeur1 (10.1.0.254) 56(84) bytes of data.
64 bytes from routeur1 (10.1.0.254): icmp_seq=1 ttl=64 time=0.207 ms
64 bytes from routeur1 (10.1.0.254): icmp_seq=2 ttl=64 time=0.180 ms
64 bytes from routeur1 (10.1.0.254): icmp_seq=3 ttl=64 time=0.196 ms
64 bytes from routeur1 (10.1.0.254): icmp_seq=4 ttl=64 time=0.337 ms
^C
--- routeur1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.180/0.230/0.337/0.062 ms
```

De même pour le serveur :

```
[vm1@serveur1 ~]$ ping routeur1
PING routeur1 (10.2.0.254) 56(84) bytes of data.
64 bytes from routeur1 (10.2.0.254): icmp_seq=1 ttl=64 time=0.266 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=2 ttl=64 time=0.290 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=3 ttl=64 time=0.317 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=4 ttl=64 time=0.341 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=5 ttl=64 time=0.241 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=6 ttl=64 time=0.310 ms
^C
--- routeur1 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5003ms
rtt min/avg/max/mdev = 0.241/0.294/0.341/0.034 ms
```

# 3. Mise en place du routage statique
Fait juste au dessus (grâce à la checklist)

Ajout des commandes sur le routeur pour :

 * activer l'IPv4 Forwarding (= transformer la machine en routeur)
   * `sudo sysctl -w net.ipv4.conf.all.forwarding=1`
 * désactiver le firewall (pour éviter certaines actions non voulues)
   * `sudo systemctl disable firewalld (permanent)`

On fait un `ip route show`
```
[vm1@routeur1 ~]$ ip route show
10.1.0.0/24 dev enp0s3 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s8 proto kernel scope link src 10.2.0.254 metric 101
```

Tout est niquel il y a déjà des routes pour accéder à **net1** et **net2** (logique :o)


Un petit traceroute est effectué du client au serveur :

```
[vm1@client1 ~]$ traceroute serveur1
traceroute to serveur1 (10.2.0.10), 30 hops max, 60 byte packets
 1  routeur1 (10.1.0.254)  0.254 ms  0.184 ms  0.136 ms
 2  serveur1 (10.2.0.10)  0.435 ms !X  0.511 ms !X  0.560 ms !X
 ```
 
 On peut voir qu'il essaye de se connecter à `10.2.0.10` via `10.1.0.254`, il y parvient.


# II. Spéléologie réseau
**SELinux est désactivé
Ma carte NAT est désactivée**

# 1. ARP
ARP est le protocole qui permet de connaître la MAC d'une machine quand on connaît son IP.
# A. Manip1
On flush la table ARP des 3 machines.

Sur le client :
```
[vm1@client1 ~]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0e DELAY
```

Cette ligne signifie que nous connaissons l'adresse MAC de notre carte réseau .. heureusement

Sur le serveur :
```
[vm1@serveur1 ~]$ ip neigh show
10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:47 DELAY
```

De même que le client


Après un petit ping sur le serveur, le client nous montre une nouvelle ligne :

```
[vm1@client1 ~]$ ip neigh show
10.1.0.254 dev enp0s8 lladdr 08:00:27:b5:2d:a1 REACHABLE
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0e DELAY
```

Elle indique que la gateway de l'IP que nous avons ping est atteignable, donc son adresse MAC est désormais connue par notre VM.

Sur le serveur :

```
[vm1@serveur1 ~]$ ip neigh show
10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:47 DELAY
10.2.0.254 dev enp0s8 lladdr 08:00:27:9e:4c:fd STALE
```

Cela signifie simplement que le routeur a communiqué avec le serveur en réagissant au ping du client, ce qui a inscrit cette ligne.

# B. Manip2

On re-flush la table ARP des 3 machines.

Sur le routeur on affiche la table :

```
[vm1@routeur1 ~]$ ip neigh show
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:0e REACHABLE
```

Cela signifie que le routeur est capable de communiquer avec le réseau par lequel on est connecté via SSH

Après un ping de client1 vers serveur1 la table du routeur a changée :

```
[vm1@routeur1 ~]$ ip neigh show
10.2.0.10 dev enp0s8 lladdr 08:00:27:e9:96:80 REACHABLE
10.1.0.10 dev enp0s3 lladdr 08:00:27:f6:ff:2c DELAY
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:0e REACHABLE
```

Il a rajouté l'adresse du serveur1, car il connais désormais qu'il y a une machine sur son réseau qui s'appelle comme cela.

# C. Manip 3
On affiche la table ARP sur l'hôte après avoir flush les tables des VM :
```
C:\Users\Ju'>arp -a

Interface : 10.33.1.139 --- 0x9
  Adresse Internet      Adresse physique      Type
  10.33.0.54            bc-a8-a6-fd-da-7c     dynamique
  10.33.0.139           5c-c5-d4-8c-83-c7     dynamique
  10.33.1.164           f8-63-3f-6b-81-4f     dynamique
  10.33.1.198           68-07-15-41-b8-fb     dynamique
  10.33.2.145           04-d3-b0-0e-73-a1     dynamique
  10.33.2.179           b4-d5-bd-ac-ca-ec     dynamique
  10.33.3.198           08-d4-0c-c7-20-04     dynamique
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  10.33.3.254           94-0c-6d-84-50-c8     dynamique
  10.33.3.255           ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.1.0.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.1.0.10             08-00-27-f6-ff-2c     dynamique
  10.1.0.254            08-00-27-b5-2d-a1     dynamique
  10.1.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 169.254.92.118 --- 0x17
  Adresse Internet      Adresse physique      Type
  169.254.255.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.2.0.1 --- 0x47
  Adresse Internet      Adresse physique      Type
  10.2.0.10             08-00-27-e9-96-80     dynamique
  10.2.0.254            08-00-27-9e-4c-fd     dynamique
  10.2.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```

On flush ensuite notre table
```
C:\WINDOWS\system32>arp -d
```

Puis on l'affiche :
```
C:\WINDOWS\system32>arp -a

Interface : 10.33.1.139 --- 0x9
  Adresse Internet      Adresse physique      Type
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.1.0.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 169.254.92.118 --- 0x17
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.2.0.1 --- 0x47
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
```

On attend un petit peu puis on la ré-affiche :
```
C:\WINDOWS\system32>arp -a

Interface : 10.33.1.139 --- 0x9
  Adresse Internet      Adresse physique      Type
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  10.33.3.255           ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.1.0.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 169.254.92.118 --- 0x17
  Adresse Internet      Adresse physique      Type
  169.254.255.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.2.0.1 --- 0x47
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

C:\WINDOWS\system32>
```

Une nouvelle ligne est apparue

# D. Manip 4
Après avoir flush la table ARP de toutes les machines

Sur client1 :

```
[vm1@client1 ~]$ ip neigh show
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0e REACHABLE
```

On active la carte NAT :

```
[vm1@client1 ~]$ sudo ifup enp0s3
[sudo] Mot de passe de vm1 : 
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/2)
```

On fait un petit curl sur google

```
[vm1@client1 ~]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
[vm1@client1 ~]$
```

Ce qui nous rajoute une ligne sur notre table ARP :

```
[vm1@client1 ~]$ ip neigh show
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:0e REACHABLE
```

`10.0.2.2` car c'est l'alias de l'interface loopback de l'hôte. Utilisé pour se connecter à un serveur web.

# 2. Wireshark
L'installation de tcpdump a déjà été faite auparavant.

Etant connecté en SSH à `10.1.0.254`, on va observer le trafic de l'interface `10.2.0.254`

# A. Interception d'ARP et `ping`

Sur le routeur on enregistre le trafic :

`sudo tcpdump -i enp0s8 -w ping.pcap`

Sur le client, on vide la table, on envoie 4 pings au serveur.

De nouveau sur le routeur, on envoi le fichier `ping.pcap` via un serveur web.

`python2 -m SimpleHTTPSHTTPServer 8888`

Cela nous permet de nous connecter à `10.1.0.254:8888` sur le web afin d'accéder au fichier.

On peut enfin ouvrir le fichier `ping.pcap` sur le PC hôte

On peut constater qu'il nous manque la moitié des trames, car le client a ping le serveur en passant par le routeur, ce qui veut dire qu'il y a aussi le serveur qui répond au routeur, et cela n'est pas affiché dans les informations.

# Interception d'une communication `netcat`

`sudo tcpdump -i enp0s8 -w netcat_ok.pcap`

![Trame WireShark](tp4/images/netcat_wireshark.PNG "Trame WireShark")

Les messages en **gris** sont une demande de synchronisation entre 10.2.0.10 et 10.1.0.10, cela veut dire que le client veut se connecter au serveur.
Une fois qu'il est connecté, nous pouvons voir des messages ARP, cela signifie que la table ARP reconnais que l'IP est atteignable, l'adresse MAC est enregistrée, ensuite les messages TCP sont notre joli dialogue entre client/serveur !

**Fermer le port firewall du serveur et refaire la même chose pour se faire jeter comme un caca ? Let's go !**

```
[vm1@serveur1 ~]$ sudo firewall-cmd --remove-port=5454/tcp --permanent
[sudo] Mot de passe de vm1 : 
success
[vm1@serveur1 ~]$ sudo firewall-cmd --reload
success
```

On relance un petit tcpdump !

`sudo tcpdump -i enp0s8 -w netcat_ko.pcap`


![Trame WireShark](tp4/images/netcat_ko.PNG "Trame WireShark")
