# I - Création et utilisation simples d'une VM CentOS
Le tp a été fait jusqu'à la partie 4
# 4. Configuration réseau d'une machine CentOS
* A : On utilise une commande sur la VM pour prouver que l'on a internet :
```
[root@localhost ~]# curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```


* B : Prouver que l'hôte et la VM communiquent se font via un ping.
    * Adresse IP de la VM : 192.168.127.10
    * Adresse IP de l'hôte : 192.168.127.1

Sur l'hôte :
```
C:\Users\Ju'>ping 192.168.127.10

Envoi d’une requête 'Ping'  192.168.127.10 avec 32 octets de données :
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.127.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```

Et sur la VM :
```
[root@localhost ~]# ping 192.168.127.1
PING 192.168.127.1 (192.168.127.1) 56(84) bytes of data.
64 bytes from 192.168.127.1: icmp_seq=1 ttl=64 time=0.218 ms
64 bytes from 192.168.127.1: icmp_seq=2 ttl=64 time=0.281 ms
64 bytes from 192.168.127.1: icmp_seq=3 ttl=64 time=0.278 ms
64 bytes from 192.168.127.1: icmp_seq=4 ttl=64 time=0.275 ms
^C
--- 192.168.127.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3311ms
rtt min/avg/max/mdev = 0.218/0.263/0.281/0.026 ms
```

Sur la VM, on affiche nôtre table de routage :
```
[root@localhost ~]# ip route
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 101
```

Sur la première et deuxième ligne, connexion au réseau avec une ip "publique" via 10.0.2.2 et la carte `enp0s3` grâce a notre OS principal (ici Windaube)


Sur la troisième, c'est l'IP "privée" pour se connecter au réseau local via 192.168.127.10 et la carte `enp0s8`


# 5. Faire joujou avec les commandes

Afficher la table de routage de l'hôte :
```
IPv4 Table de routage
===========================================================================
Itinéraires actifs :
Destination réseau    Masque réseau  Adr. passerelle   Adr. interface Métrique
          0.0.0.0          0.0.0.0      10.33.3.253      10.33.1.139     35
        10.33.0.0    255.255.252.0         On-link       10.33.1.139    291
      10.33.1.139  255.255.255.255         On-link       10.33.1.139    291
      10.33.3.255  255.255.255.255         On-link       10.33.1.139    291
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      169.254.0.0      255.255.0.0         On-link    169.254.92.118    281
   169.254.92.118  255.255.255.255         On-link    169.254.92.118    281
  169.254.255.255  255.255.255.255         On-link    169.254.92.118    281
    192.168.127.0    255.255.255.0         On-link     192.168.127.1    281
    192.168.127.1  255.255.255.255         On-link     192.168.127.1    281
  192.168.127.255  255.255.255.255         On-link     192.168.127.1    281
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link     192.168.127.1    281
        224.0.0.0        240.0.0.0         On-link       10.33.1.139    291
        224.0.0.0        240.0.0.0         On-link    169.254.92.118    281
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link     192.168.127.1    281
  255.255.255.255  255.255.255.255         On-link       10.33.1.139    291
  255.255.255.255  255.255.255.255         On-link    169.254.92.118    281
===========================================================================
Itinéraires persistants :
  Aucun
```

Afficher la table de routage de la VM :
```
[root@localhost ~]# ip route
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 101
```

Ce qui permet de se **connecter** à la Vm :
```
192.168.127.0    255.255.255.0         On-link     192.168.127.1    281
192.168.127.1  255.255.255.255         On-link     192.168.127.1    281 (Windaube)
```

On utilise wget pour telecharger l'index de google : (il a fallu installer le package "**wget**")
```
[root@localhost ~]# wget google.com
--2019-01-08 16:42:20--  http://google.com/
Résolution de google.com (google.com)... 216.58.208.238, 2a00:1450:4007:810::200e
Connexion vers google.com (google.com)|216.58.208.238|:80...connecté.
requête HTTP transmise, en attente de la réponse...301 Moved Permanently
Emplacement: http://www.google.com/ [suivant]
--2019-01-08 16:42:20--  http://www.google.com/
Résolution de www.google.com (www.google.com)... 172.217.22.132, 2a00:1450:4007:810::2004
Connexion vers www.google.com (www.google.com)|172.217.22.132|:80...connecté.
requête HTTP transmise, en attente de la réponse...200 OK
Longueur: non spécifié [text/html]
Sauvegarde en : «index.html»

    [ <=>                                   ] 11 356      --.-K/s   ds 0,001s

2019-01-08 16:42:20 (17,4 MB/s) - «index.html» sauvegardé [11356]
```


