# TP5 - Premier pas dans le monde Cisco

## I. Préparation du lab

### 1. Préparation VMs

La machine est bien configurée, la nouvelle carte "host-only" est parametrée.

`Cette carte host-only sert seulement à la connexion SSH.`

On a cloné le patron afin d'avoir les 3 VMs et on les rajoute dans GNS3 avec 2 interfaces réseau.


### 2. Préparation Routeurs Cisco

On telecharge la petite image disque **TOTALEMENT ILLEGALEMENT :D** du petit routeur Cisco afin de mettre en place les 2 routeurs.

Le réseau utilisé est 10.5.3.0/30 afin d'avoir comme IP possible 10.5.12.1 et 10.5.12.2 pour les deux routeurs.

Récapitulation des IPs:

| Machines       |    net1    |    net2    |    net3   |
|----------------|:----------:|:----------:|:---------:|
| client1.tp5.b1 |      X     |  10.5.2.10 |     X     |
| client2.tp5.b1 |      X     |  10.5.2.11 |     X     |
| router1.tp5.b1 | 10.5.1.254 |      X     | 10.5.12.1 |
| router2.tp5.b1 |      X     | 10.5.2.254 | 10.5.12.2 |
| server1.tp5.b1 |  10.5.1.10 |      X     |     X     |


## II. Lancement et configuration du lab

### Checklist IP VMs

Désactivation de SELinux et carte NAT déjà désactivée dans le patron.

Il faut ensuite définir les IPs statiques, les noms de domaines et avoir une connexion SSH, pour ca on fait comme dans les TPs précédents.

### Checklist IP Routeurs

Pour la définition des IPs statiques et des noms de domaines des routeurs `router1.tp5.b1` et `router2.tp5.b1` il faut également suivre la procédure indiquée.

On ouvre dans GNS3 le terminal.

Pour le `router1`:

```cisco
(config-if)# ip address 10.5.1.254 255.255.255.0
```

```cisco
(config-if)# hostname router1.tp5.b1
```

### Checklist routes

- [ ] `router1`:

  ```cisco
  (config)# ip route 10.5.2.0 255.255.255.0 10.5.12.2
  ```
 
 - [ ] `server1`:
 
  - Routes

    ```bash
    [root@server1 ~]# nano /etc/sysconfig/network-scripts/route-eth0

    10.2.0.0/24 via 10.2.0.254 dev eth0
    ```
  
  - `Hosts`
    
    ```bash
    [root@server1 ~]# nano /etc/hosts
    
    10.5.2.10 client1 client1.tp5.b1
    10.5.2.11 client2 client2.tp5.b1
    ```

Depuis `client2`:

```bash
[root@client2 ~]# ping client1.tp5.b1 -c 2 && echo &&  ping server1.tp5.b1 -c 2
PING client1 (10.5.2.10) 56(84) bytes of data.
64 bytes from client1 (10.5.2.10): icmp_seq=1 ttl=64 time=6.56 ms
64 bytes from client1 (10.5.2.10): icmp_seq=2 ttl=64 time=4.80 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.830/5.683/6.537/0.856 ms

PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=27.8 ms
64 bytes from server1 (10.5.1.10): icmp_seq=2 ttl=62 time=33.8 ms

--- server1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 27.911/30.842/33.774/2.936 ms
```

## III. DHCP
### 1. Mise en place du serveur DHCP
#### 1. Renommer la machine

On renomme la machine en permanent

```bash
[root@client2 ~]# nano /etc/hostname
hostname dhcp-net2.tp5.b1
```

#### 2. Installer le serveur DHCP
On active la carte NAT `ifup eth2`

```bash
[root@client2 ~]# ip a show dev eth2
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:00:00:05:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.17/24 brd 192.168.20.255 scope global noprefixroute dynamic eth2
       valid_lft 7143sec preferred_lft 7143sec
    inet6 fe80::250:ff:fe00:502/64 scope link 
       valid_lft forever preferred_lft forever
```

On installe le package DHCP sur les machines

```bash
[root@dhcp-net2 ~]# yum install -y dhcp
```

```bash
[root@dhcp-net2 ~]# ifup eth0 && ifdown eth2
```

#### 5. Démarrer le serveur DHCP

On modife le fichier de configuration puis on le lance

```bash
[root@dhcp-net2 ~]# systemctl start dhcpd
```

Pour le démarrage automatique au démarrage de la machine on rajoute cette ligne :

```bash
[root@dhcp-net2 ~]# systemctl enable dhcpd
```
On vérifie ensuite que le service est bien lancé

```bash
[root@dhcp-net2 ~]# systemctl status dhcpd -l
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since mer. 2019-02-20 17:27:20 CET; 2min 48s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 3881 (dhcpd)
   Status: "Dispatching packets..."
   CGroup: /system.slice/dhcpd.service
           └─3881 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Listening on LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   Socket/fallback/fallback-net
févr. 20 17:27:20 dhcp-net2.tp5.b1 systemd[1]: Started DHCPv4 Server Daemon.
```

#### 6. Faire un test
En utilisant la VM `client1.tp5.b1`:
- Modification du fichier `/etc/sysconfig/network-scripts/ifcfg-eth0` pour une modification permanante
- `dhclient -v` pour demander une adresse ip.

```bash
[root@client1 ~]# dhclient -v eth0
Internet Systems Consortium DHCP Client 4.2.5
Copyright 2004-2013 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/00:50:00:00:04:00
Sending on   LPF/eth0/00:50:00:00:04:00
Sending on   Socket/fallback
DHCPDISCOVER on eth0 to 255.255.255.255 port 67 interval 3 (xid=0x6ad2d7)
DHCPREQUEST on eth0 to 255.255.255.255 port 67 (xid=0x6ad2d7)
DHCPOFFER from 10.5.2.11
DHCPACK from 10.5.2.11 (xid=0x6ad2d7)
bound to 10.5.2.50 -- renewal in 254 seconds.
```

