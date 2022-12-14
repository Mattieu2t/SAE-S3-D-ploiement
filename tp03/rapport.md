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
