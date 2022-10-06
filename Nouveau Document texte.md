🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

- les deux IPs choisies, en précisant le masque : 10.10.10.1/22 et 10.10.10.2/22
- l'adresse de réseau : 10.10.8.0
- l'adresse de broadcast : 10.10.11.255
-commandes utilisées pour définir les adresses IP *via* la ligne de commande :

```
 PS C:\Windows\system32> New-NetIPAddress -InterfaceIndex 14 -IPAddress 10.10.10.1 -AddressFamily IPv4 -PrefixLength 22
```

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

```
PS C:\Windows\system32> ping 10.10.10.2

Envoi d’une requête 'Ping'  10.10.10.2 avec 32 octets de données :
Réponse de 10.10.10.2 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.2 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.2 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.2 : octets=32 temps=1 ms TTL=64

Statistiques Ping pour 10.10.10.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms
```

🌞 **Wireshark it**

echo request : 8
echo reply : 0  

![ping](https://github.com/SmlDreams/tp-reseau/blob/main/WireShark/ping.pcapng)

🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP**

# II. ARP my bro


🌞 **Check the ARP table**

```
PS C:\Windows\system32> arp -a

Interface : 10.10.10.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.10.10.2            00-e0-4c-68-00-33     dynamique

Interface : 10.33.16.205 --- 0x11
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

🌞 **Manipuler la table ARP**

```
PS C:\Windows\system32> arp -d
```

```
PS C:\Windows\system32> arp -a

Interface : 10.10.10.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.16.205 --- 0x11
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.56.1 --- 0x33
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
```

```
PS C:\Windows\system32> arp -a

Interface : 10.10.10.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.10.10.2            00-e0-4c-68-00-33     dynamique
  10.10.11.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.16.205 --- 0x11
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  10.33.19.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.56.1 --- 0x33
  Adresse Internet      Adresse physique      Type
  192.168.56.255        ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
```

🌞 **Wireshark it**

la première Tram à pour :
-source : 
    -10.10.10.1;d8:bb:c1:f4:60:61 
-et pour destination : 
    -10.10.10.2;00:e0:4c:68:00:33

la deuxieme Tram à pour :
-source : 
    -10.10.10.2;00:e0:4c:68:00:33 
-et pour destination :  
    -10.10.10.1;d8:bb:c1:f4:60:61
🦈 **PCAP qui contient les trames ARP**

![Arp](https://github.com/SmlDreams/tp-reseau/blob/main/WireShark/ARP.pcapng)

adresse : 
10.10.10.1;d8:bb:c1:f4:60:61 fait la demande ARP.
tandis que l'adresse :
10.10.10.2;00:e0:4c:68:00:33 renvoie la réponse.

# III. DHCP you too my brooo

🌞 **Wireshark it**

![DHCP](https://github.com/SmlDreams/tp-reseau/blob/main/WireShark/requ%C3%AAte%20DHCP.pcapng)

***Discover : ***

  -ip Src : 
    0.0.0.0
  -ip Dst :
    255.255.255.255

***Offer :***

  -ip Src :
    10.33.19.254
  -ip Dst :
    255.255.255.255

***Request :***

  -ip Src :
    0.0.0.0
  -ip Dst :
    255.255.255.255
***Ack :***

  -ip Src :
    10.33.19.254
  -ip Dst:
    255.255.255.255

**ip To use : 10.33.19.254**

**ip gateway network : 255.255.255.255**

**ip DNS : 8.8.8.8 & 8.8.4.4 & 1.1.1.1**
