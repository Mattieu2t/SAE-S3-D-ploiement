---
title: Rapport du TP-01 de la SAE 3.02
author: Gabriel Aumasson-Leduc
description: Procédures du TP01
date: 2022-11-22
---

# Rapport TP-03

## Accès à un service HTTP sur la VM

### Un premier service pour tester
Installation de NGONX et Curl : ```root@vm:~#: apt install curl nginx```
Verification que NGINX est installé ```root@vm:~#: systemctl status nginx```

Verifcations que le service NGINX est fonctionnel : ```user@vm:~$ curl http://localhost```
Pour le moment on ne peut pas accéder au serveur Web NGINX depuis la machine de virtualisation (*qui est bouleau10 dans mon cas*)
Pour faire cet accès on peut utiliser la commande```user@virtu:~$ curl --noproxy '*' http://192.168.194.3/``` ce qui permet de récuperer la page sans passer par le proxy de l'université de Lile (ce dernier n'a pas connaissances des VMs)

### Accès au service depuis la machine physique
On ne peut pas accéder directement au service par défaut car la VM est "separée" du réseau des machines de l'IUT. Pour y accéder depuis n'importe quelle machine, il faudrait faire une redirection entre l'interface de la VM et celle des PC de l'IUT (donc entre la VM et les machines de virtualisations)

2 méthodes : 
- le faire une seule fois avec l'option -L de la commande ssh
- faire une redirection "permanente" via le fichier de configuration SSH ```.ssh/config```

Dans le premier cas on utilisera la commande ```user@virtu:~$ ssh -L 172.18.48.176:9090:192.168.194.3:80 user@192.168.194.3```
Ici la premiière IP suivie du port correspond à la machine sur laquelle le lien doit être accessible (ici l'ip de bouleau10), la deuxième IP celle de la machine de virtualisation et enfin la connexion user@host habituelle en SSH.

Dans le deuxième cas on modifiera le fichier de configuration SSH en y ajoutant l'option suivante : ```LocalForward 0.0.0.0:9090 192.168.194.3:8080``` ou 0.0.0.0:9090 sera l'IP de la machine qui se connecte en SSH sur le port 9090 et la deuxième IP celle de la VM
## Installation de synapse

### Installation de synapse sous Debian
Pour installer le serveur Synapse on suit la documentation officielle en installant la dernière release
```
user@vm:~$ sudo apt install -y lsb-release wget apt-transport-https
sudo wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" |
    sudo tee /etc/apt/sources.list.d/matrix-org.list
sudo apt update
sudo apt install matrix-synapse-py3 
``` 

Pour le nom d'instance on choisira bien bouleau10.iutinfo.fr:8008
Le serveur écrira ses messages à destination de l’administrateur (les logs) dans le fichier /var/log/matrix-synapse/homeserver.log

### Paramétrage pour un réseau privé
On va passer la variable ```trusted_key_server``` à la valeur ```[]``` dans le fichier ```/etc/matrix-synapse/homeserver.yaml```

### Utilisation d'une base postgres
On va modifier la configuration dans le fichier ```/etc/matrix-synapse/homeserver.yaml``` en tant que **ROOT** pour utiliser une base de données PostgreSQL au lieu de SqLite
Dans la section ***database*** du fichier on mettra les lignes suivantes : 
```
name: psycopg2
  args:
    user: <user>
    password: <pass>
    database: <db>
    host: <host>
    port: 5432
    cp_min: 5
    cp_max: 10
``` 
On remplacera le user par le nom d'utilisateur choisi, de même pour le mot de passe et la database, enfin on mettra l'adresse ipV4 du host. Le port 5432 est celui utilisé par défaut pour PostgreSQL, s'il a été modifié faites en de même dans ce fichier. Sauvegardez le fichier et le quitter. 

Nous allons également créer un utilisateur, un mot de passe et une databse spécifique à l'utilisation du serveur Synapse
Pour cela passer sous l'utilisateur *"postgres"*. 
```
root@vm:~# su -l postgres
postgres@vm:~$ createuser --pwprompt <synapse_user>
postgres@vm:~$ createdb --encoding=UTF8 --locale=C --template=template0 --owner=<synapse_user> <synapse>
``` 
On remplacera <synapse_user> par un nom d'utilisateur voulu
et <synapse> par un nom de base de données au choix.

Redemarrez le service de synapse pour que les modifications prennent effet 
```
root@vm:~# systemctl restart matrix-synapse
```

### Creation d'utilisateur matrix

Pour définir un shared_secret on va créer un fichier sharedSecret.yaml contenant "registration_shared_secret: "ma_cle_partagee"" dans le dossier /etc/matrix-synapse

On utilisera la commande : 
```
register_new_matrix_user -c /chemin/vers/fichier/de/configuration -u <mon_nom_dutilisateur> -p <mon_mot_de_passe> -a
```
Afin de créer un utilisateur qui est également administrateur

### Autorisez l'ajout de nouveaux utilisateurs
On ajoutera 2 lignes dans le fichier de configuration 
```/etc/matrix-synapse/homeserver.yaml```
```
enable_registration: True
enable_registration_without_verification: True
```

Redémarrez à nouveau le service pour valider la configutration