On dig (il a fallu installer le package "**bind-utils**"):

Ynov.com : `217.70.184.38`


Google.com : `216.58.208.238`

# II. Notion de ports et SSH

# 1. Exploration des ports locaux

On utilise le ss pour voir les ports TCP que la machine virtuelle écoute


`ss -4` (Pour voir que les ports **IPv4**)
``` 
[root@localhost ~]# ss -4
Netid  State      Recv-Q Send-Q     Local Address:Port                      Peer Address:Port
tcp    ESTAB      0      64        192.168.127.10:ssh                      192.168.127.1:sunscalar-dns
```


Pour avoir les options TCP et LISTENING on rajoute (**trouvé grâce à MAN**)


`ss -4 -t -l`
```
[root@localhost ~]# ss -4 -t -l
State      Recv-Q Send-Q        Local Address:Port                         Peer Address:Port
LISTEN     0      128                       *:ssh                                     *:*
LISTEN     0      100               127.0.0.1:smtp                                    *:*
```


Pour voir les ports utilisés **on rajoute -n et pour connaitre l'application qui écoute sur ce port -p**
```
[root@localhost ~]# ss -4 -t -l -n -p
State      Recv-Q Send-Q          Local Address:Port                         Peer Address:Port
LISTEN     0      128                         *:22                                      *:*                   users:(("sshd",pid=3278,fd=3))
LISTEN     0      100                 127.0.0.1:25                                      *:*                   users:(("master",pid=3522,fd=13))
[root@localhost ~]#
```


On vois une application qui écoute sur le port 22, c'est **bel et bien le SSH** !

(Etant déjà connecté par SSH via **Putty** avant les commandes le copier coller est autorisé !)

# 2. SSH

On désactive SELinux
```
[root@localhost ~]# cd /etc/ssh/
[root@localhost ssh]# sudo setenforce 0
[root@localhost ssh]# cd /etc/selinux
[root@localhost selinux]# sudo nano config
[root@localhost selinux]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```

# 3. Firewall
Une fois ceci fait, on peut commencer sur le Firewall.

On modifie le fichier sshd_config en **changeant le port d'écoute du SSH sur 2222**,
on tape ensuite
`systemctl restart sshd`

Puis avec ss, on peut voir que le port d'écoute du SSH est **désormais 2222**.

En essayant de se connecter au **SSH** via le port 2222, on peut voir que ca marche pas.
Il faut donc **ouvrir le port dans le firewall de notre VM**

```
firewall-cmd --add-port=2222/tcp --permanent
firewall-cmd --reload
```

Une fois ceci fait, la connexion à SSH **refonctionne** !

Dans un premier terminal sur la VM, **on autorise le port 5454 en TCP et on lance NC en écoute** !
```
[root@localhost ~]# firewall-cmd --add-port=5454/tcp --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# nc -l 5454
```


Dans un deuxième terminal, sur **l'ordinateur hôte** on se connecte au serveur netcat
```
C:\Users\Ju'>cd Desktop

C:\Users\Ju'\Desktop>cd NetCat

C:\Users\Ju'\Desktop\NetCat>nc 192.168.127.10 5454
aazzaz
```


Dans un troisième terminal, sur la **VM** on utilise SS et on observe la connexion netcat en cours :
```
tcp    ESTAB      0      0      192.168.127.10:apc-5454             192.168.127.                                         1:25643
```


# III. Routage statique

L'objectif de cette partie est de transformer nos PC en routeurs et de permettre au deuxième PC de se connecter à la machine du premier, et inversement.


```
Internet            Internet
    |                    |
  WiFi                  WiFi
    |                    |
   PC 1  ---Ethernet--- PC 2
    |                    |
Host-only 1          Host-only 2
    |                    |
   VM 1                 VM 2
```


