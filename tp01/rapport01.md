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
Pour la première connection je dois vérifier les clés SSH données afin de vérifier qu'on se connecte à la bonne machine

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
- il faut lancer la VM avec les commandes ```vmiut ``` vues plus haut
- il faut obtenir l'IP de la VM avec les commandes ```vmiut ``` vues plus haut
- enfin se connecter en ssh sur l'IP de la VM en tant qu'utilisateur```ssh user@XXX.XXX.XXX.XXX```