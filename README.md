# Reseau_tp_1

>> Pour un PC Windows, on utilise la commande **ipconfig /all** :
``````
ipconfig /all
``````

## I. Exploration locale en solo
### 1. Affichage d'informations sur la pile TCP/IP locale

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

### 2. Modifications des informations

A. Modification d'adresse IP (part 1)

🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :
> On refait le même schéma que pour afficher les informations de la cart IP mais on change les propriétés.

🌞 Il est possible que vous perdiez l'accès internet.
> Il est possible de perdre l'accès internet car si on a exactement la même adresse qu'un autre utilisateur, il ne peut pas envoyer sur les deux mêmes utilisateurs.


## II. Exploration locale en duo

### 3. Modification d'adresse IP
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

### 4. Petit chat privé

## III. Manipulations d'autres outils/protocoles côté client

## 1.DHCP