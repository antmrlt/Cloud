# TP1

🌞 **Installer Docker sur la machine**

- en suivant [la doc officielle](https://docs.docker.com/engine/install/)
- démarrer le service `docker` avec une commande `systemctl`
- ajouter votre utilisateur au groupe `docker`
  - cela permet d'utiliser Docker sans avoir besoin de l'identité de `root`
  - avec la commande : `sudo usermod -aG docker $(whoami)`
  - déconnectez-vous puis relancez une session pour que le changement prenne effet

Commande docker run
```
sudo docker run --name web -d -v $(pwd)/web/index.html:/usr/share/nginx/html/index.html -v $(pwd)/conf/tp1.conf:/etc/nginx/conf.d/tp1.conf -p 8080:80 nginx
```