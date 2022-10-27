# TP4 : TCP, UDP et services réseau

# I. First steps

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

Discord :
    -ip dst : 162.159.133.234
    -port src : 443
    -port dst : 58408
    
  [tcp discord](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/tcp%20discord.pcapng)

    ```
     TCP    10.33.16.205:58408     162.159.133.234:443    ESTABLISHED
    [Discord.exe]
    ```

Brave :
    -ip dst : 151.101.18.167
    -port src : 443
    -port dst : 54471
    
  [tcp brave](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/tcp%20brave.pcapng) METTRE LA CAPTURE

    ```
     TCP    10.33.16.205:54471     151.101.18.167:443     ESTABLISHED
    [brave.exe]
    ```

rocket league :

  [udp rocket](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/tcp%20rocket%20league.pcapng)

    ```
      UDP    0.0.0.0:56227          *:*
    [RocketLeague.exe]
    ```

    rocketLeague.exe is listening for client at port 56227 on every Interfaces


teléchargement STEAM :
    -ip dst : 185.25.182.77
    -port dst : 60446
    -port src : 443

  [tcp tel steam](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/tcp%20tel%20steam.pcapng)

    ```
     TCP    10.33.16.205:60419     185.25.182.77:443    ESTABLISHED
    [steam.exe]
    ```

live twitch :
    -ip dst : 18.164.56.133
    -port dst : 54540 
    -port src : 443

  [tcp live twitch](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/tcp%20live%20twitch.pcapng)

    ```
     TCP    10.33.16.205:54540     18.164.56.133:443      ESTABLISHED
    [brave.exe]
    ```

# II. Mise en place

## 1. SSH

🌞 **Examinez le trafic dans Wireshark**

- SSH utilise du TCP normal puisqu'on parle quand même d'une connection à un utilisateur d'un autre lan 

[3-Way Handshake](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/3-Way%20Handshake.pcapng)
[packets ssh](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/sshtrafic.pcapng)
[FIN ssh](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/tcp%20fin%20ssh.pcapng)

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

🌞 **Dans le rendu, je veux**

```
[dreams@dns-server ~]$ sudo cat /etc/named.conf

options {
        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/va
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access

           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "tp4.b1" IN {
        type master;
        file "tp4.b1.db";
        allow-update { none; };
        allow-query { any; };
};

zone "1.4.10.in-addr.arpa" IN {
        type master;
        file "tp4.b1.rev";
        allow-update { none; };
        allow-query { any; };
};
```
```
[dreams@dns-server ~]$ sudo cat /var/named/tp4.b1.rev

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
        2019061800 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimum TTL
)

; Infos sur le server DNS lui même (NS =NameServer)
@ IN NS dns-server.tp4.b1.

; Reverse lookup for Name Server

201 IN PTR dns-server.tp4.b1.

11 IN PTR node1.tp4.b1
```
```
[dreams@dns-server ~]$ sudo cat /var/named/tp4.b1.db

$TTL 86400
@ IN SOA dns-server.tp4.b1 admin.tp4.b1 (
        2019061800 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimum TTL
)

; Infos sur le DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs

dns-server IN A 10.4.1.201

node1 IN A 10.4.1.11
```

```
sudo systemctl status named

 Active: active (running) since Tue 2022-10-25 15:54:40 CEST; 37min ago
```

```
ss -t -l -n

LISTEN               0                    10                                     127.0.0.1:53                                     0.0.0.0:*
```

🌞 **Ouvrez le bon port dans le firewall**

```
[dreams@dns-server ~]$ sudo firewall-cmd --add-port=53/tcp --permanent
[sudo] password for dreams:
success
[dreams@dns-server ~]$ sudo firewall-cmd --reload
success
```

## 3. Test

🌞 **Sur la machine `node1.tp4.b1`**

-add ```DNS=10.4.1.201``` in ```/etc/sysconfig/network```

```
[dreams@node1 ~]$ dig node1.tp4.b1

;; ANSWER SECTION:
node1.tp4.b1.           86400   IN      A       10.4.1.11

;; SERVER: 10.4.1.201#53(10.4.1.201)
```
```
[dreams@node1 ~]$ dig dns-server.tp4.b1

;; ANSWER SECTION:
dns-server.tp4.b1.      86400   IN      A       10.4.1.201

;; SERVER: 10.4.1.201#53(10.4.1.201)
```
```
[dreams@node1 ~]$ dig www.google.com

;; ANSWER SECTION:
www.google.com.         300     IN      A       142.250.179.100

;; SERVER: 10.4.1.201#53(10.4.1.201)
```
🌞 **Sur votre PC**

Windows :

```
PS C:\Users\quentin> nslookup
Serveur par dÚfaut :   dns.google
Address:  8.8.8.8

> set type=soa
> node1.tp4.b1 10.4.1.201
```

[DNS request](https://github.com/SmlDreams/tp-reseau/blob/main/tp4/pics/DNS%20request%20From%20windows.pcapng)