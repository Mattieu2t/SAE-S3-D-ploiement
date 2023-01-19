---
title: Rapport du TP-05 de la SAE 3.02
author: Gabriel Aumasson-Leduc / Seneschael Mattieu
description: Procédures du TP05
date: 2023-01-4
---

# Rapport TP-05

## Migration vers l’architecture finale

### Description de l’architecture finale :

> Pour toutes les machines virtuelles, vous devez appliquer les configurations minimales suivantes :

1. proxy http,

2. synchronisation du temps avec NTP,

3. déploiement de vos clés SSH,

4. configuration correcte de sudo,

5. configuration correcte du nom d’hôte et du fichier de résolution DNS local : toutes les machines doivent pouvoir connaître l’adresse de toutes les autres à partir de leur nom ;

>Pour supprimer la machine **matrix** :

`vmiut supprimer matrix`

>Pour créer une nouvelle machine **matrix**

`vmiut creer matrix`

>Pour configurer la machine **rproxy** :

`ssh -X frene14`


`vmiut demarrer rproxy`


`vmiut console rproxy`


>Se connecter en *root*

`ifdown enp0s3`
>Il faut ensuite aller modifier le fichier : 

`nano /etc/network/interfaces`

```
        iface enp0s3 inet static
            address 192.168.194.3
            gateway (routeur) 192.168.194.2
            netmask 255.255.255.0
```
>Puis aller modifier le fichier :
 
`nano /etc/rosolv.conf`

`nameserver 192.168.194.2`

>Il faut ensuite réactiver *enp0s3* et redémarrer la machine :

`ifup enp0s3`

`reboot`

>Modifier la machine **matrix** : 

>Se connecter à la machine : 

`ssh user@192.168.194.3`
>Passer en root :

`su --login root`
>Créer le fichier *environment* :

`nano /etc/environment`

```
        HTTP_PROXY=http://cache.univ-lille.fr:3128
        HTTPS_PROXY=http://cache.univ-lille.fr:3128
        http_proxy=http://cache.univ-lille.fr:3128
        https_proxy=http://cache.univ-lille.fr:3128
        NO_PROXY=localhost,192.168.194.0/24,172.18.48.0/22
```

>Puis redémarrer la machine à l'aide de `reboot`.

>Changer le nom de la machine :

`nano /etc/hostname`
    
>Changer **debian** en **matrix** (il faut être en root)

>Modifier le fichier :

`nano /etc/hosts`

>Changer debian en matrix : 
```
    192.168.194.4   Pour le rproxy
    192.168.194.5   Pour la base de données
    192.168.194.6   Pour element
```

>Il est maintenant possible de ping **matrix**

>Configuration de la synchronisation de l'horloge:

`nano /etc/systemd/timesyncd.conf`

>Rajouter la ligne : 

`NTP=ntp.univ-lille.fr`

>Puis redémarrer systemctl avec la commande : 

`sudo systemctl restart systemd-timesyncd.service`


>Installation et configuration de la commande sudo :

`apt install sudo `

`usermod -G sudo user`

Pour supprimer un user --> `sudo passwd -l NOM_USER puis sudo userdel NOM_USER`

Pour voir les user --> `nano /etc/passwd`

Pour voir les groupes --> `nano /etc/group`

>Installation de Synapse (en root) :

`apt install -y lsb-release wget apt-transport-https`

`wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg`

`echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/matrix-org.list`

`apt update`

`apt install matrix-synapse-py3`

>Au moment de l’installation, le gestionnaire de paquets demande le nom de l'instance. Il faut indiquer frene14.iutinfo.fr:9090.

>Pour que le reverse proxy soit accessible via la machine de virtualisation sur le port 9090, il faut rajouter cette configuration dans le fichier ssh : 

```
Host vmElement
HostName 192.168.194.4
User user
LocalForward 0.0.0.0:9090 localhost:80
#LocalForward 0.0.0.0:9090 localhost:8008
```