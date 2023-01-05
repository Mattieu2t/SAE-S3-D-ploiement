---
title: Rapport du TP-04 de la SAE 3.02
author: Gabriel Aumasson-Leduc / Seneschael Mattieu
description: Procédures du TP04
date: 2022-12-4
---

# Rapport TP-04

## Installation et configuration de Element Web

### Element Web

### Nginx vs Apache :

Nginx et Apache sont les deux serveurs web les plus populaires et à eux deux, ils sont responsables de plus de la moitié du trafic sur Internet.

Apache a été lancé en 1995, il est maintenu par la fondation Apache Software. Il alimente actuellement environ 34% des sites web, il occupe la première place en termes de parts de marché.

Apache est compatible avec de nombreux systèmes d'exploitation, Windows; OpenVMS et tout les systèmes de type Unix.
Si apache est si populaire, c'est en raison de sa puissance et de sa flexibilité. En effect, il dispose d'un système de modules qui permet aux utilisateurs de rajouter des fonctions, pour pouvoir modifier leur serveur en fonction de leurs besoins.

Apache dispose de 50 modules officiels.

Nginx est aussi un logiciel libre et gratuit. Il est l'un des serveurs les plus fiables en raison son évolution et de sa vitesse. C'est l'un des serveurs avec la plus grande croissance sur le marché.

Nginx a commencé à être développer en 2002 par Igor Sysoev car à l'époque peu de serveurs web étaient en mesure de supporter plus de 10 000 connexions simultanément.
En plus du serveur web, nginx peut être utilisé comme équilibreur de charge pour améliorer l'efficqcité et la disponibilité des ressources d'un serveur. Il peut aussi être utilisé comme reverse proxy.

Nginx supporte preque tous les systèmes d'exploitation de type Unix. Cependant sur Windows, peut entraîner certaines limitations de performences.

Pour servir du contenu statique, nginx est plus rapide que apache car il met en cache les fichiers statiques pour les rendre disponible dès qu'ils sont demandés.

Apache dispose de 100 modules officiels.

Nginx est multifonctionnel, il est plus performant et consomme moins de ressources que Apache, c'est donc pour ces différentes raison que nous avons choisi d'utiliser Nginx pour la SAE.

## Installer le serveur web que vous avez choisi et le configurer de façon à ce que l’application Element soit accessible sur le port 8080 sur la machine virtuelle.

Pour pouvoir installer nginx pour qu'il puisse servir l'application Element sur le port 8080, il faut s'assurer d'avoir accès en tant qu'utilisateur administrateur ou avoir avez les privilèges nécessaires pour effectuer l'installation.

Installer Nginx en exécutant : 

sudo apt-get update

sudo apt-get install nginx

Il faut ensuite vérifier qu'il est correctement installé avec la commande :

nginx -v

Il faut aller modifier le fichier de configuration pour que nginx sache comment ouvrir Element : 

sudo nano /etc/nginx/sites-available/default

Il faut ensuite reboot Nginx avec la commande : 

sudo systemctl restart nginx

En accèdant à http://localhost:8080/ Element devrait apparaître.

## Ajouter une redirection dans votre configuration SSH pour que le serveur web soit également accessible sur le port 8080 de la machine de virtualisation.

##  Se renseigner sur la notion de reverse proxy et trouver quelques logiciels capable d’agir comme tels (au moins 3). Comme précédemment, il s’agit de décider de critères de sélection et d’évaluer les différentes solutions avant de choisir celui que vous utiliserez.

Un reverse proxy, à l'inverse du proxy, permet aux utilisateux externes d'accèder à une ressource du réseau interne.
Lors d'une requête, c'est le reverse proxy qui s'occupe de contacter le serveur cible à notre place pour nous retourner la réponse. Le client n'a donc aucune visibilité sur les serveurs cachés derrière le reverse proxy.

Le reverse proxy est une protection pour les serveurs du réseaux interne.

Pour mettre en place un reverse proxy, il y a Apache et le module *mod_proxy*, Ngnix est son module *ngx_http_proxy_module*.

Pour Windows, il a IIS, qui dispose aussi d'un module de reverse proxy directement inclus à Windows.

Il existe aussi HAProxy, avec son module *proxy*.