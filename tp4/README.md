# TP4 : TCP, UDP et services réseau

# I. First steps

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

Discord :
    -ip dst : 162.159.133.234
    -port src : 443
    -port dst : 58408
    
    [tcp discord]()

    ```
     TCP    10.33.16.205:58408     162.159.133.234:443    ESTABLISHED
    [Discord.exe]
    ```

Brave :
    -ip dst : 151.101.18.167
    -port src : 443
    -port dst : 54471
    
    [tcp brave]() METTRE LA CAPTURE

    ```
     TCP    10.33.16.205:54471     151.101.18.167:443     ESTABLISHED
    [brave.exe]
    ```

rocket league :

    [udp rocket]()

    ```
      UDP    0.0.0.0:56227          *:*
    [RocketLeague.exe]
    ```

    rocketLeague.exe is listening for client at port 56227 on every Interfaces


teléchargement STEAM :
    -ip dst : 185.25.182.77
    -port dst : 60446
    -port src : 443

    [tcp tel steam]()

    ```
     TCP    10.33.16.205:60419     185.25.182.77:443    ESTABLISHED
    [steam.exe]
    ```

live twitch :
    -ip dst : 18.164.56.133
    -port dst : 54540 
    -port src : 443

    [tcp live twitch]()

    ```
     TCP    10.33.16.205:54540     18.164.56.133:443      ESTABLISHED
    [brave.exe]
    ```

# II. Mise en place

## 1. SSH

🌞 **Examinez le trafic dans Wireshark**

- SSH utilise du TCP normal puisqu'on parle quand même d'une connection à un utilisateur d'un autre lan 

[3-Way Handshake]()
[packets ssh]()
[FIN ssh]()

- entre le *3-way handshake* et l'échange `FIN`, c'est juste une bouillie de caca chiffré, dans un tunnel TCP

🌞 **Demandez aux OS**

- repérez, avec une commande adaptée (`netstat` ou `ss`), la connexion SSH depuis votre machine
- ET repérez la connexion SSH depuis votre VM

🦈 **Je veux une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion**

## 2. Routage

Ouais, un peu de répétition, ça fait jamais de mal. On va créer une machine qui sera notre routeur, et **permettra à toutes les autres machines du réseau d'avoir Internet.**

🖥️ **Machine `router.tp4.b1`**

- n'oubliez pas de dérouler la checklist (voir [les prérequis du TP](#0-prérequis))
- donnez lui l'adresse IP `10.4.1.11/24` sur sa carte host-only
- ajoutez-lui une carte NAT, qui permettra de donner Internet aux autres machines du réseau
- référez-vous au TP précédent

> Rien à remettre dans le compte-rendu pour cette partie.

# III. DNS

## 1. Présentation

Un serveur DNS est un serveur qui est capable de répondre à des requêtes DNS.

Une requête DNS est la requête effectuée par une machine lorsqu'elle souhaite connaître l'adresse IP d'une machine, lorsqu'elle connaît son nom.

Par exemple, si vous ouvrez un navigateur web et saisissez `https://www.google.com` alors une requête DNS est automatiquement effectuée par votre PC pour déterminez à quelle adresse IP correspond le nom `www.google.com`.

> La partie `https://` ne fait pas partie du nom de domaine, ça indique simplement au navigateur la méthode de connexion. Ici, c'est HTTPS.

Dans cette partie, on va monter une VM qui porte un serveur DNS. Ce dernier répondra aux autres VMs du LAN quand elles auront besoin de connaître des noms. Ainsi, ce serveur pourra :

- résoudre des noms locaux
  - vous pourrez `ping node1.tp4.b1` et ça fonctionnera
  - mais aussi `ping www.google.com` et votre serveur DNS sera capable de le résoudre aussi

*Dans la vraie vie, il n'est pas rare qu'une entreprise gère elle-même ses noms de domaine, voire gère elle-même son serveur DNS. C'est donc du savoir ré-utilisable pour tous qu'on voit ici.*

