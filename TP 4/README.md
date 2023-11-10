# Réseau - TP 4 : DHCP

# I. DHCP Client
🌞 Déterminer (avec ````ipconfig /all```)

- l'adresse du serveur DHCP : ```Serveur DHCP . . . . . . . . . . . . . : 10.33.79.254```

- l'heure exacte à laquelle vous avez obtenu votre bail DHCP : ````Bail obtenu. . . . . . . . . . . . . . : vendredi 27 octobre 2023 13:24:43````

- l'heure exacte à laquelle il va expirer : ````Bail expirant. . . . . . . . . . . . . : samedi 28 octobre 2023 13:22:23````

🌞 Capturer un échange DHCP

> Il faut changer l'adresse IP dans le panneau de configuration avant de réattribuer automatiquement l'adresse IP.


🌞 Analyser la capture Wireshark

> [Capture de tp4_dhcp_client.pcapng](./tp4_dhcp_client.pcapng)



# II. Serveur DHCP
## 3. Setup topologie
🌞 Preuve de mise en place

> Pour trouver le nom du domaine, on doit utiliser la commade :

```
$ sudo nano /etc/resolv.conf
$ nameserver 8.8.8.8
```


- depuis dhcp.tp4.b1, envoyer un ping vers un nom de domaine public (pas une IP)
```
$ [sonita@dhcp ~]$ ping -c5 www.google.com
PING www.google.com (142.250.179.100) 56(84) bytes of data.
64 bytes from par21s20-in-f4.1e100.net (142.250.179.100): icmp_seq=1 ttl=117 time=12.4 ms
64 bytes from par21s20-in-f4.1e100.net (142.250.179.100): icmp_seq=2 ttl=117 time=10.9 ms
64 bytes from par21s20-in-f4.1e100.net (142.250.179.100): icmp_seq=3 ttl=117 time=10.6 ms
64 bytes from par21s20-in-f4.1e100.net (142.250.179.100): icmp_seq=4 ttl=117 time=11.0 ms
64 bytes from par21s20-in-f4.1e100.net (142.250.179.100): icmp_seq=5 ttl=117 time=10.8 ms

--- www.google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 10.573/11.156/12.427/0.654 ms
```


- depuis node2.tp4.b1, envoyer un ping vers un nom de domaine public (pas une IP)
```
$ [sonita@node2 ~]$ ping -c5 google.com
PING google.com (216.58.214.78) 56(84) bytes of data.
64 bytes from fra15s10-in-f78.1e100.net (216.58.214.78): icmp_seq=1 ttl=116 time=11.4 ms
64 bytes from fra15s10-in-f14.1e100.net (216.58.214.78): icmp_seq=2 ttl=116 time=11.2 ms
64 bytes from par10s39-in-f14.1e100.net (216.58.214.78): icmp_seq=3 ttl=116 time=14.1 ms
64 bytes from fra15s10-in-f78.1e100.net (216.58.214.78): icmp_seq=4 ttl=116 time=11.2 ms
64 bytes from 216.58.214.78: icmp_seq=5 ttl=116 time=13.7 ms

--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20116ms
rtt min/avg/max/mdev = 11.181/12.293/14.057/1.284 ms
```


- depuis ```node2.tp4.b1```, un traceroute vers une IP publique pour montrer que vos paquets à destination d'internet passent bien par le router.tp4.b1
```
[sonita@node2 ~]$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.4.1.254)  1.870 ms  1.914 ms  1.781 ms
 2  10.0.3.2 (10.0.3.2)  1.750 ms  1.412 ms  1.377 ms
 3  10.0.3.2 (10.0.3.2)  4.533 ms  4.453 ms  4.333 ms
```


## 4. Serveur DHCP

```
[sonita@dhcp ~]$ sudo dnf -y install dhcp-server
[sudo] password for sonita:
Rocky Linux 9 - BaseOS                                       5.0 kB/s | 4.1 kB     00:00
Rocky Linux 9 - BaseOS                                       2.5 MB/s | 1.9 MB     00:00
Rocky Linux 9 - AppStream                                     14 kB/s | 4.5 kB     00:00
Rocky Linux 9 - AppStream                                    9.7 MB/s | 7.1 MB     00:00
Rocky Linux 9 - Extras                                        10 kB/s | 2.9 kB     00:00
Rocky Linux 9 - Extras                                        33 kB/s |  11 kB     00:00
Dependencies resolved.
=============================================================================================
 Package               Architecture     Version                       Repository        Size
=============================================================================================
Installing:
 dhcp-server           x86_64           12:4.4.2-18.b1.el9            baseos           1.2 M

Transaction Summary
=============================================================================================
Install  1 Package

Total download size: 1.2 M
Installed size: 3.9 M
Downloading Packages:
dhcp-server-4.4.2-18.b1.el9.x86_64.rpm                       4.8 MB/s | 1.2 MB     00:00
---------------------------------------------------------------------------------------------
Total                                                        2.5 MB/s | 1.2 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                     1/1
  Running scriptlet: dhcp-server-12:4.4.2-18.b1.el9.x86_64                               1/1
  Installing       : dhcp-server-12:4.4.2-18.b1.el9.x86_64                               1/1
  Running scriptlet: dhcp-server-12:4.4.2-18.b1.el9.x86_64                               1/1
  Verifying        : dhcp-server-12:4.4.2-18.b1.el9.x86_64                               1/1

Installed:
  dhcp-server-12:4.4.2-18.b1.el9.x86_64

Complete!
```


> Pour avoir les autorisations de modifier le fichier, il faut entrer cette commande : ```sudo chmod 777 dhcpd.conf```

```
# nom du domaine specifique
option domain-name "srv.world";

# server DNS hostname ou adresse IP specifique
option domain-name-servers dlp.srv.world;

# durée du bail par défaut
default-lease-time 600;

# durée du bail maximum
max-lease-time 7200;

# ce serveur DHCP permet de valider
authoritative;

# Adresse network et masque sous reseau
subnet 10.4.1.253 netmask 255.255.255.0 {
        range dynamic-bootp 10.4.1.137 10.4.1.237;
        option routers 10.4.1.254;
}       
```
