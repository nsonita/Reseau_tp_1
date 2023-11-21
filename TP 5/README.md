# Réseau - TP 5 : TCP, UDP et services réseau

# I. First steps

🌞 Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP

🌞 Demandez l'avis à votre OS

- Lien 1 : [Youtube : Gaspard de la nuit, M. 55: I. Ondine](./tp5_service_1.pcapng) c'est un UDP.

- Lien 2 : [Discord (avec un appel)](./tp5_service_2.pcapng) c'est un UDP.

- Lien 3 : [Steam (téléchargement)](./tp5_service_3.pcapng) c'est un TCP.

- Lien 4 : [Instagram (messages)](./tp5_service_4.pcapng) c'est un TCP.

- Lien 5 : [Opéra Bordeaux : Bertrand Chamayou](./tp5_service_5.pcapng) c'est un TCP.


# II. Setup Virtuel
## 1. SSH

🌞 Demandez aux OS
> Le SSH utilise uniquement du TCP.
```
PS C:\Users\Sonita Nou> netstat

Connexions actives
  Proto  Adresse locale         Adresse distante       État
  TCP    10.5.1.1:65226         10.5.1.254:ssh         ESTABLISHED
```

```
[sonita@node1 ~]$ ss
Netid State Recv-Q Send-Q                Local Address:Port    Peer Address:Port   
tcp   ESTAB 0      0                         10.5.1.11:ssh         10.5.1.1:56638 
```

🌞 Examinez le trafic dans Wireshark
[Capture du trafic dans Wireshark](./tp5_3_way.pcapng)



## 2. Routage
🌞 Prouvez que

- node1.tp5.b1 a un accès internet
```
[sonita@node1 network-scripts]$ ping -c5 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=14.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=12.3 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=115 time=16.6 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=115 time=14.3 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=115 time=15.0 ms
--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4009ms
rtt min/avg/max/mdev = 12.305/14.540/16.619/1.383 ms
```

- node1.tp5.b1 peut résoudre des noms de domaine publics (comme www.ynov.com)
```
[sonita@router ~]$ dig www.ynov.com
;; ANSWER SECTION:
www.ynov.com.           300     IN      A       104.26.11.233
www.ynov.com.           300     IN      A       104.26.10.233
www.ynov.com.           300     IN      A       172.67.74.226
```


## 3. Serveur Web
🌞 Installez le paquet nginx

🌞 Créer le site web

🌞 Donner les bonnes permissions


🌞 Créer un fichier de configuration NGINX pour notre site web
```
[sonita@web conf.d]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2023-11-21 10:29:54 CET; 10s ago
    Process: 1807 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1808 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1809 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1810 (nginx)
      Tasks: 2 (limit: 4674)
     Memory: 2.0M
        CPU: 20ms
     CGroup: /system.slice/nginx.service
             ├─1810 "nginx: master process /usr/sbin/nginx"
             └─1811 "nginx: worker process"

Nov 21 10:29:54 web.tp5.b1 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 21 10:29:54 web.tp5.b1 nginx[1808]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 21 10:29:54 web.tp5.b1 nginx[1808]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Nov 21 10:29:54 web.tp5.b1 systemd[1]: Started The nginx HTTP and reverse proxy server.
```

🌞 Ouvrir le port firewall
```
[sonita@web conf.d]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[sonita@web conf.d]$ sudo firewall-cmd --reload
success
```

🌞 Visitez le serveur web !
```
[sonita@node1 ~]$ curl 10.5.1.12
<h1>MEOW</h1>
```

🌞 Visualiser le port en écoute
```
[sonita@web ~]$ ss
Netid    State    Recv-Q    Send-Q                      Local Address:Port          Peer Address:Port     Process
```

🌞 Analyse trafic
[Capture d'analyse web](./tp5_3_way.pcapng)