# 1. Préparation des hôtes
Ok, alors la le principe c'est de modifier l'adresse de la carte HOST ONLY et l'adresse de la carte ETHERNET.
Nos cartes Ethernet sont dans le réseau : 192.168.112.0/30

PC1 : réseau 1 : 192.168.101.0/24


VM1 (sur PC1) : 192.168.101.10


PC2 : réseau 2 : 192.168.102.0/24


VM2 (sur PC2) : 192.168.102.10


# Check 

On check PC1 vers VM1 :


```
C:\Users\Ju'>ping 192.168.101.10

Envoi d’une requête 'Ping'  192.168.101.10 avec 32 octets de données :
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.101.10 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.101.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```


VM1 vers PC1 : 


```
[root@localhost ~]# ping 192.168.101.1
PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
64 bytes from 192.168.101.1: icmp_seq=1 ttl=64 time=0.224 ms
64 bytes from 192.168.101.1: icmp_seq=2 ttl=64 time=0.274 ms
64 bytes from 192.168.101.1: icmp_seq=3 ttl=64 time=0.291 ms
64 bytes from 192.168.101.1: icmp_seq=4 ttl=64 time=0.293 ms
64 bytes from 192.168.101.1: icmp_seq=5 ttl=64 time=0.290 ms
^C
--- 192.168.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4013ms
rtt min/avg/max/mdev = 0.224/0.274/0.293/0.030 ms
```

PC2 vers VM2 :

```
C:\Users\pierr>ping 192.168.102.10

Envoi d’une requête 'Ping'  192.168.102.10 avec 32 octets de données :
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.102.10 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.102.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```

VM2 vers PC2 :

```
[admin@localhost ~]$ ping 192.168.102.1
PING 192.168.102.1 (192.168.102.1) 56(84) bytes of data.
64 bytes from 192.168.102.1: icmp_seq=1 ttl=128 time=0.306 ms
64 bytes from 192.168.102.1: icmp_seq=2 ttl=128 time=0.579 ms
64 bytes from 192.168.102.1: icmp_seq=3 ttl=128 time=0.278 ms
64 bytes from 192.168.102.1: icmp_seq=4 ttl=128 time=0.562 ms
^C
--- 192.168.102.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3001ms
rtt min/avg/max/mdev = 0.278/0.431/0.579/0.140 ms
```


PC1 vers PC2 :


```
C:\Users\Ju'>ping 192.168.112.2

Envoi d’une requête 'Ping'  192.168.112.2 avec 32 octets de données :
Réponse de 192.168.112.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.112.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.112.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.112.2 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 192.168.112.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms

C:\Users\Ju'>
```


PC2 vers PC1 :


```
C:\Users\pierr>ping 192.168.112.1

Envoi d’une requête 'Ping'  192.168.112.1 avec 32 octets de données :
Réponse de 192.168.112.1 : octets=32 temps=1 ms TTL=64
Réponse de 192.168.112.1 : octets=32 temps=2 ms TTL=64
Réponse de 192.168.112.1 : octets=32 temps=2 ms TTL=64
Réponse de 192.168.112.1 : octets=32 temps=1 ms TTL=64

Statistiques Ping pour 192.168.112.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms

```


# Activation du routage sur les PCs

Vu que nous sommes sur Windows, il a fallut modifier la clé registre `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\ Services\Tcpip\Parameters\IPEnableRouter` en passant sa valeur de **0** à **1**

Ensuite, on a démarré le service après un reboot.

# 2. Configuration du routage

Bilan des adresses IP :

* PC1 :
    * Carte Ethernet : 192.168.112.1
    * Carte Host Only : 192.168.101.1
* VM1 :
    * Carte Host Only : 192.168.101.10
* PC2 :
    * Carte Ethernet : 192.168.112.2
    * Carte Host Only : 192.168.102.1
* VM2 :
    * Carte Host Only : 192.168.102.10


# Sur le PC1 :

On doit dire au PC1 comment accéder au réseau 2 :

`route add 192.168.102.0/24 mask 255.255.255.0 192.168.112.2`

Signifiant : " On ajoute une route pour aller sur l'adresse 192.168.102.0 en passant par le chemin 192.168.112.2 "

