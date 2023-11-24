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
subnet 10.4.1.0 netmask 255.255.255.0 {
        range dynamic-bootp 10.4.1.137 10.4.1.237;
        option routers 10.4.1.254;
}       
```

> Pour afficher l'état du serveur :
```
[sonita@dhcp dhcp]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-10 14:33:53 CET; 3min 36s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1580 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11115)
     Memory: 4.6M
        CPU: 12ms
     CGroup: /system.slice/dhcpd.service
             └─1580 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --n>

Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Config file: /etc/dhcp/dhcpd.conf
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Database file: /var/lib/dhcpd/dhcpd.leases
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: PID file: /var/run/dhcpd.pid
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Source compiled to use binary-leases
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Wrote 0 leases to leases file.
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Listening on LPF/enp0s3/08:00:27:b1:90:55/10.4.1.0/>
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Sending on   LPF/enp0s3/08:00:27:b1:90:55/10.4.1.0/>
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Sending on   Socket/fallback/fallback-net
Nov 10 14:33:53 dhcp.tp4.b1 dhcpd[1580]: Server starting service.
Nov 10 14:33:53 dhcp.tp4.b1 systemd[1]: Started DHCPv4 Server Daemon.
```

## 5. Client DHCP

🌞 Test !
```
[sonita@localhost network-scripts]$ cat ifcfg-enp0s3
DEVICE=enp0s3

BOOTPROTO=dhcp
ONBOOT=yes
```

🌞 Prouvez que
```
[sonita@node1 ~]$ nmcli con show "System enp0s3" | grep -i dhcp4
DHCP4.OPTION[1]:                        dhcp_client_identifier = 01:08:00:27:82:79:8c
DHCP4.OPTION[2]:                        dhcp_lease_time = 600
DHCP4.OPTION[3]:                        dhcp_server_identifier = 10.4.1.253
DHCP4.OPTION[4]:                        expiry = 1699626422
DHCP4.OPTION[5]:                        ip_address = 10.4.1.137
```

> ping ```router.tp4.b1``` :
```
[sonita@node1 ~]$ ping -c 3 10.4.1.254
PING 10.4.1.254 (10.4.1.254) 56(84) bytes of data.
64 bytes from 10.4.1.254: icmp_seq=1 ttl=64 time=0.392 ms
64 bytes from 10.4.1.254: icmp_seq=2 ttl=64 time=0.544 ms
64 bytes from 10.4.1.254: icmp_seq=3 ttl=64 time=0.478 ms

--- 10.4.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2083ms
rtt min/avg/max/mdev = 0.392/0.471/0.544/0.062 ms
```


> ping ```node2.tp4.b1```
```
[sonita@node1 ~]$ ping -c 3 10.4.1.12
PING 10.4.1.12 (10.4.1.12) 56(84) bytes of data.
64 bytes from 10.4.1.12: icmp_seq=1 ttl=64 time=0.419 ms
64 bytes from 10.4.1.12: icmp_seq=2 ttl=64 time=0.461 ms
64 bytes from 10.4.1.12: icmp_seq=3 ttl=64 time=0.480 ms

--- 10.4.1.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2054ms
rtt min/avg/max/mdev = 0.419/0.453/0.480/0.025 ms
```


🌞 Bail DHCP serveur
```
[sonita@dhcp dhcp]$ sudo journalctl -xe -u dhcpd -f
Nov 10 14:44:15 dhcp.tp4.b1 dhcpd[1580]: DHCPDISCOVER from 08:00:27:82:79:8c via enp0s3
Nov 10 14:44:16 dhcp.tp4.b1 dhcpd[1580]: DHCPOFFER on 10.4.1.137 to 08:00:27:82:79:8c via enp0s3
Nov 10 14:44:16 dhcp.tp4.b1 dhcpd[1580]: DHCPREQUEST for 10.4.1.137 (10.4.1.253) from 08:00:27:82:79:8c via enp0s3
Nov 10 14:44:16 dhcp.tp4.b1 dhcpd[1580]: DHCPACK on 10.4.1.137 to 08:00:27:82:79:8c via enp0s3
Nov 10 14:46:42 dhcp.tp4.b1 dhcpd[1580]: reuse_lease: lease age 146 (secs) under 25% threshold, reply with unaltered, existing lease for 10.4.1.137
Nov 10 14:46:42 dhcp.tp4.b1 dhcpd[1580]: DHCPDISCOVER from 08:00:27:82:79:8c via enp0s3
Nov 10 14:46:42 dhcp.tp4.b1 dhcpd[1580]: DHCPOFFER on 10.4.1.137 to 08:00:27:82:79:8c via enp0s3
Nov 10 14:46:42 dhcp.tp4.b1 dhcpd[1580]: reuse_lease: lease age 146 (secs) under 25% threshold, reply with unaltered, existing lease for 10.4.1.137
Nov 10 14:46:42 dhcp.tp4.b1 dhcpd[1580]: DHCPREQUEST for 10.4.1.137 (10.4.1.253) from 08:00:27:82:79:8c via enp0s3
```



## 6. Options DHCP

🌞 Nouvelle conf !
```
[sonita@dhcp dhcp]$ cat dhcpd.conf
option domain-name-servers 8.8.8.8;

default-lease-time 21600;

max-lease-time 43200;

authoritative;

subnet 10.4.1.0 netmask 255.255.255.0 {
        range dynamic-bootp 10.4.1.137 10.4.1.237;
        option routers 10.4.1.254;

}
```

🌞 Test !
> La commande pour demander une nouvelle adresse IP est : ```dhclient```
> Enregistrement de l'adresse d'un serveur DNS :
```
[sonita@node1 ~]$ nmcli con show "System enp0s3" | grep -i dhcp4
DHCP4.OPTION[1]:                        dhcp_client_identifier = 01:08:00:27:82:79:8c
DHCP4.OPTION[2]:                        dhcp_lease_time = 21346
DHCP4.OPTION[3]:                        dhcp_server_identifier = 10.4.1.253
DHCP4.OPTION[4]:                        domain_name_servers = 8.8.8.8
DHCP4.OPTION[5]:                        expiry = 1699649527
DHCP4.OPTION[6]:                        ip_address = 10.4.1.137
DHCP4.OPTION[24]:                       routers = 10.4.1.254
```

> Accès Internet :
```
[sonita@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=17.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=17.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=23.2 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 17.551/19.547/23.170/2.565 ms
```

🌞 Capture Wireshark
(Machines perdues depuis la réinitialisation de mon OS ☹️)