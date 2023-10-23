# <span style="color:red">Réseau - TP 2

## <span style="color:orangered">I. Setup IP

🌞 Mettez en place une configuration réseau fonctionnelle entre les deux machines
> Pour changer l'adresse IP et le masque sous-réseau : **[Panneau de configuration/Réseau et internet/Connexions réseau/Propriétés de Ethernet/Protocole Internet version 4 (TCP/IPv4)]**

```
$ ipconfig
Carte Ethernet Ethernet :

   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.22
```

> L'adresse de réseau : ```10.10.10.20```

> L'adresse broadcast : ```10.10.10.23```

🌞 Prouvez que la connexion est fonctionnelle entre les deux machines
```
$ ping 10.10.10.21

Envoi d’une requête 'Ping'  10.10.10.21 avec 32 octets de données :
Réponse de 10.10.10.21 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.21 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.21 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.10.21 : octets=32 temps=3 ms TTL=128

Statistiques Ping pour 10.10.10.21:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 3ms, Moyenne = 2ms
```

🌞 Wireshark it
> echo request (ping) : **8**

> echo reply (pong) : **0**

[Différence entre **echo request** et **echo reply**](./echo_8_&_0.pcapng)


## <span style="color:orangered">II ARP my bro

🌞 Check the ARP table
> Adresse physique de Brendan :
```
$ arp -a
Interface : 10.10.10.22 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.10.10.21           84-69-93-59-20-70     dynamique
```

> Adresse de la passerelle d'Ynov

```
$ arp -a 10.33.79.254

Interface : 10.33.70.192 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.33.79.254          7c-5a-1c-d3-d8-76     dynamique
```

🌞 Manipuler la table ARP

> Pour vider la table arp, il faut ouvrir un PowerShell administrateur puis faire :
```
$ netsh interface IP delete arpcache
Ok.
```

> On constate dans le PowerShell administrateur qu'il y a moins d'adresses contrairement au PowerShell classique après avoir ```ping 10.10.10.21```

> PowerShell administrateur :
```
$ arp -a

Interface : 192.168.88.1 --- 0xa
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.70.192 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.33.65.106          d8-f2-ca-3b-76-ee     dynamique
  10.33.79.254          7c-5a-1c-d3-d8-76     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.10.10.22 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.10.10.21           84-69-93-59-20-70     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.126.1 --- 0x17
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  ```

  > PowerShell classique :
  ```
  $ arp -a

Interface : 192.168.88.1 --- 0xa
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.70.192 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.33.65.106          d8-f2-ca-3b-76-ee     dynamique
  10.33.79.254          7c-5a-1c-d3-d8-76     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.10.10.22 --- 0x14
  Adresse Internet      Adresse physique      Type
  10.10.10.21           84-69-93-59-20-70     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.126.1 --- 0x17
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  ```

🌞 Wireshark it
[Les deux trames ARP](./trames_ARP.pcapng)
```
adresse source : 10.10.10.22        adresse destination : 10.10.10.21
adresse source : 10.10.10.22        adresse destination : 2c:f0:5d:66:be:f2
```
> La source envoie au routeur qui reçoit puis celui-ci va chercher l'adresse MAC de la source.

## <span style="color:orangered">III. DHCP
🌞 Wireshark it
