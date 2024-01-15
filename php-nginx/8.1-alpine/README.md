# Imagem Docker Alpine com PHP, PHP-FPM e Nginx

* Alpine v3.15
* PHP v8.1.2
* Nginx v1.20.2

Este repositório contém o Dockerfile e os arquivos necessários para a construção da imagem. O processo que a imagem deve rodar inicialmente é um script "run" em `/usr/libexec/s2i/`.


## Utilização

**Download da imagem:**

```sh
docker pull iurigustavo/php-nginx:8.1-alpine
```

**Exemplo de utilização em ambiente local com docker-compose:**

 *docker-compose.yaml*

```yaml
version: "3.7"
services:
  app-name:
    container_name: app-name
    image: iurigustavo/php-nginx:8.1-alpine
    volumes:
      - "./:/opt/app-root/src"
    ports:
      - "8080:8080"
    environment:
      ENV_ALIAS: dev
      ASSEMBLE: docker/assemble
      DOCUMENTROOT: /opt/app-root/src/public
      XDEBUG_ENABLE: 0
      XDEBUG_CLIENT_HOST: <ip-da-máquina-host>
```


## Customização através do `bin/run`

O arquivo `bin/run` pode ser modificado para customizar diversas configurações do NGINX, PHP e PHP-FPM, além de diversas extensões do PHP como OPcache, Memcached, Xdebug, entre outras.

É aconselhado realizar a alteração dos valores utilizados através da declaração das variáveis no arquivo `run`, dessa forma o projeto sempre será executado utilizando os mesmos valores, ao invés de utilizar variáveis de ambiente. Salvo as variáveis que precisam ser declaradas no `docker-compose.yaml` para ativação de alguma extensão, ex.: `MEMCACHED_ENABLE`.

```shell
export PHP_DEFAULT_SOCKET_TIMEOUT=${PHP_DEFAULT_SOCKET_TIMEOUT:-60}
export PHP_DISPLAY_ERRORS=${PHP_DISPLAY_ERRORS:-Off}
export PHP_DISPLAY_STARTUP_ERRORS=${PHP_DISPLAY_STARTUP_ERRORS:-On}
export PHP_ERROR_REPORTING=${PHP_ERROR_REPORTING:-E_ALL & ~E_DEPRECATED & ~E_STRICT}
export PHP_FILE_UPLOADS=${PHP_FILE_UPLOADS:-On}
export PHP_HTML_ERRORS=${PHP_HTML_ERRORS:-On}
export PHP_IGNORE_REPEATED_ERRORS=${PHP_IGNORE_REPEATED_ERRORS:-Off}
export PHP_IGNORE_REPEATED_SOURCE=${PHP_IGNORE_REPEATED_SOURCE:-Off}
export PHP_LOG_ERRORS=${PHP_LOG_ERRORS:-On}
export PHP_MAX_FILE_UPLOADS=${PHP_MAX_FILE_UPLOADS:-20}
export PHP_MAX_INPUT_TIME=${PHP_MAX_INPUT_TIME:-60}
export PHP_MAX_INPUT_VARS=${PHP_MAX_INPUT_VARS:-2000}
export PHP_MEMORY_LIMIT=${PHP_MEMORY_LIMIT:-128M}
export PHP_POST_MAX_SIZE=${PHP_POST_MAX_SIZE:-8M}
export PHP_SHORT_OPEN_TAG=${PHP_SHORT_OPEN_TAG:-On}
export PHP_TIMEZONE=${PHP_TIMEZONE:-America/Sao_Paulo}
export PHP_UPLOAD_MAX_FILESIZE=${PHP_UPLOAD_MAX_FILESIZE:-10M}
export PHP_VARIABLES_ORDER=${PHP_VARIABLES_ORDER:-EGPCS}
```

Nginx server settings:

```shell
export DOCUMENTROOT=${DOCUMENTROOT:-/opt/app-root/src/}
export NGINX_ASSETS_EXPIRATION=${NGINX_ASSETS_EXPIRATION:-1d}
export NGINX_ASSETS_LOG=${NGINX_ASSETS_LOG:-off}
export NGINX_ASSETS_NOT_FOUND_LOG=${NGINX_ASSETS_NOT_FOUND_LOG:-off}
```

Nginx http settings:

