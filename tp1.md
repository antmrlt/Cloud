# TP1

🌞 **Installer Docker sur la machine**

- en suivant [la doc officielle](https://docs.docker.com/engine/install/)
- démarrer le service `docker` avec une commande `systemctl`
- ajouter votre utilisateur au groupe `docker`
  - cela permet d'utiliser Docker sans avoir besoin de l'identité de `root`
  - avec la commande : `sudo usermod -aG docker $(whoami)`
  - déconnectez-vous puis relancez une session pour que le changement prenne effet

**Commande docker run**s
```
sudo docker run --name web -d -v $(pwd)/web/index.html:/usr/share/nginx/html/index.html -v $(pwd)/conf/tp1.conf:/etc/nginx/conf.d/tp1.conf -p 8080:80 snginx
```

**Conf file**
```
server {
  # on définit le port où NGINX écoute dans le conteneur
  listen 8080;

  # on définit le chemin vers la racine web
  # dans ce dossier doit se trouver un fichier index.html
  root /usr/share/nginx/html;
}
```

🌞 **Construire votre propre image**

- image de base (celle que vous voulez : debian, alpine, ubuntu, etc.)
  - une image du Docker Hub
  - qui ne porte aucune application par défaut
- vous ajouterez
  - mise à jour du système
  - installation de Apache (pour les systèmes debian, le serveur Web apache s'appelle `apache2` et non pas `httpd` comme sur Rocky)
  - page d'accueil Apache HTML personnalisée

**dockerfile**
```
FROM ubuntu

RUN apt-get update && apt-get upgrade -y

RUN apt install -y apache2

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

On build l'image avec `sudo docker build . -t dockerfile`