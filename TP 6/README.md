# TP6 : Un LAN maîtrisé meo ?

# II. Serveur DNS
## 2. Setup

🌞 Dans le rendu, je veux

- un cat des 3 fichiers de conf (fichier principal + deux fichiers de zone)
> Fichier named.conf :
```
[sonita@dns ~]$ sudo cat /etc/named.conf
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "tp6.b1" IN {
        type master;
        file "tp6.b1.db";
        allow-update { none; };
        allow-query {any; };
};
zone "1.4.10.in-addr.arpa" IN {
        type master;
        file "tp6.b1.rev";
        allow-update { none; };
        allow-query { any; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

> fichier tp6.b1.db :
```
[sonita@dns named]$ sudo cat tp6.b1.db
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
        2019061800 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimum TTL
)

@ IN NS dns.tp6.b1.

dns     IN A 10.6.1.101
john    IN A 10.6.1.11
```

> fichier tp6.b1.rev :
```
[sonita@dns named]$ sudo cat tp6.b1.rev
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
        2019061800 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimum TTL
)

@ IN NS dns.tp6.b1.


101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```


- un systemctl status named qui prouve que le service tourne bien
```
[sonita@dns ~]$ sudo systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Thu 2023-11-23 14:44:37 CET; 5min ago
   Main PID: 1532 (named)
      Tasks: 5 (limit: 4674)
     Memory: 19.6M
        CPU: 37ms
     CGroup: /system.slice/named.service
             └─1532 /usr/sbin/named -u named -c /etc/named.conf

Nov 23 14:44:37 dns.tp6.b1 named[1532]: network unreachable resolving './DNSKEY/IN': 199.9.14.201#53
Nov 23 14:44:37 dns.tp6.b1 named[1532]: zone localhost.localdomain/IN: loaded serial 0
Nov 23 14:44:37 dns.tp6.b1 named[1532]: network unreachable resolving './NS/IN': 199.9.14.201#53
Nov 23 14:44:37 dns.tp6.b1 named[1532]: network unreachable resolving './DNSKEY/IN': 199.7.83.42#53
Nov 23 14:44:37 dns.tp6.b1 named[1532]: network unreachable resolving './NS/IN': 199.7.83.42#53
Nov 23 14:44:37 dns.tp6.b1 named[1532]: all zones loaded
Nov 23 14:44:37 dns.tp6.b1 systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 23 14:44:37 dns.tp6.b1 named[1532]: running
Nov 23 14:44:37 dns.tp6.b1 named[1532]: resolver priming query complete
```


- une commande ss qui prouve que le service écoute bien sur un port
```
[sonita@dns ~]$ sudo ss -lntpu
Netid   State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port   Process
tcp     LISTEN   0        10            10.6.1.101:53             0.0.0.0:*       users:(("named",pid=1683,fd=21))
```


🌞 Ouvrez le bon port dans le firewall
```
[sonita@dns ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 53/tcp 53/udp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```


## 3. Test
🌞 Sur la machine john.tp6.b1

- configurez la machine pour qu'elle utilise votre serveur DNS quand elle a besoin de résoudre des noms

- voir mémo pour ça aussi
    - john ne doit connaître qu'un seul serveur DNS : le vôtre
    - une seule IP renseignée dans le fichier ```resolv.conf```
```
[sonita@john etc]$ sudo cat resolv.conf
nameserver 10.6.1.101
```

- assurez-vous que vous pouvez :
    - résoudre des noms comme john.tp6.b1 et dns.tp6.b1
    - mais aussi des noms comme www.ynov.com

```
[sonita@john ~]$ dig john.tp6.b1

; <<>> DiG 9.16.23-RH <<>> john.tp6.b1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51964
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 169eaa70c8a9be5e01000000655f6ce34221029f43824672 (good)
;; QUESTION SECTION:
;john.tp6.b1.                   IN      A

;; ANSWER SECTION:
john.tp6.b1.            86400   IN      A       10.6.1.11
```

```
[sonita@john ~]$ dig dns.tp6.b1

; <<>> DiG 9.16.23-RH <<>> dns.tp6.b1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1004
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 663c433c72a2078101000000655f6cfd8e01d77f96d7013e (good)
;; QUESTION SECTION:
;dns.tp6.b1.                    IN      A

;; ANSWER SECTION:
dns.tp6.b1.             86400   IN      A       10.6.1.101
```

```
[sonita@john ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21066
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 39f564a098eb63e601000000655f75925777530236ca68f1 (good)
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               284     IN      A       172.67.74.226
ynov.com.               284     IN      A       104.26.11.233
ynov.com.               284     IN      A       104.26.10.233
```


🌞 Sur votre PC
```
PS C:\Windows\system32> Set-DNSClientServerAddress "Wi-Fi" -ServerAddresses ("10.6.1.101")
PS C:\Windows\system32> nslookup
Serveur par dÚfaut :   UnKnown
Address:  10.6.1.101

> john.tp6.b1
Serveur :   UnKnown
Address:  10.6.1.101

Nom :    john.tp6.b1
Address:  10.6.1.11
```

> 🦈[Capture tp6_dns.pcapng](./tp6_dns.pcapng)


# III. Serveur DHCP