```shell
export NGINX_WORKER_PROCESSES=${NGINX_WORKER_PROCESSES:-auto}
export NGINX_WORKER_CPU_AFFINITY=${NGINX_WORKER_CPU_AFFINITY:-auto}
export NGINX_WORKER_CONNECTIONS=${NGINX_WORKER_CONNECTIONS:-4096}
export NGINX_SENDFILE=${NGINX_SENDFILE:-on}
export NGINX_SENDFILE_MAX_CHUNK=${NGINX_SENDFILE_MAX_CHUNK:-512k}
export NGINX_AIO=${NGINX_AIO:-threads}
export NGINX_DIRECTIO=${NGINX_DIRECTIO:-100m}
export NGINX_CLIENT_MAX_BODY_SIZE=${NGINX_CLIENT_MAX_BODY_SIZE:-1000m}
export NGINX_KEEPALIVE_TIMEOUT=${NGINX_KEEPALIVE_TIMEOUT:-75s}
export NGINX_CLIENT_BODY_TIMEOUT=${NGINX_CLIENT_BODY_TIMEOUT:-10s}
export NGINX_CLIENT_HEADER_TIMEOUT=${NGINX_CLIENT_HEADER_TIMEOUT:-10s}
export NGINX_SEND_TIMEOUT=${NGINX_SEND_TIMEOUT:-10s}
export NGINX_GZIP=${NGINX_GZIP:-on}
export NGINX_GZIP_STATIC=${NGINX_GZIP_STATIC:-on}
export NGINX_GZIP_MIN_LENGTH_BYTES=${NGINX_GZIP_MIN_LENGTH_BYTES:-1000}
```

PHP-fpm settings:

```shell
export FPM_MODE=${FPM_MODE:-dynamic}
export FPM_MAX_CHILDREN=${FPM_MAX_CHILDREN:-15}
export FPM_START_SERVERS=${FPM_START_SERVERS:-4}
export FPM_MIN_SPARE_SERVERS=${FPM_MIN_SPARE_SERVERS:-4}
export FPM_MAX_SPARE_SERVERS=${FPM_MAX_SPARE_SERVERS:-11}
export FPM_MAX_REQUESTS=${FPM_MAX_REQUESTS:-4000}
export FPM_PROCESS_IDLE_TIMEOUT=${FPM_PROCESS_IDLE_TIMEOUT:-10s}
export PHP_MAX_EXECUTION_TIME=${PHP_MAX_EXECUTION_TIME:-30}
```

OPcache settings:

```shell
export OPCACHE_ENABLE=${OPCACHE_ENABLE:-0}
if [ "${OPCACHE_ENABLE}" == 1 ]; then
    export OPCACHE_ENABLE_CLI=${OPCACHE_ENABLE_CLI:-1}
    export OPCACHE_PRELOAD_USER=${OPCACHE_PRELOAD_USER:-}
    export OPCACHE_MEMORY_CONSUMPTION=${MEMORY_LIMIT_IN_MB:-128}
    export OPCACHE_REVALIDATE_FREQ=${OPCACHE_REVALIDATE_FREQ:-2}
    export OPCACHE_MAX_FILES=${OPCACHE_MAX_FILES:-4000}
    export OPCACHE_INTERNED_STRINGS_BUFFER=${OPCACHE_INTERNED_STRINGS_BUFFER:-8}

    ## OPcache JIT settings (JIT will only work if opcache is enabled)
    export OPCACHE_JIT=${OPCACHE_JIT:-1254}
    export OPCACHE_JIT_BUFFER=${OPCACHE_JIT_BUFFER:-100M}
    export OPCACHE_JIT_DEBUG=${OPCACHE_JIT_DEBUG:-0}
    envsubst < /opt/app-root/etc/php.d/opcache.ini.template > /etc/php/conf.d/opcache.ini
fi
```

Memcached:

```shell
export MEMCACHED_ENABLE=${MEMCACHED_ENABLE:-0}
```

Xdebug v3 settings:

```shell
export XDEBUG_ENABLE=${XDEBUG_ENABLE:-0}
if [ "${XDEBUG_ENABLE}" == 1 ]; then
    export XDEBUG_EXTENSION=${XDEBUG_EXTENSION:-/usr/local/lib/php/extensions/no-debug-non-zts-20210902/xdebug.so}
    export XDEBUG_START_WITH_REQUEST=${XDEBUG_START_WITH_REQUEST:-yes}
    export XDEBUG_CLIENT_PORT=${XDEBUG_CLIENT_PORT:-9003}
    export XDEBUG_CLIENT_HOST=${XDEBUG_CLIENT_HOST:-127.0.0.1}
    export XDEBUG_DISCOVER_CLIENT_HOST=${XDEBUG_DISCOVER_CLIENT_HOST:-true}
    export XDEBUG_MODE=${XDEBUG_MODE:-debug}
fi
```

New Relic settings:

```shell
if [ -n "${NEW_RELIC_LICENSE_KEY}" ] && [ -n "${NEW_RELIC_APPNAME}" ]; then
    export NEW_RELIC_LICENSE_KEY=${NEW_RELIC_LICENSE_KEY}
    export NEW_RELIC_APPNAME=${NEW_RELIC_APPNAME}
    envsubst < /opt/app-root/etc/php.d/newrelic.ini.template > /etc/php/conf.d/newrelic.ini
    newrelic-daemon
fi
```
