# Dépôt GitHub

## [https://github.com/adxdits/docker-tp](https://github.com/adxdits/docker-tp)

# TP Docker

docker hub : adxdits
dossier : C:\Users\adamalmounayar\docker-tp

## What is the difference between an image and a container?

image = le modele (fichiers + config), on la modifie pas. conteneur = une instance de l'image qui tourne, avec son propre etat. on peut lancer plusieurs conteneurs avec la meme image.

## What is the difference between docker run and docker start?

docker run cree un nouveau conteneur et le lance. docker start relance un conteneur qui existe deja (arrete), avec ses donnees et sa config.

## What do -i and -t do, separately and together?

-i garde l'entree standard ouverte (on peut envoyer du texte)
-t donne un pseudo terminal
ensemble (-it) = shell interactif normal

## What happens if you omit -it when creating a container from this Debian image? Does that apply to every image?

sans -it le shell de debian a rien a lire donc il se ferme direct et le conteneur s'arrete. pas pour toutes les images, nginx ou mysql tournent tres bien sans -it.

## How do you remove a stopped container? How do you stop a running one first?

```
docker stop ostrogoth
docker rm ostrogoth
```

docker stop seulement si il tourne encore, sinon juste docker rm.

## What does -d do? What happens without it? Why are -i and -t unnecessary here?

```
docker run -d -p 5000:80 --name MyNgx nginx:stable
```

-d = detache, le conteneur tourne en fond et on recupere le terminal. sans -d le terminal reste bloque sur les logs de nginx. pas besoin de -it, nginx attend des requetes http pas du clavier.

## Submit the full image name, including its tag, and the exact command needed to run it.

C:\Users\adamalmounayar\docker-tp\myimage\Dockerfile :

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["echo", "Hello"]
```

```
cd C:\Users\adamalmounayar\docker-tp\myimage
docker buildx build --load -t adxdits/myimage:0.1 .
docker login -u adxdits
docker push adxdits/myimage:0.1
```

image : adxdits/myimage:0.1

```
docker run --rm adxdits/myimage:0.1
```

ca affiche Hello

## Build an image that runs your program, publish it, and submit its full name and run command.

programme en python, C:\Users\adamalmounayar\docker-tp\random-date\main.py :

```python
import random
from datetime import date

print("Bonjour, voici mon programe Docker")
for i in range(3):
    print(random.randint(1, 1000))
print("Date du jour :", date.today())
```

randint(1, 1000) inclut 1 et 1000.

Dockerfile :

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY main.py .
CMD ["python", "main.py"]
```

```
cd C:\Users\adamalmounayar\docker-tp\random-date
docker buildx build --load -t adxdits/random-date:0.1 .
docker push adxdits/random-date:0.1
```

image : adxdits/random-date:0.1

```
docker run --rm adxdits/random-date:0.1
```

affiche un texte, 3 nombres aleatoires entre 1 et 1000 et la date du jour

## Submit the command you used and the URL for accessing the website.

```
docker run -d --name apache -p 8080:80 -v C:\Users\adamalmounayar\docker-tp\website:/usr/local/apache2/htdocs:ro httpd:2.4
```

url : http://localhost:8080 (:ro = lecture seule)

## Submit the commands you used and your README.md. (MySQL)

lancer mysql sans publier de port :

```
docker run -d --name mysql-lab -e MYSQL_ROOT_PASSWORD=Adam1234 mysql:8.4
docker logs -f mysql-lab
```

attendre "ready for connections" port 3306 puis ctrl+c

```
docker exec -it mysql-lab mysql -u root -pAdam1234
```

```sql
CREATE DATABASE lab;
USE lab;
CREATE TABLE students (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50), course VARCHAR(50));
INSERT INTO students (name, course) VALUES ('Alex', 'Docker'), ('Sam', 'Reseaux'), ('Lina', 'BDD');
SELECT * FROM students;
exit
```

stop, start et verif (attendre quelques secondes apres le start) :

```
docker stop mysql-lab
docker start mysql-lab
docker exec mysql-lab mysql -u root -pAdam1234 -e "SELECT * FROM lab.students"
```

les lignes sont toujours la.

