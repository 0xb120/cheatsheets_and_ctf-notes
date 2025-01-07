
# PHP 101
## Print info inside logs

```php
error_log("message", 0);
```

## Enable remote debug in PHP with Xdebug

Guide from *How to setup PHP remote debugging* [^remote-debug-php] and Debugging PHP on Docker with VS Code [^remote-debug-php-docker].

```sh
sudo apt-get install php-xdebug
sudo pecl install xdebug
```

[^remote-debug-php]: [How to setup PHP remote debugging](https://blog.devsense.com/2020/remote-debugging), devsense.com
[^remote-debug-php-docker]: [Debugging PHP on Docker with VS Code](https://blog.devsense.com/2019/debugging-php-on-docker-with-visual-studio-code), devsense.com

If you are not using docker, you can simply enable remote debugging by editing `php.ini`

```ini title:"php.ini"
[Xdebug]
zend_extension=xdebug.so
xdebug.remote_enable=1
xdebug.remote_host=<debugger ip>
xdebug.remote_port=<debugger port>
```

If you are using docker, you can specify some docker environment variables and install xdebug from the Dockerfile:

```Dockerfile title:Dockerfile
FROM php:8.2-apache

RUN apt-get update \
  && apt-get install -y \
      wait-for-it \
      unzip \
      libfreetype6-dev \
      libjpeg62-turbo-dev \
      libpng-dev \
      libzip-dev \
      libcurl3-dev \
	  libwebp-dev

RUN pecl install xdebug

RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
  && docker-php-ext-install -j$(nproc) gd zip mysqli curl \
  && docker-php-ext-enable gd zip mysqli curl \
  && docker-php-ext-enable xdebug

RUN chmod -R 777 /var/www/

RUN a2enmod rewrite
```

```yml title:docker-compose.yml
services:
  opencart:
    build: 
      dockerfile: Dockerfile
    user: root
    ports:
      - "80:80"
      - "9000:9000"
    volumes:
      - ./upload:/var/www/html:rw
    environment:
      XDEBUG_MODE: debug
      XDEBUG_CONFIG: client_host=172.17.0.1 client_port=9003
      ...
```

- **XDEBUG_MODE: debug** configures which Xdebug features should be enabled. For our case we will just set `debug`. For more possible values, click [here](https://xdebug.org/docs/step_debug#mode)

You can verify if XDEBUG is enabled searching the xdebug info inside the `phpinfo()` output. Then you can debug the application using the vscode debugger extension "PHP Debug" and the following launcher:
```json title:"launch-xdebug.json"
{
"name": "Listen for Xdebug",
"type": "php",
"request": "launch",
"port": 9003,
"pathMappings": {
"/var/www/html": "${workspaceFolder}\\data"
}
```

When you make a request to [http://localhost:8080?XDEBUG_SESSION_START=1](http://localhost:8080/?XDEBUG_SESSION_START=1) the execution should be halted. `XDEBUG_SESSION_START` argument instructs Xdebug to start debugging the request. [^xdebug-step-debug]

[^xdebug-step-debug]: [Step debugging](https://xdebug.org/docs/step_debug), xdebug.org