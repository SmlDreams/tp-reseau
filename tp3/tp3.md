
## I. ARP

### 1. Echange ARP

🌞**Générer des requêtes ARP**

```
[dreams@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.411 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.231 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.261 ms
64 bytes from 10.3.1.12: icmp_seq=4 ttl=64 time=0.197 ms
^C
--- 10.3.1.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3090ms
rtt min/avg/max/mdev = 0.197/0.275/0.411/0.081 ms
```

machine who request : (PC1)

```
[dreams@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3c REACHABLE
10.3.1.12 dev enp0s8 lladdr 08:00:27:ee:13:7d REACHABLE
```

IPMAC PC1 : 08:00:27:08:f4:fe

machine who reply : (PC1)

```
[dreams@localhost ~]$ ip neigh show
10.3.1.11 dev enp0s8 lladdr 08:00:27:08:f4:fe REACHABLE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3c REACHABLE
```

IPMAC PC2 : 0a:00:27:00:00:3c

```
[dreams@localhost ~]$ ip a

2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:08:f4:fe brd ff:ff:ff:ff:ff:ff
```

### 2. Analyse de trames

🌞**Analyse de trames**

[ARP request reply](https://github.com/SmlDreams/tp-reseau/blob/main/tp3/pics/ARP_request_reply.pcap)

> **Si vous ne savez pas comment récupérer votre fichier `.pcapng`** sur votre hôte afin de l'ouvrir dans Wireshark, et me le livrer en rendu, demandez-moi.

## II. Routage

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

```
firewall-cmd --add-masquerade --zone=public --permanent
```

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

```
ip route add 10.3.1.0 via 10.3.1.254
```
```
ip route add 10.3.2.0 via 10.3.2.254
```
```
[dreams@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.21 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.738 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.657 ms
^C
--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2014ms
rtt min/avg/max/mdev = 0.657/0.866/1.205/0.241 ms
```

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

```
ip neigh flush all
```
```
[dreams@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.21 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.738 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.657 ms
^C
--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2014ms
rtt min/avg/max/mdev = 0.657/0.866/1.205/0.241 ms
```
PC1 :
```
[dreams@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3c REACHABLE
10.3.1.254 dev enp0s8 lladdr 08:00:27:dc:a1:15 STALE
```
PC2 :
```
[dreams@localhost ~]$ ip neigh show
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:41 DELAY
10.3.2.254 dev enp0s8 lladdr 08:00:27:b2:cd:14 STALE
```
ROUTEUR :
```
[dreams@localhost ~]$ ip neigh show
10.3.1.11 dev enp0s8 lladdr 08:00:27:08:f4:fe STALE
10.3.2.12 dev enp0s9 lladdr 08:00:27:ee:13:7d STALE
10.3.2.1 dev enp0s9 lladdr 0a:00:27:00:00:41 REACHABLE
```

PC1 ask who has MAC of 10.3.1.254, ROUTEUR reply 
then
ROUTEUR request if MAC of PC2 is always here , PC2 reply


- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange

| ordre | type trame  | IP source | MAC source        | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------|----------------|----------------------------|
| 1     | Requête ARP | 10.3.1.254| 08:00:27:dc:a1:15 |10.3.1.11       | 08:00:27:08:f4:fe          |
| 2     | Réponse ARP | 10.3.1.11 | 08:00:27:08:f4:fe |10.3.1.254      | 08:00:27:dc:a1:15          | 
| 3     | Ping        | 10.3.1.254| 08:00:27:dc:a1:15 | 10.3.1.11      | 08:00:27:08:f4:fe          |
| 4     | Pong        | 10.3.1.11 | 08:00:27:08:f4:fe | 10.3.1.254     | 08:00:27:dc:a1:15          |

🦈 **Capture réseau `tp2_routage_marcel.pcapng`**

[arp+ping john à marcel](https://github.com/SmlDreams/tp-reseau/blob/main/tp3/pics/arp%2Bping.pcap)

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

```
nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
```
```
[root@localhost ~]# ping 1.1.1.1

PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=24.3 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=23.4 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=53 time=24.5 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=53 time=24.3 ms
64 bytes from 1.1.1.1: icmp_seq=5 ttl=53 time=24.1 ms
64 bytes from 1.1.1.1: icmp_seq=6 ttl=53 time=21.2 ms
64 bytes from 1.1.1.1: icmp_seq=7 ttl=53 time=22.3 ms
64 bytes from 1.1.1.1: icmp_seq=8 ttl=53 time=21.6 ms
64 bytes from 1.1.1.1: icmp_seq=9 ttl=53 time=20.9 ms
^C
--- 1.1.1.1 ping statistics ---
9 packets transmitted, 9 received, 0% packet loss, time 8014ms
rtt min/avg/max/mdev = 20.866/22.965/24.517/1.383 ms
```

```
[dreams@localhost network-scripts]$ dig google.com

; <<>> DiG 9.16.23-RH <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48953
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       216.58.209.238

;; Query time: 32 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Tue Oct 11 17:45:53 CEST 2022
;; MSG SIZE  rcvd: 55
```

```
[dreams@localhost network-scripts]$ ping 1.1.1.1

PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=24.6 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=23.2 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 23.180/23.871/24.562/0.691 ms
```
🌞**Analyse de trames**

```
[root@localhost ~]# ping 8.8.8.8

PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=22.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=21.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=21.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=21.3 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=112 time=21.7 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=112 time=24.5 ms
64 bytes from 8.8.8.8: icmp_seq=7 ttl=112 time=23.6 ms
64 bytes from 8.8.8.8: icmp_seq=8 ttl=112 time=25.4 ms

--- 8.8.8.8 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 7012ms
rtt min/avg/max/mdev = 21.297/22.791/25.445/1.464 ms
```

- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source      | MAC source       |IP destination| MAC destination |
|-------|------------|----------------|------------------|--------------|-----------------|
| 1     | ping       | 10.3.1.11      | 08:00:27:08:f4:fe| 8.8.8.8      |08:00:27:dc:a1:15|
| 2     | pong       | 8.8.8.8        | 08:00:27:dc:a1:15| 10.3.1.11    |08:00:27:08:f4:fe|

🦈 **Capture réseau `tp2_routage_internet.pcapng`**

[ping to internet](https://github.com/SmlDreams/tp-reseau/blob/main/tp3/pics/ping_internet.pcap)

## III. DHCP


### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP**

```
dnf install dhcp-server
```
```
sudo nano /etc/dhcp/dhcpd.conf
```
change file into :

```
default-lease-time 900;
max-lease-time 10800;

authoritative;

subnet 192.168.20.0 netmask 255.255.255.0 {
  range 192.168.20.10 192.168.20.200;
  option routers 192.168.20.1;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 192.168.20.1;

}
```

reboot bob to assure dhcp server is ready 
🌞**Améliorer la configuration du DHCP**

```
nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
```

change file into :

```
NAME=enp0s8
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes

GATEWAY=10.3.1.254
DNS=8.8.8.8
```
```
dhclient -r -v
```
```
dhclient -v
```
```
[dreams@localhost ~]$ ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:0c:f3:a7 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 508sec preferred_lft 508sec
    inet 10.3.1.3/24 brd 10.3.1.255 scope global secondary dynamic enp0s8
       valid_lft 731sec preferred_lft 731sec
    inet6 fe80::a00:27ff:fe0c:f3a7/64 scope link
       valid_lft forever preferred_lft forever
```
```
[dreams@localhost ~]$ ping 8.8.8.8

PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=22.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=29.7 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 22.136/25.931/29.727/3.795 ms
```
```
[dreams@localhost ~]$ ip r s

default via 10.3.1.254 dev enp0s8
default via 10.3.1.254 dev enp0s8 proto dhcp src 10.3.1.2 metric 100
10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.2 metric 100
```
```
[dreams@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.787 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.819 ms
^C
--- 10.3.2.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1045ms
rtt min/avg/max/mdev = 0.787/0.803/0.819/0.016 ms
```
```
[dreams@localhost ~]$ dig google.com

google.com.             287     IN      A       216.58.209.238
```
```
[dreams@localhost ~]$ ping google.com

PING google.com (216.58.209.238) 56(84) bytes of data.
64 bytes from par10s29-in-f238.1e100.net (216.58.209.238): icmp_seq=1 ttl=247 time=24.4 ms
64 bytes from par10s29-in-f238.1e100.net (216.58.209.238): icmp_seq=2 ttl=247 time=23.2 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 23.214/23.816/24.418/0.602 ms
```

### 2. Analyse de trames

🌞**Analyse de trames**

[DORA](https://github.com/SmlDreams/tp-reseau/blob/main/tp3/pics/requestDHCP.pcapng)
