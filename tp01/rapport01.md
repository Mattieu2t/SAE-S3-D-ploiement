---
title: Rapport du TP-01 de la SAE 3.02
author: Gabriel Aumasson-Leduc
description: Procédures du TP01
date: 2022-11-22
---

# Rapport TP-01

## Connexion a distance
Afin de se connecter à distance nous utiliserons le protocole SSH
Nous utiliserons également 3 machines au total :
- la machine physique sur laquelle on travaillle
- la machine de virtualisation sur laquelle s'executeront les VMs
- la machine virtuelle crée dans la machine de virtualisation à l'aide de VirtualBox


Personnellement j'utiliserai la machine ***bouleau10*** pour éxecuter mes VMs

### Clé SSH de la machine de virtualisation
Afin de me connecter à la machine de virtualisation je rentre dans un terminal la commande : ```ssh bouleau10.iutinfo.fr ```
Pour la première connection je dois vérifier les clés SSH données afin de vérifier qu'on se connecte à la bonne machine.

La clé RSA attendue pour la machine ***bouleau10*** est ```SHA256:Tcd6H/GAL04UmfZ8QZnS/6en3jbVozkjdJ9uArAaEpg``` 

La clé ECDSA attendue pour la machine ***bouleau10*** est ```SHA256:+GogFtAKmffpYy+VXTFlcUegQc8ewCqlBoXI3Y6hPAY``` 

La clé ED25519 attendue pour la machine ***bouleau10*** est ```SHA256:yf+1Jdn95Yf4f1qL1UEZ8pZdKRp450uqlbuc5v9ZRyc``` 

### Fabriquer une paire de clés SSH
Afin de simplifier la connexion à la machine de virtualisation nous allons fabriquer une paire de clé publique-privée, ces clés fabriquées ne sont valables que sur la machine sur laquelle elles ont été produites

Afin de générer une paire de clé on utilisera la commande : ```ssh-keygen```
Il sera alors demandé un nom pour les fichiers générés (par défaut les fichers seront **id_rsa** et **id_rsa.pub**)
Il est également recommandé de metrte un mot de passe aux clés générées afin d'éviter leur utilisation par une tierce personne ayant accès à la même session. 

### Copier la clé publique sur le serveur distant
Après avoir généré les clés, on peut envoyer notre clé **PUBLIQUE** sur le serveur afin de s'y connecter plus rapidement par la suite
La commande appropriée pour cela est : ```ssh-copy-id -i <pathToPublicKey> user@serverAdress```
Ici j'ai donc fait dans mon cas ```ssh-copy-id -i ~/.ssh/id_rsa.pub gabriel.aumassonleduc.etu@bouleau10.iutinfo.fr```

Ainsi lors des prochaines connexions, je n'aurais besoin de rentrer uniquement le mot de passe de la clé

## Gestion des machines virtuelles

### Gestion des VMs
Afin de gérer les VMs nous allons utiliser un script écrit par monsieur Beaufils.
Ce dernier se trouve dans le répertoire ```/home/public/vm/vm.env```
Afin de le charger à chaque fois dans un terminal et donc de ne pas avoir à répéter l'opération, je conseille d'ajouter la ligne suivante à votre fichier **.bashrc** : ```source /home/public/vm/vm.env```

Voici une petite liste des commandes utiles pour gérer les VMs :
- Créer une VM : ```vmiut creer <nomDeLaVM>```
- Lister les VM existantes : ```vmiut lister```
- Démarrer une VM : ```vmiut demarrer <nomDeLaVM>```
- Arreter une VM : ```vmiut arreter <nomDeLaVM>```
- Supprimer une VM : ```vmiut supprimer <nomDeLaVM>```
- Obtenir des info sur une VM (une fois uniquement la machine démarrée) : ```vmiut info```

### Utilisation des VMs
Par défaut les VMs sont configuré avec 2 utilisateurs :
- un utilisateur "user" dont le mot de passe est : user
- un administrateur "root" dont le mot de passe est : root
- empreinte de la clé SSH: ```SHA256:SUHhxVJVZFiBQ6/koNbZfA9reKHyzIrvPgJvOEJ8zuE```

