# <span style="color:red">Réseau - TP 1

>> Pour un PC Windows, on utilise la commande **ipconfig /all** :
``````
ipconfig /all
``````

## <span style="color:orangered">I. Exploration locale en solo
### <span style="color:springgreen">1. Affichage d'informations sur la pile TCP/IP locale

🌞 Affichez les infos des cartes réseau de votre PC
> Pour l'interface wifi :
```
$ ipconfig
Carte réseau sans fil Wi-Fi :
   Adresse physique . . . . . . . . . . . : C8-B2-9B-61-44-0F
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.48.146
```
> Pour l'interface Ethernet, il n'y a pas d'adresse IP
``````
$ ipconfig /all
Carte Ethernet Ethernet :
   Adresse physique . . . . . . . . . . . : 2C-F0-5D-66-BE-F2
``````
🌞 Affichez votre gateway
``````
$ ipconfig /all
Carte réseau sans fil Wi-Fi :
   Passerelle par défaut. . . . . . . . . : 10.33.51.254
``````

🌞 Déterminer la MAC de la passerelle
``````
$ arp -a
Interface : 10.33.48.146 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.33.51.254          7c-5a-1c-cb-fd-a4     dynamique
  ``````

🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)
> En faisant [Panneau de configuration/Réseau et Internet/Centre Réseau et partage/Etat de Wi-Fi], on retrouve ces informations :
```
Adresse physique: C8-B2-9B-61-44-0F
Adresse IPv4: 10.33.48.146
Passerelle par défaut IPv4: 10.33.51.254
```

### <span style="color:springgreen">2. Modifications des informations

A. Modification d'adresse IP (part 1)

🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :
> On refait le même schéma que pour afficher les informations de la cart IP mais on change les propriétés.

🌞 Il est possible que vous perdiez l'accès internet.
> Il est possible de perdre l'accès internet car si on a exactement la même adresse qu'un autre utilisateur, il ne peut pas envoyer sur les deux mêmes utilisateurs.


## <span style="color:orangered">II. Exploration locale en duo

### <span style="color:springgreen">3. Modification d'adresse IP
🌞 Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau
> En faisant **[Panneau de configuration/Réseau et Internet/Connexion réseau/Ethernet]** on modifie l'adresse IP dans les **[Propriétés/Protocole Internet version 4 (TCP/IPv4)]** avec l'adresse IPv4 : 10.10.10.24 et le masque de sous-réseau IPv4 : 255.255.255.0

🌞 Vérifier à l'aide d'une commande que votre IP a bien été changée
 En retournant dans le PowerShell :
 ```
$ ipconfig /all
Carte Ethernet Ethernet :
   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.24(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
```

🌞 Vérifier que les deux machines se joignent
```
$ ping 10.10.10.20
Envoi d’une requête 'Ping'  10.10.10.24 avec 32 octets de données :
Réponse de 10.10.10.20 : octets=32 temps<1ms TTL=128
Réponse de 10.10.10.20 : octets=32 temps<1ms TTL=128
Réponse de 10.10.10.20 : octets=32 temps<1ms TTL=128
Réponse de 10.10.10.20 : octets=32 temps<1ms TTL=128
Statistiques Ping pour 10.10.10.24:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```

🌞 Déterminer l'adresse MAC de votre correspondant
```
$ arp -a
Interface : 10.10.10.24 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.10.10.20           84-69-93-59-20-70     dynamique
```

### <span style="color:springgreen">4. Petit chat privé
🌞 sur le PC serveur
```
$ nc.exe -l -p 8833
```

🌞 sur le PC client
```
$ nc.exe 10.10.10.24 8833
[message]
```

🌞 Visualiser la connexion en cours
```
$ netstat -a -n -b | Select-String 8833
TCP    10.10.10.20:8833       10.10.10.24:54261      ESTABLISHED
```

🌞 Pour aller un peu plus loin
```
$ nc.exe -l -p 8833 -s 10.10.10.20
[message]
```


## <span style="color:orangered">III. Manipulations d'autres outils/protocoles côté client

### <span style="color:springgreen">1.DHCP
🌞Exploration du DHCP, depuis votre PC
```
$ ipconfig /all
adresse ip du serveur DHCP du réseau wifi ynov :
10.33.51.254
date d'expiration de votre bail DHCP :
mardi 17 octobre 2023 13:27:28
```

🌞** Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**
```
$ ipconfig /all
adresse IP du serveur DNS :
10.33.10.2
```

🌞 Utiliser, en ligne de commande l'outil nslookup (Windows, MacOS) ou dig (GNU/Linux, MacOS) pour faire des requêtes DNS à la main
**Pour *google.com* :**
```
$ nslookup
$ google.com
Nom :    google.com
Addresses:  2a00:1450:4007:819::200e
          142.250.179.110
```

**Pour *ynov.com* :**
```
$ nslookup
$ ynov.com
Nom :    ynov.com
Addresses:  2606:4700:20::681a:be9
          2606:4700:20::ac43:4ae2
          2606:4700:20::681a:ae9
          172.67.74.226
          104.26.10.233
          104.26.11.233
```

**Pour l'adresse *231.34.113.12* :**
```
$ nslookup
$ 231.34.113.12
Serveur :   UnKnown
Address:  fe80::887:c7ff:fed5:d764

*** UnKnown ne parvient pas à trouver 231.34.113.12 : Non-existent domain
```

**Pour l'adresse *78.34.2.17* :**
```
$ nslookup
$ 78.34.2.17
Serveur :   UnKnown
Address:  fe80::887:c7ff:fed5:d764

Nom :    cable-78-34-2-17.nc.de
Address:  78.34.2.17
```