> En réalité, ce n'est pas votre serveur DNS qui pourra résoudre `www.google.com`, mais il sera capable de *forward* (faire passer) votre requête à un autre serveur DNS qui lui, connaît la réponse.

![Haiku DNS](./pics/haiku_dns.png)

## 2. Setup

🖥️ **Machine `dns-server.tp4.b1`**

- n'oubliez pas de dérouler la checklist (voir [les prérequis du TP](#0-prérequis))
- donnez lui l'adresse IP `10.4.1.201/24`

Installation du serveur DNS :

```bash
# assurez-vous que votre machine est à jour
$ sudo dnf update -y

# installation du serveur DNS, son p'tit nom c'est BIND9
$ sudo dnf install -y bind bind-utils
```

La configuration du serveur DNS va se faire dans 3 fichiers essentiellement :

- **un fichier de configuration principal**
  - `/etc/named.conf`
  - on définit les trucs généraux, comme les adresses IP et le port où on veu écouter
  - on définit aussi un chemin vers les autres fichiers, les fichiers de zone
- **un fichier de zone**
  - `/var/named/tp4.b1.db`
  - je vous préviens, la syntaxe fait mal
  - on peut y définir des correspondances `IP ---> nom`
- **un fichier de zone inverse**
  - `/var/named/tp4.b1.rev`
  - on peut y définir des correspondances `nom ---> IP`

➜ **Allooooons-y, fichier de conf principal**

```bash
# éditez le fichier de config principal pour qu'il ressemble à :
$ sudo cat /etc/named.conf
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
[...]
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        recursion yes;
[...]
# référence vers notre fichier de zone
zone "tp4.b1" IN {
     type master;
     file "tp4.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp4.b1.rev";
     allow-update { none; };
     allow-query { any; };
};
```

➜ **Et pour les fichiers de zone**

```bash
# Fichier de zone pour nom -> IP

$ sudo cat /var/named/tp4.b1.db

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns-server IN A 10.4.1.201
node1      IN A 10.4.1.11
```

```bash
# Fichier de zone inverse pour IP -> nom

$ sudo cat /var/named/tp4.b1.rev

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

;Reverse lookup for Name Server
201 IN PTR dns-server.tp4.b1.
11 IN PTR node1.tp4.b1.
```

➜ **Une fois ces 3 fichiers en place, démarrez le service DNS**

```bash
# Démarrez le service tout de suite
$ sudo systemctl start named

# Faire en sorte que le service démarre tout seul quand la VM s'allume
$ sudo systemctl enable named

# Obtenir des infos sur le service
$ sudo systemctl status named

# Obtenir des logs en cas de probème
$ sudo journalctl -xe -u named
```

🌞 **Dans le rendu, je veux**

- un `cat` des fichiers de conf
- un `systemctl status named` qui prouve que le service tourne bien
- une commande `ss` qui prouve que le service écoute bien sur un port

🌞 **Ouvrez le bon port dans le firewall**

- grâce à la commande `ss` vous devrez avoir repéré sur quel port tourne le service
  - vous l'avez écrit dans la conf aussi toute façon :)
- ouvrez ce port dans le firewall de la machine `dns-server.tp4.b1` (voir le mémo réseau Rocky)

## 3. Test

🌞 **Sur la machine `node1.tp4.b1`**

- configurez la machine pour qu'elle utilise votre serveur DNS quand elle a besoin de résoudre des noms
- assurez vous que vous pouvez :
  - résoudre des noms comme `node1.tp4.b1` et `dns-server.tp4.b1`
  - mais aussi des noms comme `www.google.com`

🌞 **Sur votre PC**

- utilisez une commande pour résoudre le nom `node1.tp4.b1` en utilisant `10.4.1.201` comme serveur DNS

> Le fait que votre serveur DNS puisse résoudre un nom comme `www.google.com`, ça s'appelle la récursivité et c'est activé avec la ligne `recursion yes;` dans le fichier de conf.

🦈 **Capture d'une requête DNS vers le nom `node1.tp4.b1` ainsi que la réponse**