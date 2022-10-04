## 1. Affichage d'informations sur la pile TCP/IP locale

### En ligne de commande

En utilisant la ligne de commande (CLI) de votre OS :

**🌞 Affichez les infos des cartes réseau de votre PC**

```
PS C:\Users\quentin> ipconfig /All

Carte réseau sans fil Wi-Fi :

   Adresse physique . . . . . . . . . . . : 88-D8-2E-EB-70-AD
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.175(préféré)
```
```
PS C:\Users\quentin> ipconfig /All

Carte Ethernet Ethernet :

    Adresse physique . . . . . . . . . . . : D8-BB-C1-F4-60-61

```

**🌞 Affichez votre gateway**

```
PS C:\Users\quentin> ipconfig /All

   Passerelle par défaut. . . . . . . . . : 10.33.19.254

```

**🌞 Déterminer la MAC de la passerelle**

```
PS C:\Users\quentin> arp -a

    10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

En utilisant l'interface graphique de votre OS :  

**🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

```
Nom :	Wi-Fi
Adresse physique (MAC) :	88:d8:2e:eb:70:ad
Adresse IPv4 :	10.33.16.175/22
Passerelle par défaut IPv4 :	10.33.19.254
```

## 2. Modifications des informations

### A. Modification d'adresse IP (part 1)  

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

j'ai modifié mon adresse grâce au panneau de configuration
maintenant :
```
PS C:\Users\quentin> ipconfig /All

     Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.175
```

🌞 **Il est possible que vous perdiez l'accès internet.** Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.

en changeant d'adresse il se peut que l'on se trouve couper a internet car cette adresse est déjà utilisé et le réseau envoie un packet au premier client.

## 3. Modification d'adresse IP


🌞 Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau

je suis passé encore par le panneau de configuration

🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**

```
PS C:\Users\quentin> ipconfig /All

     Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.77
```

🌞 **Vérifier que les deux machines se joignent**

```
PS C:\Users\quentin> ping 10.10.10.27

Envoi d’une requête 'Ping'  10.10.10.27 avec 32 octets de données :
Réponse de 10.10.10.27 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.27 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.27 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.27 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 10.10.10.27:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 2ms, Moyenne = 2ms
```

🌞 **Déterminer l'adresse MAC de votre correspondant**

```
arp -a

 10.10.10.27           00-68-eb-59-ae-90     dynamique
```

## 4. Utilisation d'un des deux comme gateway

🌞**Tester l'accès internet**

```
PS C:\Users\quentin> ping 192.168.137.1

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps<1ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps<1ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps<1ms TTL=128

Statistiques Ping pour 192.168.137.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 1ms, Moyenne = 0ms


PS C:\Users\quentin> ping 8.8.8.8

Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
Réponse de 8.8.8.8 : octets=32 temps=25 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=24 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=26 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=28 ms TTL=113

Statistiques Ping pour 8.8.8.8:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 24ms, Maximum = 28ms, Moyenne = 25ms

Envoi d’une requête 'ping' sur google.com [172.217.18.206] avec 32 octets de données :
Réponse de 172.217.18.206 : octets=32 temps=29 ms TTL=114
Réponse de 172.217.18.206 : octets=32 temps=31 ms TTL=114
Réponse de 172.217.18.206 : octets=32 temps=26 ms TTL=114
Réponse de 172.217.18.206 : octets=32 temps=25 ms TTL=114

Statistiques Ping pour 172.217.18.206:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 25ms, Maximum = 31ms, Moyenne = 27ms
```

🌞 **Prouver que la connexion Internet passe bien par l'autre PC**

```
PS C:\Users\quentin> tracert -4 1.1.1.1

Détermination de l’itinéraire vers one.one.one.one [1.1.1.1]
avec un maximum de 30 sauts :

  1     3 ms     3 ms     2 ms  LAPTOP-R8S29PG0 [192.168.137.1]
  2     *        *        *     Délai d’attente de la demande dépassé.
  3    10 ms     8 ms     9 ms  10.33.19.254
  4     7 ms     6 ms     4 ms  137.149.196.77.rev.sfr.net [77.196.149.137]
  5    12 ms    11 ms    10 ms  108.97.30.212.rev.sfr.net [212.30.97.108]
  6    24 ms    25 ms    27 ms  222.172.136.77.rev.sfr.net [77.136.172.222]
  7    22 ms    23 ms    23 ms  77.136.172.221
  8    27 ms    24 ms    25 ms  221.10.136.77.rev.sfr.net [77.136.10.221]
  9    25 ms    26 ms    23 ms  221.10.136.77.rev.sfr.net [77.136.10.221]
 10    48 ms    26 ms    28 ms  141.101.67.254
 11    27 ms   114 ms    31 ms  172.71.128.2
 12    23 ms    23 ms    28 ms  one.one.one.one [1.1.1.1]
