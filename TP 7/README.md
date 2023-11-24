# TP7 : Do u secure

# II. SSH
## 1. Fingerprint
### B. Manips

🌞 Effectuez une connexion SSH en vérifiant le fingerprint
```
PS C:\Users\Sonita Nou> ssh sonita@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ECDSA key fingerprint is SHA256:zHq8Q0uJZ8y/zU8vGJme5mYG7raQ4j73mC1kkKGmXE4.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.7.1.11' (ECDSA) to the list of known hosts.
```

```
[sonita@john ~]$ sudo ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
256 SHA256:cafbLpSksNa81mgbdf9egE/jp1AC5rnttm+bzRdRn8o /etc/ssh/ssh_host_ed25519_key.pub (ED25519)
```

## 2. Conf serveur SSH
🌞 Consulter l'état actuel
```
[sonita@router ssh]$ ss -altpn
State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       Process
LISTEN        0             128                        0.0.0.0:22                      0.0.0.0:*
```

🌞 Modifier la configuration du serveur SSH
```
[sonita@router ssh]$ sudo cat sshd_config
Port 22000
ListenAddress 0.0.0.0
ListenAddress 10.7.1.254
```

🌞 Prouvez que le changement a pris effet
```
State        Recv-Q        Send-Q               Local Address:Port                Peer Address:Port       Process
LISTEN       0             128                     10.7.1.254:22000                    0.0.0.0:*
```

🌞 N'oubliez pas d'ouvrir ce nouveau port dans le firewall
```
[sonita@router ssh]$ sudo firewall-cmd --add-port=22000/tcp --permanent
success
[sonita@router ssh]$ sudo firewall-cmd --reload
success
```

🌞 Effectuer une connexion SSH sur le nouveau port
```
PS C:\Users\Sonita Nou> ssh sonita@10.7.1.254 -p 22000
```

## 3. Connexion par clé
### B. Manips
🌞 Générer une paire de clés
```
Sonita Nou@DESKTOP-EKV04VJ MINGW64 ~
$ ssh-copy-id sonita@10.7.1.11
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/Sonita Nou/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
sonita@10.7.1.11's password:
Permission denied, please try again.
sonita@10.7.1.11's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'sonita@10.7.1.11'"
and check to make sure that only the key(s) you wanted were added.
```

🌞 Connectez-vous en SSH à la machine
> On n'a pas eu besoin d'entrer le mot de passe pour rentrer chez John.
```
PS C:\Users\Sonita Nou> ssh sonita@10.7.1.11
Last failed login: Fri Nov 24 15:27:09 CET 2023 from 10.7.1.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Fri Nov 24 15:26:42 2023 from 10.7.1.1
[sonita@john ~]$

```

🌞 Supprimer les clés sur la machine router.tp7.b1
> Pour supprimer tous les fichiers qui commencent par [ssh_host_] :
```
[sonita@router ssh]$ sudo rm ssh_host_*
```

🌞 Regénérez les clés sur la machine router.tp7.b1
```
[sonita@router ssh]$ sudo ssh-keygen -A
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
```

🌞 Tentez une nouvelle connexion au serveur
> Le message d'erreur indique qu'il y a un changement d'identification, qu'il estpossibleque les clefs ont changé, qu'il y a peut être une attaque, que l'empreinte a été perdue et qu'il faut contacter l'administrateur. On ne peut pas se reconnecter pour le moment.

> Pour pouvoir se reconnecter à son SSH, il faut modifier le fichier ```known_hosts```



# III. Web sécurisé

🌞 Montrer sur quel port est disponible le serveur web
```
[sonita@web conf.d]$ ss -ltpun
Netid      State       Recv-Q      Send-Q            Local Address:Port             Peer Address:Port      Process
tcp        LISTEN      0           511                     0.0.0.0:80                    0.0.0.0:*
tcp        LISTEN      0           511                        [::]:80                       [::]:*
```

## 1. Setup HTTPS
🌞 Générer une clé et un certificat sur web.tp7.b1

🌞 Modification de la conf de NGINX

🌞 Conf firewall

🌞 Redémarrez NGINX

🌞 Prouvez que NGINX écoute sur le port 443/tcp

```
[sonita@web /]$ ss -ltpn
State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       Process
LISTEN        0             511                      10.7.1.12:443                     0.0.0.0:*
```

🌞 Visitez le site web en https
```
[sonita@john ~]$ curl -k https://10.7.1.12
<h1>MEOW<h1>
```