# Sur le PC2 :

On fait la chose inverse


# Sur la VM1

Pour lui dire d'accéder au réseau 12 en passant par 1:

`ip route add 192.168.112.0/24 via 192.168.101.1 dev enp0s8`

Pour lui dire d'accéder au réseau 2 :

`ip route add 192.168.102.0/24 via 192.168.101.1 dev enp0s8`

VM1 Ping l'adresse de PC2 dans le réseau 12 :

```
[root@localhost ~]# ping 192.168.112.1
PING 192.168.112.1 (192.168.112.1) 56(84) bytes of data.
64 bytes from 192.168.112.1: icmp_seq=1 ttl=63 time=0.781 ms
64 bytes from 192.168.112.1: icmp_seq=2 ttl=63 time=0.336 ms
64 bytes from 192.168.112.1: icmp_seq=3 ttl=63 time=0.278 ms
64 bytes from 192.168.112.1: icmp_seq=4 ttl=63 time=0.278 ms
64 bytes from 192.168.112.1: icmp_seq=5 ttl=63 time=0.271 ms
^C
--- 192.168.112.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4000ms
rtt min/avg/max/mdev = 0.271/0.388/0.781/0.199 ms
```

VM1 Ping l'adresse de PC2 dans le réseau 2 :

```
[root@localhost ~]# ping 192.168.102.1
PING 192.168.102.1 (192.168.102.1) 56(84) bytes of data.
64 bytes from 192.168.102.1: icmp_seq=1 ttl=126 time=2.12 ms
64 bytes from 192.168.102.1: icmp_seq=2 ttl=126 time=1.99 ms
64 bytes from 192.168.102.1: icmp_seq=3 ttl=126 time=1.99 ms
64 bytes from 192.168.102.1: icmp_seq=4 ttl=126 time=1.82 ms
64 bytes from 192.168.102.1: icmp_seq=5 ttl=126 time=1.83 ms
^C
--- 192.168.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4226ms
rtt min/avg/max/mdev = 1.829/1.955/2.126/0.112 ms
```

# Sur la VM2

Pareil que la VM1

# 3. Configuration des noms de domaine :

VM1 qui ping PC2 et VM2 :
```
[root@localhost etc]# ping pc2.tp3.b1
PING pc2 (192.168.112.2) 56(84) bytes of data.
64 bytes from pc2 (192.168.112.2): icmp_seq=1 ttl=127 time=1.81 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=2 ttl=127 time=1.60 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=3 ttl=127 time=1.49 ms
64 bytes from pc2 (192.168.112.2): icmp_seq=4 ttl=127 time=1.81 ms
^C
--- pc2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4009ms
rtt min/avg/max/mdev = 1.492/1.655/1.817/0.137 ms
[root@localhost etc]# ping vm2.tp3.b1
PING vm2 (192.168.102.10) 56(84) bytes of data.
64 bytes from vm2 (192.168.102.10): icmp_seq=1 ttl=62 time=2.59 ms
64 bytes from vm2 (192.168.102.10): icmp_seq=2 ttl=62 time=2.36 ms
64 bytes from vm2 (192.168.102.10): icmp_seq=3 ttl=62 time=2.25 ms
64 bytes from vm2 (192.168.102.10): icmp_seq=4 ttl=62 time=2.65 ms
^C
--- vm2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 2.253/2.468/2.658/0.164 ms
```

La configuration du fichier hosts sur PC1 :

```
#
127.0.0.1 localhost pc1 pc1.tp3.b1
::1 localhost
192.168.112.2 pc2 pc2.tp3.b1
192.168.102.10 vm2 vm2.tp3.b1
```

Le but ultime de netcat entre VM1 et VM2 :

```
[root@localhost etc]# nc -l 5454
Bonjour, voii^Hci^H^H^H^H^H^H^[[3~^[[3~^H^H^H^H^H^H^H
^H^H^H^H^H^H^H^Hslt
Voici slt
le test du netcat termin2
````

(Sachant que la VM2 a utilisé `nc vm1 5454`)

La VM1 peut désormais ping tous les FQDN (y compris vm1.tp3.b1)

De même pour la VM2 (pouvant ping vm2.tp3.b1)