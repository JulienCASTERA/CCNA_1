
# TP Réseau

## I. Exploration locale en solo

**1. Affichage d’informations sur la pile TCP / IP locale**
 

En utilisant la ligne de commande :

  
  
`ipconfig /all`

*  **Interface wifi** :

* Nom : Killer Wireless-n/a/ac 1535 Wireless Network Adapter

* Adresse IP : 10.33.0.211

* Adresse MAC : 9C-B6-D0-1F-EE-D5

* Adresse Réseau : 10.33.0.0

* Adresse Broadcast : 10.33.255.255

  

*  **Interface Ethernet** :

* Nom : Killer E2500 Gigabit Ethernet Controller

* Adresse IP : None

* Adresse MAC : 90-20-31-1B-46-AD

  

> Pour l'interface ethernet, n'étant pas connecté par cable, il n'y a **pas d'adresse réseau et de broadcast**.

  

Pour trouver la passerelle par défaut de notre carte WiFi il faut taper cette commande sur Windows.

  
  

`ipconfig | findstr /i "Passerelle"`

  

**En graphique (GUI : Graphical User Interface)**

Sur Windows, il suffit d'aller comme indiqué sur le screenshot.

![Image du GUI](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/ipmacgatewayGUI.PNG)

  

La gateway dans le réseau d'Ingésup sert à relier le réseau à un autre réseau comme **Internet**.

  

**2. Modifications des informations**

*  **A - Modification d'adresse IP - pt.1**

*  **Réseau Ynov** :

* Première IP : 10.33.0.1

* Dernière IP : 10.33.3.254

  

**Changer d'adresse IP nôtre carte WiFi pour une autre sur le même réseau d'Ynov**

![Image du changement](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/changerip.PNG)

![Image du changement done](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/changeripdone.PNG)

  

*  **B - nmap**

  
  

Pour utiliser nmap et scanner le réseau d'Ynov, il va falloir rentrer cette commande dans le terminal.

  
  

`nmap -sn -PE 10.33.0.0/22`

  
  

Cela nous donne à la fin du scan tous les 'clients' détectés sur le réseau.

  

Par exemple :

> MAC Address: 34:F3:9A:74:6F:4E (Intel Corporate)

  
  

> Nmap scan report for 10.33.2.172

  
  

> Host is up (0.14s latency).

  
  

> MAC Address: 6C:96:CF:DF:C5:63 (Apple)

  

> Nmap scan report for 10.33.2.175

  

> Host is up (0.39s latency).

  

> MAC Address: E4:CE:8F:2C:76:14 (Apple)

  

> Nmap scan report for 10.33.2.183

  

> Host is up (0.33s latency).

  

> MAC Address: B8:8A:60:5A:6C:11 (Intel Corporate)

  

> Nmap scan report for 10.33.2.185

  

> Host is up (0.37s latency).

  

*  **C - Modification d'adresse IP - pt.2**

  
  

On a ensuite modifié notre adresse IP comme ceci :

![Image du changement done](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/ipmacgatewayGUI.PNG)

  

## II. Exploration locale en duo

  

Après avoir brancé le câble Ethernet aux PCs on peut commencer.

*  **Création du réseau**

  
On définit une adresse IP sur nos cartes réseau afin que notre réseau existe.

Sur notre carte Ethernet nous avons modifié comme suis :

*  **Réseau local** :

* Adresse IP PC1 : 172.16.1.1

* Adresse IP PC2 : 172.16.1.2

* Masque de réseau : 255.255.255.252

  
  

On vérifie grâce à cette commande que nos changements ont pris effet :

  
  

`ipconfig /all`

  
  

![Image du réseau local](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/duo_changeip.PNG)

  
  

Après s'être assuré que nos deux ordinateurs ont leur adresse IP bien configurée, on fait un ping sur un PC :

  
  

`ping 172.16.1.1`

  

![Ping local](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/pinglocal.PNG)

  
  

Le ping marche, c'est que nous sommes connectés.

*  **Utilisation d'un des deux comme gateway**

On désactive l'interface WiFi sur l'un des deux postes,
les PCs sont bel et bien connectés par ethernet.
* Sur le PC qui n'a plus internet on a défini comme passerelle l'adresse IP de l'autre PC :
![Passerelle](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/duo_connected.PNG)

* Sur le PC qui a toujours internet on a mis le partage de connexion sur la carte wifi connecté à la connexion Ethernet.

Pour tester la connectivité on s'est pingé, avec une réponse des deux côtés.

Malgré ça, la connexion à internet demeure impossible sur un navigateur. (Vive Windaube) 

Du coup, on a modifié le DNS sur le PC n'ayant pas internet par 
* DNS Primaire : 8.8.8.8
* DNS Secondaire : 8.8.4.4

Une fois ceci fait, la connexion à Google est possible.
Le partage de connexion fonctionne désormais.

*  **Petit chat privé ?** :
Une fois netcat installé sur Windows, on a tapé les commandes suivantes pour communiquer :

	* Sur le PC serveur : `nc.exe -l -p 80`
	* Sur le PC client : `nc.exe 192.168.137.1 80`
	
Une fois ceci fait nous pouvons parler sur le CMD ! Youhouu
![Connected](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/duo_netcat.PNG)

*  **Wireshark** :
	* Les trames circulant pendant un ping :
![Trames](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/duo_wping.PNG)

	* Les trames circulant pendant un netcat :
![Trames](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/duo_wnetcat.PNG)

	* Les trames circulant pendant que le PC2 se connecte à internet par la gateway du PC1 :
![Trames](https://raw.githubusercontent.com/JulienCASTERA/CCNA_1/master/tp2/images/duo_wgateway.PNG)

*  **Firewall** :
	* Pour activer le ping sur le firewall de Windows on tape cette commande la sur le CMD :
`netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow`

	* Pour activer le netcat on est allé sur le pare-feu Windows afin d'activer le port 8888 en port local.
On autorise tous les ports en port distant car on ne connais pas le port de connexion du pc distant.

Le netcat sous Firewall marche !

# III. Manipulations d'autres outils/protocoles côté client

*  **DHCP** :
	* Serveur DHCP du réseau WiFi : 10.33.3.254
	* Bail obtenu. . . . . . . . . . . . . . : jeudi 20 décembre 2018 12:21:16
    * Bail expirant. . . . . . . . . . . . . : jeudi 20 décembre 2018 13:21:15
	* La commande pour changer d'IP : `ipconfig /renew Wi-Fi`
* **DNS** :
	* Adresse IP du serveur DNS : 
		* Serveurs DNS : 10.33.10.20
		* 10.33.10.7
   		* 8.8.8.8
	* NSlookup :
		* Google : `Nom :    google.com
		Addresses:  2a00:1450:4007:816::200e
          172.217.19.238`
		* Ynov : `Nom :    ynov.com
		Address:  217.70.184.38`
	* Reverse Lookup :
		* 78.78.21.21 :
		`Nom :    host-78-78-21-21.mobileonline.telia.com
		Address:  78.78.21.21`

		
		Pour cette IP, on vois que l'host est un site internet nommé "Telia" 

		* 92.16.54.88 :
		`Nom :    host-92-16-54-88.as13285.net
		Address:  92.16.54.88` 


		Et pour celle la, on ne sais pas à qui elle appartient mais elle existe.