### 2. Explorer un peu DHCP

On va effectuer un `dhcpclient -v -r` pendant une capture afin de récupérer les trames correspondant au serveur DHCP:

```bash
[root@client1 ~]# dhclient -r
```

Les trames en provenance de l'adresse MAC `50:00:00:07:00:01` correspondents à la carte `eth1` qui n'a jamais été utilisé pendant le TP:

```bash
[root@client1 ~]# ip a s dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:00:00:04:01 brd ff:ff:ff:ff:ff:ff
```
Lien vers le [fichier Wireshark](https://github.com/JulienCASTERA/CCNA_1/blob/master/tp5/wireshark_dhcp.pcapng).

Sur le fichier de capture on peut observer les trames entre le serveur et le client DHCP (`client1.tp5.b1` et `dhcp-net2.tp5.b1`).

## IV. Bonus
### 1. Installer un serveur web
#### 1.1 Installation du serveur

Sur le `server1.tp5.b1`

```bash
# Allumer l'interface NAT
ifup eth2

# Installation de nginx
yum install -y nginx

# Ouverture du port firewall
firewall-cmd --add-port=80/tcp
firewall-cmd --reload

# Lancement du serveur web
systemctl start nginx

# Eteindre l'interface NAT
ifdown eth2
```

Vérification:

```bash
[root@server1 ~]# systemctl status -l nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/nginx.service.d
           └─override.conf
   Active: active (running) since mer. 2019-02-20 19:13:07 CET; 1min 2s ago
  Process: 7182 ExecStartPost=/bin/sleep 0.1 (code=exited, status=0/SUCCESS)
  Process: 7179 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 7177 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 7175 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 7181 (nginx)
   CGroup: /system.slice/nginx.service
           ├─7181 nginx: master process /usr/sbin/ngin
           └─7183 nginx: worker proces

févr. 20 19:13:07 server1.tp5.b1 systemd[1]: Stopped The nginx HTTP and reverse proxy server.
févr. 20 19:13:07 server1.tp5.b1 systemd[1]: Starting The nginx HTTP and reverse proxy server...
févr. 20 19:13:07 server1.tp5.b1 nginx[7177]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
févr. 20 19:13:07 server1.tp5.b1 nginx[7177]: nginx: configuration file /etc/nginx/nginx.conf test is successful
févr. 20 19:13:07 server1.tp5.b1 systemd[1]: Started The nginx HTTP and reverse proxy server.
```

#### 1.2 Test depuis le `client1.tp5.b1`:

```bash
[root@client1 ~]# curl server1.tp5.b1 > curl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3700  100  3700    0     0  15210      0 --:--:-- --:--:-- --:--:-- 15352
```


On y est !
```
    <body>
        <h1>Welcome to <strong>nginx</strong> on Fedora!</h1>

        <div class="content">
            <p>This page is used to test the proper operation of the
            <strong>nginx</strong> HTTP server after it has been
            installed. If you can read this page, it means that the
            web server installed at this site is working
            properly.</p>

            <div class="alert">
                <h2>Website Administrator</h2>
                <div class="content">
                    <p>This is the default <tt>index.html</tt> page that
                    is distributed with <strong>nginx</strong> on
                    Fedora.  It is located in
                    <tt>/usr/share/nginx/html</tt>.</p>

                    <p>You should now put your content in a location of
                    your choice and edit the <tt>root</tt> configuration
                    directive in the <strong>nginx</strong>
                    configuration file
                    <tt>/etc/nginx/nginx.conf</tt>.</p>

                </div>
            </div>

            <div class="logos">
                <a href="http://nginx.net/"><img
                    src="nginx-logo.png" 
                    alt="[ Powered by nginx ]"
                    width="121" height="32" /></a>

                <a href="http://fedoraproject.org/"><img 
                    src="poweredby.png" 
                    alt="[ Powered by Fedora ]" 
                    width="88" height="31" /></a>
            </div>
        </div>
    </body>
</html>
```

### 2. Clé SSH
#### 2.1 Sur `client1.tp5.b1`
Création des clés publique et privée:

```bash
[root@client1 ~]# ssh-keygen
```

```bash
[root@client1 ~]# ls ~/.ssh/
id_rsa	id_rsa.pub
```

`id_rsa` est la clé privée et `id_rsa.pub` la clé publique qui sont stockées dans le répertoire `~/.ssh/`.\
Lors de la création des clés il est possible de choisir de protéger la clé privée par un mot de passe ou non.

Puis il faut donner au `server1.tp5.b1` la clé publique pour permettre l'identification.

```bash
[root@client1 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@server1.tp5.b1
```

On teste ensuite le fonctionnement de l'identification par clés:

```bash
[root@client1 ~]# ssh -i ~/.ssh/id_rsa root@server1.tp5.b1
Last login: Wed Feb 20 22:01:24 2019 from 10.5.2.50
[root@server1 ~]# 
```

#### 2.2 Sur `server1.tp5.b1`

Comme seul l'utilisateur `root` est utilisé pour plus de sécurité il est possible de remplacer dans le fichier `/etc/ssh/sshd_config` `#PermitRootLogin yes` par `PermitRootLogin without-password` afin d'autoriser la connexion SSH pour l'utilisateur `root` uniquement en utilisant uniquement une identification par clés et plus par mot de passe.
