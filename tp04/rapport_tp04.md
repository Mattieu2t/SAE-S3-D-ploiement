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

>Nginx et Apache sont les deux serveurs web les plus populaires et à eux deux, ils sont responsables de plus de la moitié du trafic sur Internet.

1. Apache a été lancé en 1995, il est maintenu par la fondation Apache Software. Il alimente actuellement environ *34%* des sites web, il occupe la première place en termes de parts de marché.

2. Apache est compatible avec de nombreux systèmes d'exploitation, Windows; OpenVMS et tout les systèmes de type Unix.

3. Si apache est si populaire, c'est en raison de sa puissance et de sa flexibilité. En effect, il dispose d'un système de modules qui permet aux utilisateurs de rajouter des fonctions, pour pouvoir modifier leur serveur en fonction de leurs besoins.

4. Apache dispose de *50* modules officiels.

5. Nginx est aussi un logiciel libre et gratuit. Il est l'un des serveurs les plus fiables en raison son évolution et de sa vitesse. C'est l'un des serveurs avec la plus grande croissance sur le marché.

6. Nginx a commencé à être développer en 2002 par Igor Sysoev car à l'époque peu de serveurs web étaient en mesure de supporter plus de 10 000 connexions simultanément.
En plus du serveur web, nginx peut être utilisé comme équilibreur de charge pour améliorer l'efficqcité et la disponibilité des ressources d'un serveur. Il peut aussi être utilisé comme reverse proxy.

7. Nginx supporte preque tous les systèmes d'exploitation de type Unix. Cependant sur Windows, peut entraîner certaines limitations de performences.

8. Pour servir du contenu statique, nginx est plus rapide que apache car il met en cache les fichiers statiques pour les rendre disponible dès qu'ils sont demandés.

9. Nginx dispose de *100* modules officiels.

>Nginx est multifonctionnel, il est plus performant et consomme moins de ressources que Apache, c'est donc pour ces différentes raison que nous avons choisi d'utiliser Nginx pour la SAE.

## Installer le serveur web que vous avez choisi et le configurer de façon à ce que l’application Element soit accessible sur le port 8080 sur la machine virtuelle.

>Pour pouvoir installer nginx pour qu'il puisse servir l'application Element sur le port 8080, il faut s'assurer d'avoir accès en tant qu'utilisateur administrateur ou avoir avez les privilèges nécessaires pour effectuer l'installation.

Installer Nginx en exécutant : 

1. `sudo apt-get update`

2. `sudo apt-get install nginx`

>Il faut ensuite vérifier qu'il est correctement installé avec la commande :

`nginx -v`

>Il faut aller modifier le fichier de configuration pour que nginx sache comment ouvrir Element : 

`sudo nano /etc/nginx/sites-available/default`

>Il faut ensuite reboot Nginx avec la commande :

`sudo systemctl restart nginx`

>En accèdant à **http://localhost:8080/** Element devrait apparaître.

## Ajouter une redirection dans votre configuration SSH pour que le serveur web soit également accessible sur le port 8080 de la machine de virtualisation.

>Il faut modifier le fichier : 

`sudo nano /etc/ssh/sshd_config`


>Il faut redemareer le système ssh :

`sudo systemctl restart ssh`

>Il faut aussi installer Element : 

Il faut aller ici : 
  
  `sudo mkdir /var/www/`


>On récupère l'archive de Element :

  `sudo -E weget` https://github.com/vector-im/element-web/releases/download/v1.11.16/element-v1.11.16.tar.gz

>On déarchive :

   `sudo rm -vrf element/`
>Ensuite on attribue la propriété du répertoire avec la variable d’environnement **$USER**
  
  `sudo chown -R $USER:$USER /var/www/element`
  
  `sudo chmod -R 755 /var/www/element`

>Pour qu’Apache puisse servir ce contenu, il faut créer un fichier d’hôte virtuel avec les directives correctes. On en crée un fichier avec la commande :

  `sudo nano /etc/apache2/sites-available/element.conf`

>On modifie le bloc de configuration similaire à celui par défault :

```
  <VirtualHost *:8080>
      ServerAdmin webmaster@localhost
      ServerName element
      ServerAlias www.element
      DocumentRoot /var/www/element
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
```


>On active le fichier de configuration avec :

  `sudo a2ensite element.conf`

>On déactive le fichier par défault :

  `sudo a2dissite 000-default.conf`


##  Se renseigner sur la notion de reverse proxy et trouver quelques logiciels capable d’agir comme tels (au moins 3). Comme précédemment, il s’agit de décider de critères de sélection et d’évaluer les différentes solutions avant de choisir celui que vous utiliserez.

>Un reverse proxy, à l'inverse du proxy, permet aux utilisateux externes d'accèder à une ressource du réseau interne.

>Lors d'une requête, c'est le reverse proxy qui s'occupe de contacter le serveur cible à notre place pour nous retourner la réponse. Le client n'a donc aucune visibilité sur les serveurs cachés derrière le reverse proxy.

>Le reverse proxy est une protection pour les serveurs du réseaux interne.

1. Pour mettre en place un reverse proxy, il y a Apache et le module *mod_proxy*, Ngnix est son module *ngx_http_proxy_module*.

2. Pour Windows, il a **IIS**, qui dispose aussi d'un module de reverse proxy directement inclus à Windows.

3. Il existe aussi HAProxy, avec son module *proxy*.

4. Il y a aussi Squid qui dispose d'un reverse proxy.

5. Nous utiliseront nginx, avec son module *ngx_http_proxy_module*.

## Installation d’un reverse proxy

>Pour créer une nouvelle machine, nous pouvons nous aider du rapport du **tp01** afin de savoir comment configurer la machine.

### Sur cette nouvelle machine virtuelle, installer un reverse proxy qui écoute sur le port 80 et redirige toutes les requêtes vers le serveur matrix (et donc vers 192.168.194.3 en port 8008).

>Pour installer le reverse proxy, il faut installer Nginx comme indiqué plus haut.

>Une fois fait, il faut créer un nouveau fichier de configuration :

`sudo nano /etc/nginx/conf.d/matrix.conf`

```
location / {
    proxy_pass http://192.168.194.3:8008;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

>Recharger Nginx avec la commande : 

`sudo nginx -s reload`

### Modifier vos paramètres de redirection SSH pour que le serveur matrix ne soit plus accessible directement sur le port 8008 de la machine de virtualisation mais que ça soit le reverse proxy qui soit accessible (sur le port 8080 de la machine de virtualisation).

>Il faut modifier  le .ssh/config de la machine de virtualisation  :

```
  Host rproxy
        User user
        HostName 192.168.194.4
        ForwardAgent yes
        LocalForward 0.0.0.0:9090 localhost:8
```