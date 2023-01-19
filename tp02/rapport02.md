---
title: Rapport du TP-02 de la SAE 3.02
author: Gabriel Aumasson-Leduc
description: Procédures du TP02
date: 2022-11-22
---

# Rapport TP-02

## Dernière connexion à la VM

### Changement de nom de la machine
Pour changer le nom de la machine on doit écrire dans 2 fichiers en tant que **root**
- /etc/hostname
- /etc/hosts

Dans le premier fichier il faut remplacer ```debian``` par le nom choisi pout la VM
Et dans le deuxième fichier, sur la deuxième ligne remplacer à nouveau ```debian``` par le nom choisi dans le premier fichier.

Pour valider les modifications, redémarrez la VM ```root@debian:~# reboot```

### Installation et configuration de la commande ```sudo```

Installer la commande en tant que root ```root@debian:~# apt install sudo```

Et enfin d'ajouter l'utilisateur 'user' en tant que root entrer la commande ```root@debian:~# adduser <user> sudo```
Redémarrez la VM pour valider l'ajout
Afin de vérifier l'ajout de l'utilisateur 'user' en tant que root faites la commande  ```user@debian:~$ sudo -E apt update```
Vous ne devriez pas avoir de problèmes. 

### Configuration de la synchronisation de l'horloge
Pour vérifier la date et l'heure de la machine il suffit d'utiliser la commande ```date``` dans le terminal
Afin d'utiliser le serveur NTP de l'université de Lille il faut :
- configurer le serveur dans le fichier /etc/systemd/timesyncd.conf
- - ouvrir le fichier dans un éditeur (nano par exemple)
- - retirer le symbole # devant la ligne ou se situe le mot NTP
- - entrer ```NTP=ntp.univ-lille.fr```
- - sauvegarder et fermer le fichier
- redémarrer le service à l'aide de la commande ```systemctl restart systemd-timesyncd.service```
Puis relancer la commande ```date``` afin de vérifier la mise à jour de l'heure. 

## Installation et configuration basique d’un serveur de base de données

### Installer PostgreSQL

En tant que root : ```root@vm:~# apt install postgresql postgresql-client```  

Afin de vérifier l'etat de postgre : ```root@vm:~# systemctl status postgresql```

###  Creer un utilisateur avec PostgreSQL
Pour pouvoir créer un utilisateur de la base de donnée postgre il faut se connecter au seul eôle qui a les droits de gérer la base de données : postgres
```su -l postgres```

Afin de créer l'utilisateur :  ```createuser -W -d <username>``` (ajoute l'utilisateur qui a les droits de créer sa base de données)

Il faudra alors spécifier un mot de passe. 

### Creer une base de données appartenant à un utilisateur
La commande ```createdb -O <proprietaire> -E UTF-8 <nom de la bdd>``` permet de créer une base de données appartenant à l'utilisateur spécifié

### Se connecter à la base de données 
Pour se connecter à la base en tant que 'user' la commande est : ```psql -U <username> -W <database> -h 127.0.0.1```
On utilise l'option -U pour spécifier un utilisateur
On utilise l'option -W pour spécifier qu'il faut un mot de passe
On utilise l'option -h pour spécifier un host qui héberge la base de données (on se connecte ici par le mode IP-V4 qui utilise un mot de passe et non par le mot par défaut (peer-mode))


