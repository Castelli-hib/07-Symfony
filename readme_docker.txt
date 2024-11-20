Étapes pour intégrer Docker Compose dans votre projet Symfony
1. Créer un fichier docker-compose.yml
Ce fichier définit les services nécessaires à votre application. Exemple typique pour un projet Symfony avec PHP, Nginx et MySQL :

yaml
Copier le code
version: '3.8'

services:
  app:
    image: php:8.1-fpm
    container_name: app
    volumes:
      - .:/var/www/html
      - ./php.ini:/usr/local/etc/php/php.ini
    working_dir: /var/www/html
    networks:
      - symfony

  web:
    image: nginx:alpine
    container_name: web
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - symfony

  db:
    image: mysql:8.0
    container_name: db
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: symfony
      MYSQL_USER: symfony
      MYSQL_PASSWORD: symfony
    networks:
      - symfony

networks:
  symfony:
2. Ajouter les fichiers de configuration supplémentaires
Fichier php.ini : Créez un fichier php.ini à la racine du projet pour configurer PHP (par exemple, pour Symfony) :

ini
Copier le code
memory_limit = 512M
upload_max_filesize = 64M
post_max_size = 64M
Fichier nginx.conf : Créez un fichier nginx.conf pour configurer Nginx en tant que serveur web :

nginx
Copier le code
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
3. Lancer les conteneurs Docker Compose
Dans le répertoire racine de votre projet (où se trouve docker-compose.yml), exécutez la commande suivante pour démarrer vos services :

bash
Copier le code
docker-compose up -d
-d lance les conteneurs en arrière-plan.
Une fois lancé, accédez à votre application à l'adresse : http://localhost:8080.
4. Vérifier les logs et l'état des conteneurs
Pour voir les logs :

bash
Copier le code
docker-compose logs -f
Pour vérifier l'état des conteneurs :

bash
Copier le code
docker-compose ps
5. Accéder à votre conteneur PHP (optionnel)
Si vous avez besoin d'accéder au conteneur PHP pour exécuter des commandes Symfony :


Copier le code
docker exec -it app bash
Étapes supplémentaires (si nécessaire)
Ajouter une base de données : Dans Symfony, configurez le fichier .env pour utiliser la base de données MySQL du conteneur :

env
Copier le code
DATABASE_URL=mysql://symfony:symfony@db:3306/symfony
Recréer les conteneurs après modification : Si vous modifiez le fichier docker-compose.yml ou d'autres configurations, recréez les conteneurs avec :


Copier le code
docker-compose up -d --build
Arrêter les conteneurs : Pour arrêter tous les conteneurs :

Copier le code
docker-compose down