user student pour se connecter depuis la machine (on utilise pas root depuis l'exterieur) :

```
docker exec mysql-lab mysql -u root -pAdam1234 -e "CREATE USER 'student'@'%' IDENTIFIED BY 'Adam123root'; GRANT ALL ON lab.* TO 'student'@'%';"
```

recreer le serveur avec le port publie en gardant les donnees :

```
docker stop mysql-lab
docker rename mysql-lab mysql-old
docker run -d --name mysql-lab --volumes-from mysql-old -p 127.0.0.1:3306:3306 mysql:8.4
```

--volumes-from reprend le volume /var/lib/mysql de l'ancien conteneur donc la base est toujours la. faut pas supprimer mysql-old.

depuis windows (client mysql installe) :

```
mysql -h 127.0.0.1 -P 3306 -u student -pAdam123root lab
```

README.md :

```
# MySQL

image mysql:8.4, conteneur mysql-lab, base lab, table students
user student pour se connecter depuis la machine (mdp pas mis ici)

demarrer : docker start mysql-lab
attendre que ce soit pret : docker logs -f mysql-lab

se connecter dans le conteneur : docker exec -it mysql-lab mysql -u root -p
depuis windows : mysql -h 127.0.0.1 -P 3306 -u student -p lab
(port publie seulement sur 127.0.0.1)

arreter : docker stop mysql-lab
quitter le client mysql (exit) ca arrete pas le serveur

donnees : dans le volume anonyme de /var/lib/mysql, pas dans l'image
le nouveau conteneur le reprend avec --volumes-from mysql-old donc pas supprimer mysql-old ni le volume
changer MYSQL_ROOT_PASSWORD apres coup change rien, la base existe deja
```

## Submit your commands and an updated README.md. (phpMyAdmin)

```
docker network create myNet
docker stop mysql-lab
docker rename mysql-lab mysql-old2
docker run -d --name mysql-lab --network myNet --volumes-from mysql-old2 mysql:8.4
docker run -d --name pma --network myNet -e PMA_HOST=mysql-lab -p 8081:80 phpmyadmin:5.2-apache
```

http://localhost:8081, login student / Adam123root

mysql recree sans -p donc plus de port publie, donnees gardees avec --volumes-from. le user student existe deja, MYSQL_USER etc marchent que sur une base vide.

README.md :

```
# MySQL + phpMyAdmin

2 conteneurs sur le reseau bridge myNet : mysql-lab (mysql:8.4) et pma (phpmyadmin:5.2-apache)
pma se connecte a mysql avec PMA_HOST=mysql-lab, pma c'est juste l'interface web pas le serveur

demarrer : docker start mysql-lab puis docker start pma
ouvrir http://localhost:8081, login avec student + son mdp (pas mis ici)
arreter : docker stop pma mysql-lab

seul pma a un port (8081), mysql n'a plus de port publie
donnees : volume /var/lib/mysql repris avec --volumes-from mysql-old2, garder les anciens conteneurs arretes
MYSQL_DATABASE / MYSQL_USER / MYSQL_PASSWORD marchent que sur une base vide, sinon creer le user en SQL
```

## Submit the commands you used. (Named volume)

export :

```
docker exec mysql-lab mysqldump -u root -pAdam1234 --single-transaction --databases lab --result-file=/tmp/lab.sql
docker cp mysql-lab:/tmp/lab.sql C:\Users\adamalmounayar\docker-tp\lab.sql
```

nouveau conteneur avec un volume nomme (vide au debut, il recupere pas tout seul les anciennes donnees) :

```
docker stop mysql-lab
docker rename mysql-lab mysql-old3
docker run -d --name mysql-lab --network myNet -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Adam1234 mysql:8.4
docker logs -f mysql-lab
```

quand c'est pret, restore + recreer le user (le dump a pas les users) :

```
docker cp C:\Users\adamalmounayar\docker-tp\lab.sql mysql-lab:/tmp/lab.sql
docker exec mysql-lab sh -c "mysql -u root -pAdam1234 < /tmp/lab.sql"
docker exec mysql-lab mysql -u root -pAdam1234 -e "CREATE USER 'student'@'%' IDENTIFIED BY 'Adam123root'; GRANT ALL ON lab.* TO 'student'@'%';"
```

verif en recreant le conteneur avec le meme volume (attendre un peu avant le exec) :

```
docker stop mysql-lab
docker rm mysql-lab
docker run -d --name mysql-lab --network myNet -v mysql-data:/var/lib/mysql mysql:8.4
docker exec mysql-lab mysql -u root -pAdam1234 -e "SELECT * FROM lab.students"
```

les lignes sont toujours la, je garde le volume mysql-data.

backup / restore : un volume c'est pas une sauvegarde, si je supprime des lignes elles sont supprimees dans le volume aussi. pour sauvegarder : mysqldump avec --single-transaction (copie coherente des tables innodb sans arreter le serveur) et garder le .sql en dehors de docker. pour restaurer : nouveau conteneur avec un volume vide, docker cp du .sql, mysql < lab.sql, verifier les lignes. copier direct /var/lib/mysql pendant que ca tourne c'est pas fiable.

## Why is Compose useful even for one container?

tout est ecrit dans un fichier (image, ports, volumes, reseaux, variables) au lieu d'une longue commande docker run qu'il faut retenir. on peut ducoup le mettre sur github et le partager, et on lance / arrete tout avec docker compose up -d et docker compose down. en plus compose cree tout seul le volume et le reseau.

## Submit both compose.yaml files and the corresponding README.md files.

C:\Users\adamalmounayar\docker-tp\compose-nodered\compose.yaml :

```yaml
services:
  nodered:
    image: nodered/node-red
    ports:
      - "2000:1880"
    volumes:
      - save_nodered_data:/data

volumes:
  save_nodered_data:
```

README.md :

```
# Node-RED

cd C:\Users\adamalmounayar\docker-tp\compose-nodered
docker compose config
docker compose up -d

ouvrir http://localhost:2000, faire un flow (inject -> debug) et deploy

docker compose down puis docker compose up -d -> le flow est toujours la
donnees dans le volume save_nodered_data (/data)
pas faire docker compose down -v sinon ca supprime le volume
```

C:\Users\adamalmounayar\docker-tp\compose-mysql\compose.yaml :

```yaml
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: lab
      MYSQL_USER: student
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - myNet
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1"]
      interval: 5s
      retries: 30

  phpmyadmin:
    image: phpmyadmin:5.2-apache
    environment:
      PMA_HOST: mysql
    ports:
      - "3000:80"
    networks:
      - myNet
    depends_on:
      mysql:
        condition: service_healthy

networks:
  myNet:

volumes:
  mysql_data:
```

.env dans le meme dossier (pas rendu) :

```
MYSQL_ROOT_PASSWORD=Adam1234
MYSQL_PASSWORD=Adam123root
```

README.md :

```
# MySQL + phpMyAdmin (compose)

mdp dans le fichier .env (MYSQL_ROOT_PASSWORD et MYSQL_PASSWORD), pas dans le readme
base lab, user student

cd C:\Users\adamalmounayar\docker-tp\compose-mysql
docker compose config
docker compose up -d
docker compose ps

ouvrir http://localhost:3000, login student + MYSQL_PASSWORD
pma se connecte avec PMA_HOST=mysql sur le reseau myNet, seul pma a un port publie
depends_on tout seul attend pas que mysql soit pret donc healthcheck + condition service_healthy

arreter : docker compose down
donnees dans le volume mysql_data, pas de down -v
la base et le user sont crees que la premiere fois (volume vide), changer le .env apres change rien
```

## Projet final - Option B (Energy and Carbon Dashboard)

dossier : Option-B-student
frontend/Dockerfile :

```dockerfile
FROM nginx:stable-alpine
COPY index.html app.js style.css /usr/share/nginx/html/
COPY default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

frontend/default.conf :

```nginx
server {
    listen 80;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://api:5000;
    }
}
```

pas de / a la fin du proxy_pass donc nginx garde le chemin /api/... pour flask

backend/Dockerfile :

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["flask", "--app", "app", "run", "--host=0.0.0.0", "--port=5000"]
```

