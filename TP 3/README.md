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
🌞Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping
