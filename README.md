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

### FORMATEO DE CODIGO

### agrego un paquete para formatear el codigo php segun reglas definidas

    docker-compose exec app composer require friendsofphp/php-cs-fixer

### creo el archivo con las reglas para el formato del codigo php

    touch .php-cs-fixer.php

### defino las reglas en .php-cs-fixer.php

    <?php

    use PhpCsFixer\Config;
    use PhpCsFixer\Finder;

    $rules = [
        'array_syntax' => ['syntax' => 'short'],
        'binary_operator_spaces' => [
            'default' => 'single_space',
            'operators' => ['=>' => null]
        ],
        'blank_line_after_namespace' => true,
        'blank_line_after_opening_tag' => true,
        'blank_line_before_statement' => [
            'statements' => ['return']
        ],
        'braces' => true,
        'cast_spaces' => true,
        'class_attributes_separation' => [
            'elements' => ['method' => 'one']
        ],
        'class_definition' => true,
        // 'concat_space' => [
        //     'spacing' => 'one'
        // ],
        'declare_equal_normalize' => true,
        'elseif' => true,
        'encoding' => true,
        'full_opening_tag' => true,
        'fully_qualified_strict_types' => true,
        'function_declaration' => true,
        'function_typehint_space' => true,
        'heredoc_to_nowdoc' => true,
        'include' => true,
        'increment_style' => ['style' => 'post'],
        'indentation_type' => true,
        'linebreak_after_opening_tag' => true,
        'line_ending' => true,
        'lowercase_cast' => true,
        'constant_case' => true,
        'lowercase_keywords' => true,
        'lowercase_static_reference' => true,    
        'magic_method_casing' => true,
        'magic_constant_casing' => true,
        'method_argument_space' => true,
        'native_function_casing' => true,
        'no_alias_functions' => true,
        'class_attributes_separation' => [
            'elements' => [
                'trait_import' => 'none'
            ]
        ],
        'no_blank_lines_after_class_opening' => true,
        'no_blank_lines_after_phpdoc' => true,
        'no_closing_tag' => true,
        'no_empty_phpdoc' => true,
        'no_empty_statement' => true,
        'no_leading_import_slash' => true,
        'no_leading_namespace_whitespace' => true,
        'no_mixed_echo_print' => [
            'use' => 'echo'
        ],
        'no_multiline_whitespace_around_double_arrow' => true,
        'multiline_whitespace_before_semicolons' => [
            'strategy' => 'no_multi_line'
        ],
        'no_short_bool_cast' => true,
        'no_singleline_whitespace_before_semicolons' => true,
        'no_spaces_after_function_name' => true,
        'no_spaces_around_offset' => true,
        'no_spaces_inside_parenthesis' => true,
        'no_trailing_comma_in_list_call' => true,
        'no_trailing_comma_in_singleline_array' => true,
        'no_trailing_whitespace' => true,
        'no_trailing_whitespace_in_comment' => true,
        'no_unneeded_control_parentheses' => true,
        'no_unreachable_default_argument_value' => true,
        'no_useless_return' => true,
        'no_whitespace_before_comma_in_array' => true,
        'no_whitespace_in_blank_line' => true,
        'normalize_index_brace' => true,
        'not_operator_with_successor_space' => false,
        'object_operator_without_whitespace' => true,
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'phpdoc_indent' => true,
        'general_phpdoc_tag_rename' => true,
        'phpdoc_inline_tag_normalizer' => true,
        'phpdoc_tag_type' => true,
        'phpdoc_no_access' => true,
        'phpdoc_no_package' => true,
        'phpdoc_no_useless_inheritdoc' => true,
        'phpdoc_scalar' => true,
        'phpdoc_single_line_var_spacing' => true,
        'phpdoc_summary' => true,
        'phpdoc_to_comment' => true,
        'phpdoc_trim' => true,
        'phpdoc_types' => true,
        'phpdoc_var_without_name' => true,
        'psr_autoloading' => true,
        'self_accessor' => true,
        'short_scalar_cast' => true,
        'simplified_null_return' => false,
        'single_blank_line_at_eof' => true,
        'single_blank_line_before_namespace' => true,
        'single_class_element_per_statement' => true,
        'single_import_per_statement' => true,
        'single_line_after_imports' => true,
        'single_line_comment_style' => [
            'comment_types' => ['hash']
        ],
        'single_quote' => true,
        'space_after_semicolon' => true,
        'standardize_not_equals' => true,
        'switch_case_semicolon_to_colon' => true,
        'switch_case_space' => true,
        'ternary_operator_spaces' => true,
        'trailing_comma_in_multiline' => true,
        'trim_array_spaces' => true,
        'unary_operator_spaces' => false,
        'visibility_required' => [
            'elements' => ['method', 'property']
        ],
        'whitespace_after_comma_in_array' => true,
        'no_unused_imports' => false,
    ];

    $finder = Finder::create()
        ->in([
            __DIR__ . '/app',
            __DIR__ . '/config',
            __DIR__ . '/database',
            __DIR__ . '/resources',
            __DIR__ . '/routes',
            __DIR__ . '/tests',
        ])
        ->name('*.php')
        ->notName('*.blade.php')
        ->ignoreDotFiles(true)
        ->ignoreVCS(true);

    $config = new Config();
    return $config->setFinder($finder)
        ->setRules($rules)
        ->setRiskyAllowed(true)
        ->setUsingCache(true);

### arreglar los errores de formato de php y su alias
    docker-compose exec app composer run-script lint
    alias phpfixer-lint="docker-compose exec app composer run-script lint"

### mostrar los errores de formato de php y su alias
    docker-compose exec app composer run-script sniff
    alias phpfixer-sniff="docker-compose exec app composer run-script sniff"