compose.yaml :

```yaml
services:
  web:
    build: ./frontend
    ports:
      - "127.0.0.1:8080:80"
    networks:
      - web_net
    depends_on:
      - api

  api:
    build: ./backend
    environment:
      DB_HOST: mysql
      DB_NAME: energy_lab
      DB_USER: energy_user
      DB_PASSWORD_FILE: /run/secrets/mysql_user_password
    secrets:
      - mysql_user_password
    networks:
      - web_net
      - data_net
    depends_on:
      mysql:
        condition: service_healthy

  mysql:
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: energy_lab
      MYSQL_USER: energy_user
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_root_password
      MYSQL_PASSWORD_FILE: /run/secrets/mysql_user_password
    secrets:
      - mysql_root_password
      - mysql_user_password
    volumes:
      - mysql_data:/var/lib/mysql
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - data_net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1"]
      interval: 5s
      timeout: 5s
      retries: 20
      start_period: 30s

networks:
  web_net:
  data_net:
    internal: true

volumes:
  mysql_data:

secrets:
  mysql_root_password:
    file: ./secrets/mysql-root-password.txt
  mysql_user_password:
    file: ./secrets/mysql-user-password.txt
```

commandes :

```
cp secrets/mysql-root-password.example secrets/mysql-root-password.txt
cp secrets/mysql-user-password.example secrets/mysql-user-password.txt
docker compose config
docker compose up -d --build
docker compose ps
```

dashboard : http://localhost:8080
api : http://localhost:8080/api/measurements

seul web a un port publie (127.0.0.1:8080), api et mysql non
