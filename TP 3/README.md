# Réseau - TP 3

# I. ARP

## 1.Echange ARP

🌞Générer des requêtes ARP
> Pour commencer, on doit définir deux adresses IP différentes dans deux VM différentes.
```
$ cd /etc/sysconfig/network-scripts
$ sudo nano ifcfg-enp8s3
_____________________
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.11 (et 10.3.1.12 pour Marcel)
NETMASK=255.255.255.0
______________________
$ sudo systemctl restart NetworkManager
$ nmcli con show
$ sudo nmcli con up "System enp0s3"
```

> A partir de ça, on pourra ping l'autre machine, donc si John veut ping Marcel, il doit écrire ça :
```
$ ping 10.3.1.12
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.597 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.529 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.403 ms
```

> Ensuite, pour connaître l'adressse physique du PC de Marcel, il faut faire cette commande :

```
$ ip neigh show
10.3.1.12 dev enp0s3 08:00:27:43:54:a7 STALE
```

> Et du côté de John :
```
$ ip neigh show
10.3.1.11 dev enp0s3 08:00:27:47:0b:a8 STALE
```

## 2. Analyse de trames

🌞Analyse de trames
> Pour vider la table ARP : ```$ sudo ip neigh flush all```

> Pour faire la capture de **tp3_arp3.pcap** on doit faire cette commande : ```scp sonita@10.3.1.11:/home/sonita/tp3_arp.pcapng```

> [Capture de tp3_arp3.pcap](./tp3_arp3.pcap])



# II. Routage
## 1. Mise en place du routage

➜ Activer le routage sur le noeud ```router```
```
$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s3 enp0s8

$ sudo firewall-cmd --add-masquerade --zone=public
success

$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```


🌞Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping

> Depuis John :
```
$ sudo ip route add 10.3.2.12 via 10.3.1.254 dev enp0s3
```

> Depuis Marcel :
```
$ sudo ip route add 10.3.1.11 via 10.3.2.254 dev enp0s3
```

> Voici le ping de John par exemple :
```
$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.72 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.888 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.07 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.33 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=0.666 ms
64 bytes from 10.3.2.12: icmp_seq=6 ttl=63 time=1.70 ms
64 bytes from 10.3.2.12: icmp_seq=7 ttl=63 time=1.09 ms
64 bytes from 10.3.2.12: icmp_seq=8 ttl=63 time=1.13 ms
--- 10.3.2.12 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 7030ms
rtt min/avg/max/mdev = 0.666/1.197/1.716/0.343 ms
```

## 2. Analyse de trames

🌞Analyse des échanges ARP
> Pour vider la table ARP :
```
$ sudo ip neigh flush all
[sudo] password for sonita:
[sonita@john network-scripts]$ ip neigh show
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:15 REACHABLE
```

> ```tcpdump``` pour John :
```
$ sudo tcpdump -i enp0s3 -c 10 -w tp3_2routage_john.pcap not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

> Ping depuis John :
```
$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.16 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.07 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.903 ms
--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.903/1.045/1.163/0.107 ms
```

> Les trois tables ARP :

```
[sonita@john network-scripts]$ ip neigh show
10.3.1.254 dev enp0s3 lladdr 08:00:27:62:3c:81 STALE
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:15 DELAY
```
```
[sonita@marcel network-scripts]$ ip neigh show
10.3.2.1 dev enp0s3 lladdr 0a:00:27:00:00:05 REACHABLE
10.3.2.254 dev enp0s3 lladdr 08:00:27:47:3a:38 STALE
```
```
[sonita@router network-scripts]$ ip neigh show
10.3.2.11 dev enp0s3 lladdr 08:00:27:47:0b:a8 STALE
10.3.2.12 dev enp0s3 lladdr 08:00:27:43:54:a7 STALE
```
