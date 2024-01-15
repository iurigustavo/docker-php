# Imagem Docker Alpine com PHP, PHP-FPM e Nginx

## 8.2
* Alpine v3.16
* PHP v8.2.4
* Nginx v1.22.0

## 8.1
* Alpine v3.16
* PHP v8.1.2
* Nginx v1.20.2

## 7.4
* Alpine v3.16
* PHP v7.4.27
* Nginx v1.20.2

Este repositório contém o Dockerfile e os arquivos necessários para a construção da imagem. O processo que a imagem deve rodar inicialmente é um script "run" em `/usr/libexec/s2i/`.


## Utilização

**Download da imagem:**

```sh
docker pull iurigustavo/php-nginx:{versao}-alpine
```

**Exemplo de utilização em ambiente local com docker-compose:**

*docker-compose.yaml*

```yaml
version: "3.7"
services:
  app-name:
    container_name: app-name
    image: iurigustavo/php-nginx:{versao}-alpine
    volumes:
      - "./:/opt/app-root/src"
    ports:
      - "8080:8080"
    environment:
      ENV_ALIAS: dev
      DOCUMENTROOT: /opt/app-root/src/public
      XDEBUG_ENABLE: 0
      XDEBUG_CLIENT_HOST: <ip-da-máquina-host>
```


## Customização através do `bin/run`

O arquivo `bin/run` pode ser modificado para customizar diversas configurações do NGINX, PHP e PHP-FPM, além de diversas extensões do PHP como OPcache, Memcached, Xdebug, entre outras.

É aconselhado realizar a alteração dos valores utilizados através da declaração das variáveis no arquivo `run`, dessa forma o projeto sempre será executado utilizando os mesmos valores, ao invés de utilizar variáveis de ambiente. Salvo as variáveis que precisam ser declaradas no `docker-compose.yaml` para ativação de alguma extensão, ex.: `MEMCACHED_ENABLE`.

A image executará automaticamente: `"${HOME}"/docker/"${ENV_ALIAS}"`

Nginx server settings:

* **DOCUMENTROOT**

* **NGINX_ASSETS_EXPIRATION**

* **NGINX_ASSETS_LOG**

* **NGINX_ASSETS_NOT_FOUND_LOG**

Nginx http settings:

* **NGINX_WORKER_PROCESSES**

* **NGINX_WORKER_CPU_AFFINITY**

* **NGINX_WORKER_CONNECTIONS**

* **NGINX_SENDFILE**

* **NGINX_SENDFILE_MAX_CHUNK**

* **NGINX_AIO**

* **NGINX_DIRECTIO**

* **NGINX_CLIENT_MAX_BODY_SIZE**

* **NGINX_KEEPALIVE_TIMEOUT**

* **NGINX_CLIENT_BODY_TIMEOUT**

* **NGINX_CLIENT_HEADER_TIMEOUT**

* **NGINX_SEND_TIMEOUT**

* **NGINX_GZIP**

* **NGINX_GZIP_STATIC**

* **NGINX_GZIP_MIN_LENGTH_BYTES**

PHP-fpm settings:

* **FPM_MODE**

* **FPM_MAX_CHILDREN**

* **FPM_START_SERVERS**

* **FPM_MIN_SPARE_SERVERS**

* **FPM_MAX_SPARE_SERVERS**

* **FPM_MAX_REQUESTS**

* **FPM_PROCESS_IDLE_TIMEOUT**

OPcache settings:

* **OPCACHE_ENABLE**

* **OPCACHE_MEMORY_CONSUMPTION**

* **OPCACHE_REVALIDATE_FREQ**

* **OPCACHE_MAX_FILES**

* **OPCACHE_INTERNED_STRINGS_BUFFER**

* **OPCACHE_PRELOAD**

* **OPCACHE_PRELOAD_USER**

Memcached:

* **MEMCACHED_ENABLE**

Xdebug v3 settings:

* **XDEBUG_EXTENSION**

* **XDEBUG_START_WITH_REQUEST**

* **XDEBUG_CLIENT_PORT**

* **XDEBUG_CLIENT_HOST**

* **XDEBUG_DISCOVER_CLIENT_HOST**

* **XDEBUG_MODE**

New Relic settings:

* **NEW_RELIC_LICENSE_KEY**

* **NEW_RELIC_APPNAME**