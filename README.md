### creo el proyecto clonandome el repositorio de github

    cd /var/wwww/html && git clone https://github.com/laravel/laravel.git api-market && cd api-market
    sudo rm -R .git
    git init
    git add .
    git commit -m "initial commit"

### creo el archivo para levantar las imagenes docker

    touch docker-compose.yml

### defino el archivo docker-compose.yml

    version: "3.8"

    services:
        app:
            build:
                args:
                    user: richard
                    uid: 1000
                context: ./
                dockerfile: Dockerfile
            extra_hosts:
                - "host.docker.internal:host-gateway"
            image: php8-laravel
            container_name: php8-laravel
            restart: unless-stopped
            working_dir: /var/www/${PROJECT_NAME}/
            volumes:
                - ./:/var/www/${PROJECT_NAME}
                - ./docker-compose/php/conf.d/xdebug.ini:/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
                - ./docker-compose/php/conf.d/error_reporting.ini:/usr/local/etc/php/conf.d/error_reporting.ini
            networks:
                - env_php_8

        db:
            image: mysql:8
            command: --default-authentication-plugin=mysql_native_password
            container_name: db
            restart: unless-stopped
            environment:
                MYSQL_DATABASE: ${DB_DATABASE}
                MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
                MYSQL_PASSWORD: ${DB_PASSWORD}
                MYSQL_USER: ${DB_USERNAME}
                SERVICE_TAGS: dev
                SERVICE_NAME: mysql
            ports:
                - 33061:3306
            networks:
                - env_php_8

        db-test:
            image: mysql:8
            container_name: db-test
            restart: unless-stopped
            environment:
                MYSQL_DATABASE: ${DB_DATABASE_TESTING}
                MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
                MYSQL_PASSWORD: ${DB_PASSWORD}
                MYSQL_USER: ${DB_USERNAME}
                SERVICE_TAGS: dev
                SERVICE_NAME: mysql-test
            networks:
                - env_php_8

        nginx:
            image: nginx:alpine
            container_name: nginx
            restart: unless-stopped
            ports:
                - 8000:80
            volumes:
                - ./:/var/www/${PROJECT_NAME}
                - ./docker-compose/nginx:/etc/nginx/conf.d/
            networks:
                - env_php_8

        redis:
            image: redis:6.2.1-buster
            container_name: redis
            restart: unless-stopped
            tty: true
            volumes:
                - ./docker-compose/redis/data:/data
            networks:
                - env_php_8

        mailhog:
            image: mailhog/mailhog:v1.0.1
            container_name: mailhog
            restart: unless-stopped
            ports:
                - 8025:8025
            networks:
                - env_php_8

    networks:
        env_php_8:
            driver: bridge

cp .env.example .env
agregar al .env -> PROJECT_NAME=api-market
ademas editar la conf de base de datos

### edito el archivo .env

    PROJECT_NAME=api-market
    DB_CONNECTION=mysql
    DB_HOST=db
    DB_PORT=3306
    DB_DATABASE=db
    DB_DATABASE_TESTING=db-testing
    DB_USERNAME=ricardo
    DB_PASSWORD=123456

### creo el archivo .env.testing

    touch .env.testing

### defino el .env.testing

    APP_NAME=Laravel
    APP_ENV=testing
    APP_KEY=
    APP_DEBUG=true
    APP_URL=http://localhost

    DB_CONNECTION=mysql
    DB_HOST=db-test
    DB_PORT=3306
    DB_DATABASE=db-testing
    DB_USERNAME=ricardo
    DB_PASSWORD=123456

### creo el archivo que configura la imagen de php en su version 8.0.14

    touch Dockerfile

### defino el Dockerfile

    FROM php:8.0.14-fpm-buster

    ARG user
    ARG uid

    RUN apt-get update && apt-get install -y \
        git \
        curl \
        libpng-dev \
        libonig-dev \
        libxml2-dev \
        zip \
        unzip

    RUN pecl install -f xdebug \
        && docker-php-ext-enable xdebug

    RUN apt-get clean && rm -rf /var/lib/apt/lists/*

    RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

    COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

    RUN useradd -G www-data,root -u $uid -d /home/$user $user
    RUN mkdir -p /home/$user/.composer && \
        chown -R $user:$user /home/$user

    WORKDIR /var/www

    USER $user

### creo las carpetas para los archivos de configuracion

    mkdir -p docker-compose/nginx
    mkdir -p docker-compose/php/conf.d
    mkdir -p docker-compose/redis/data

### creo el archivo de configuracion para nginx

    touch docker-compose/nginx/config.conf

### agrego el archivo de configuracion para nginx

    server {
        listen 80;
        index index.php index.html;
        error_log  /var/log/nginx/error.log;
        access_log /var/log/nginx/access.log;
        root /var/www/api-market/public;
        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass app:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }
        location / {
            try_files $uri $uri/ /index.php?$query_string;
            gzip_static on;
        }
    }

### creo el archivo de configuracion para xdebug

    touch docker-compose/php/conf.d/xdebug.ini

### defino el archivo de configuracion para xdebug

    zend_extension=xdebug

    [xdebug]
    xdebug.mode=develop,debug
    xdebug.client_host=host.docker.internal
    xdebug.start_with_request=yes
    xdebug.log=/tmp/xdebug.log
    xdebug.log_level=7
    xdebug.idekey=VSCODE

### creo el archivo de configuracion para los reportes de errores de php

    touch docker-compose/php/conf.d/error_reporting.ini

### defino el archivo para los errores

    error_reporting=E_ALL

### creo el archivo .dockerignore

    touch .dockerignore

### defino el archivo .dockerignore

    **/.classpath
    **/.dockerignore
    **/.env
    **/.git
    **/.gitignore
    **/.project
    **/.settings
    **/.toolstarget
    **/.vs
    **/.vscode
    **/*.*proj.user
    **/*.dbmdl
    **/*.jfm
    **/bin
    **/charts
    **/docker-compose*
    **/compose*
    **/Dockerfile*
    **/node_modules
    **/npm-debug.log
    **/obj
    **/secrets.dev.yaml
    **/values.dev.yaml
    README.md

### creo la carpeta y el archivo para el debugger php

    mkdir .vscode
    touch .vscode/launch.json

### defino el archivo para el debug

    {
        // Use IntelliSense to learn about possible attributes.
        // Hover to view descriptions of existing attributes.
        // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Listen for XDebug",
                "type": "php",
                "request": "launch",
                "port": 9003,
                "pathMappings": {
                    "/var/www": "${workspaceFolder}"
                },
                "xdebugSettings": {
                    "max_data": 65535,
                    "show_hidden": 1,
                    "max_children": 100,
                    "max_depth": 5
                }
            }
        ]
    }

### arranco el entorno docker

    docker-compose up -d

### instalo las dependencias de composer

    docker-compose exec app composer install

### genero la clave de encriptacion para la aplicacion laravel

    docker-compose exec app php artisan key:generate

### verifico la instalacion de laravel

    http://localhost:8000/