Itinéraire déterminé.
```

## 5. Petit chat privé

🌞 **sur le PC *serveur***

```
PS C:\Users\lebou\Documents\B1\Tp reseau\netcat-1.11> ./nc64.exe -l -p 8888
coucou
salut
ca va
```

🌞 **sur le PC *client***

```
PS C:\Users\quentin\Desktop\netcat-1.11> .\nc.exe 10.33.17.26 8888
coucou
salut
ca va
```

🌞 **Visualiser la connexion en cours**

```
PS C:\Windows\system32> netstat -a -n -b | Select-String 8888

  TCP    10.33.17.91:61170      10.33.17.26:8888       ESTABLISHED
```

🌞 **Pour aller un peu plus loin**

```
S C:\Users\lebou\Documents\B1\Tp reseau\netcat-1.11> ./nc64.exe -l -p 8888 -s 192.168.137.1
PS C:\WINDOWS\system32> netstat -a -n -b | Select-String 8888

  TCP    192.168.137.1:8888     0.0.0.0:0              LISTENING 
```

## 6. Firewall

Toujours par 2.

Le but est de configurer votre firewall plutôt que de le désactiver

🌞 **Activez et configurez votre firewall**

```
PS C:\Windows\system32> ping 8.8.8.8

Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=114
Réponse de 8.8.8.8 : octets=32 temps=22 ms TTL=114
Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=114
Réponse de 8.8.8.8 : octets=32 temps=21 ms TTL=114

Statistiques Ping pour 8.8.8.8:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 20ms, Maximum = 22ms, Moyenne = 20ms
```

```
PS C:\Users\quentin\Desktop\netcat-1.11> .\nc.exe 10.33.17.26 8888
ccddggggccgr
ok,
dkdd
```
# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP


🌞**Exploration du DHCP, depuis votre PC**

```
PS C:\Users\quentin\Desktop\netcat-1.11> ipconfig /all

  Bail expirant. . . . . . . . . . . . . : mercredi 5 octobre 2022 12:51:33
  Serveur DHCP . . . . . . . . . . . . . : 10.33.19.254
```

## 2. DNS

🌞** Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

```
PS C:\Users\quentin\Desktop\netcat-1.11> ipconfig /all

  Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
```

🌞 Utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main

```
PS C:\Users\quentin\Desktop\netcat-1.11> nslookup.exe
Serveur par dÚfaut :   LAPTOP-R8S29PG0
Address:  fe80::3cb7:857a:66c8:e053

> google.com
Serveur :   LAPTOP-R8S29PG0
Address:  fe80::3cb7:857a:66c8:e053

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:813::200e
          216.58.214.78

> ynov.com
Serveur :   LAPTOP-R8S29PG0
Address:  fe80::3cb7:857a:66c8:e053

Réponse ne faisant pas autorité :
Nom :    ynov.com
Addresses:  2606:4700:20::681a:ae9
          2606:4700:20::681a:be9
          2606:4700:20::ac43:4ae2
          104.26.10.233
          172.67.74.226
          104.26.11.233

> 231.34.113.12
Serveur :   LAPTOP-R8S29PG0
Address:  fe80::3cb7:857a:66c8:e053

*** LAPTOP-R8S29PG0 ne parvient pas à trouver 231.34.113.12 : Non-existent domain
> 78.34.2.17
Serveur :   LAPTOP-R8S29PG0
Address:  fe80::3cb7:857a:66c8:e053

Nom :    cable-78-34-2-17.nc.de
Address:  78.34.2.17
```

il y a plusieurs adresse Ip quand on questionne le serveur de ynov car il nous renvoie les 3 adresse des 3 serveurs existants.

le serveur DNS ne considère pas l'adresse Ip 231.34.113.12 comme un domaine.

# IV. Wireshark

![netcat](https://github.com/SmlDreams/tp-reseau/pics/netcat.png)


![ping 8.8.8.8](https://github.com/SmlDreams/tp-reseau/pics/ping%208.8.8.8.png)

![requête DNS](https://github.com/SmlDreams/tp-reseau/pics/requete%20DNS%20google.com.png)