Concernant le réseau des VMs, ces dernière utilisent un réseau privé virtuel :
- l'adresse de réseau est : 192.168.194.0/24
- l'adresse de la machine de virtualisation est : 192.168.194.1
- l'adresse du routeur et du DNS est : 192.168.194.2
- l'adresse de la machine (attribuée automatiquement par virtual box et obtenable via la commande ```vmiut info```) : 192.168.194.25-192.168.194.128

### Connexion à une VM
Afin de se connecter à une machine on a 2 possibilité :
1. Via une console virtuelle
2. Via le SSH

Pour l'option 1 : 
- il faut être connecté avec la redirection graphique à la machine de virtualisatin ```ssh -X user@machineDeVirtualisation```
- il faut lancer la vm avec les commandes ```vmiut ``` vues plus haut
- lancer la commande ```vmiut console <nomDeLaVM>```

Pour l'option 2 : 
- il faut lancer la VM avec les commandes ```vmiut``` vues plus haut
- il faut obtenir l'IP de la VM avec les commandes ```vmiut``` vues plus haut
- enfin se connecter en ssh sur l'IP de la VM en tant qu'utilisateur ```ssh user@XXX.XXX.XXX.XXX```

## Configuration de la première machine virtuelle

### Changer la configuration réseau de la machine
Ces opération doit se faire impérativement via le mode **console virtuelle** et en tant que **root**.

#### Changer la default gateway
Pour changer le routeur par défaut  il faut modifier le fichier */etc/resolve.conf* et y ajouter le champ : ```gateway XXX.XXX.XXX.XXX``` en remplace les X par l'addresse IP voulue. 

#### Changer l'adresse IP de la VM
Pour changer l'addresse IP il faut
- couper l'interface réseau enp0s3 ```root@matrix:~# ifdown enp0s3```
- ouvrir le fichier */etc/newtork/intefaces* en tant que ***ROOT***
- remplacer les lignes du  bloc concernant enp0s3 par ```
    address 192.168.194.3  
    netmask 255.255.255.0  
    gateway 192.168.194.2  ``` en mettant l'ip voulue, le masque voulu et la gateway voulue
- relancer l'interface réseau enp0s3 ```root@matrix:~# ifup enp0s3```
- redemarrer la VM pour vérifier que la config est bien prise en compte ``` root@matrix:~# reboot ```
et arpès rédemarrage ``` root@matrix:~# ip a ``` ou l'interface enp0s3 devrait avoir pris les modifications faites


### Accès root à la machine
On ne peut pas se connecter directement en tant que root à la VM par le SSh. Cette connexion a été bloquée. Il faut donc se connecter par le compte user et utiliser la commande ```user@matrix:~$ su -l``` qui permet de charger les variable d'environnement du root tout en devenant root


### Configuration du proxy
En tant que **root** modifier le fichier /etc/environment et y ajouter ceci ```HTTP_PROXY=http://cache.univ-lille.fr:3128
HTTPS_PROXY=http://cache.univ-lille.fr:3128
http_proxy=http://cache.univ-lille.fr:3128
https_proxy=http://cache.univ-lille.fr:3128
NO_PROXY=localhost,192.168.194.0/24,172.18.48.0/22```

Cela permettra que la VM utilise le proxy de l'université
pour verifier le bon fonctionnement ``` root@matrix:~# apt update && apt upgrade ``` (ces commandes permettent aussi de mettre à jour le serveur)
Si cela fonctionne alors le proxy est correctement configuré


### Installation d'outils
Utiliser la commande ``` root@matrix:~# apt install <package> ``` pour installer les packages voulus. Donc pour installer les packages vim less et rsync on fera 
``` root@matrix:~# apt install -y less vim rsync ```


## Configuration ssh rapide

On peut ajouter des configurations personnalisées a ssh afin d'accélerer le temps d'accès
ces modifications se font dans le fichier ```$HOME/.ssh/config```
afin de créer un alias on ajoutera dans le fichier 
```
    host virt 
        HostName bouleau10.iutinfo.fr
```

si on veut utliser une clé ajouter en dessous 
    ```IdentityFile ~/.ssh/<OnePublicKey.pub>```
    
si on veut transférer sa clé pour faire plusieurs connexion d'affilée en SSH 
    ```ForwardAgent yes```

si on veut directement se connecter sans préciser le nom d'utilisateur à chaque fois ajouter : 
    ```User user```


