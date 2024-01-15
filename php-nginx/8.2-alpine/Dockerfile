FROM iurigustavo/alpine-3.16

# Install dependencies
RUN apk update && \
    apk add --no-cache \
        ca-certificates \
        curl \
        libcurl>=8.0.1-r0 \
        gd \
        gettext \
        git \
        gnu-libiconv \
        icu \
        libpng \
        libressl \
        imagemagick \
        imagemagick-libs \
        imagemagick-dev \
        linux-headers \
        oniguruma-dev \
        openssh-client \
        pcre-dev \
        tar \
        xz \
        zlib \
        unixodbc-dev \
        acl \
        libcrypto3>=3.0.8-r0 \
        libxml2-dev \
        supervisor \
        procps \
        nano

ENV NGINX_VERSION=1.24.0

ENV PHP_VERSION='8.2.8' \
    PHP_INI_DIR=/etc/php \
    PHP_EXTRA_CONFIGURE_ARGS='--enable-fpm --with-fpm-user=default --with-fpm-group=default --disable-cgi' \
    PHP_CFLAGS='-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64' \
    PHP_CPPFLAGS='-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64' \
    PHP_LDFLAGS='-Wl,-O1 -pie' \
    PHP_SHA256='cfe1055fbcd486de7d3312da6146949aae577365808790af6018205567609801' \
    PHP_BINARY='/usr/local/bin/php'

ENV PHP_URL=https://www.php.net/distributions/php-$PHP_VERSION.tar.xz

# Dependencies required for running "phpize"
ENV PHPIZE_DEPS \
        autoconf \
        dpkg-dev \
        dpkg \
        file \
        g++ \
        gcc \
        libc-dev \
        make \
        pkgconf \
        re2c

# Build dependencies
ENV BUILD_DEPENDENCIES \
        argon2-dev \
        coreutils \
        curl-dev \
        gnu-libiconv-dev \
        freetype-dev \
        gettext-dev \
        icu-dev \
        imagemagick \
        libedit-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libpng-dev \
        libpq-dev \
        libsodium-dev \
        libtool \
        libwebp-dev \
        libxslt-dev \
        libzip-dev \
        oniguruma-dev \
        openssl-dev \
        readline-dev \
        pcre-dev \
        sqlite-dev \
        zlib-dev

# Install Packages
RUN set -eux && \
    apk add --no-cache --virtual .build-deps $BUILD_DEPENDENCIES && \
    apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS

# Download and Install Nginx
RUN cd /tmp && \
    curl -Lo nginx-${NGINX_VERSION}.tar.gz http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar xzf nginx-${NGINX_VERSION}.tar.gz && \
    cd /tmp/nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=/etc/nginx \
        --sbin-path=/usr/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/run/nginx.lock \
        --user=default \
        --group=root \
        --with-threads \
        --with-file-aio \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-http_realip_module \
        --with-http_addition_module \
        --with-http_sub_module \
        --with-http_dav_module \
        --with-http_flv_module \
        --with-http_mp4_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_auth_request_module \
        --with-http_random_index_module \
        --with-http_secure_link_module \
        --with-http_slice_module \
        --with-http_stub_status_module \
        --http-log-path=/var/log/nginx/access.log \
        --http-client-body-temp-path=/var/cache/nginx/client_temp \
        --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
        --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
        --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
        --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
        --with-mail \
        --with-mail_ssl_module \
        --with-stream \
        --with-stream_ssl_module \
        --with-stream_realip_module && \
    make -j$(getconf _NPROCESSORS_ONLN) && \
    make install && \
    make clean && \
    mkdir -p /var/cache/nginx

# Download PHP source
RUN set -eux && \
    \
    apk add --no-cache --virtual .fetch-deps gnupg; \
    \
    mkdir -p "$PHP_INI_DIR/conf.d" && \
    \
    mkdir -p /usr/src; \
    cd /usr/src; \
    \
    curl -fsSL -o php.tar.xz "$PHP_URL"; \
    if [ -n "$PHP_SHA256" ]; then \
        echo "$PHP_SHA256 *php.tar.xz" | sha256sum -c -; \
    fi;

# Build and Install PHP
RUN set -eux; \
    export \
        CFLAGS="$PHP_CFLAGS" \
        CPPFLAGS="$PHP_CPPFLAGS" \
        LDFLAGS="$PHP_LDFLAGS" \
    ; \
    mkdir -p /usr/src/php && \
    tar -Jxf /usr/src/php.tar.xz -C /usr/src/php --strip-components=1

RUN cd /usr/src/php; \
    gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
    ./configure \
        --build="$gnuArch" \
        --without-sqlite3 \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        --enable-option-checking=fatal \
        --enable-ftp \
        --enable-mbstring \
        --enable-mysqlnd \
        --enable-bcmath \
        --enable-exif \
        --enable-gd \
        --enable-intl \
        --enable-pcntl \
        --enable-phpdbg \
        --enable-sockets \
        --enable-sysvmsg \
        --enable-sysvsem \
        --enable-sysvshm \
        --disable-phpdbg \
        --with-curl \
        --with-iconv=/usr \
        --with-openssl \
        --with-readline \
        --with-freetype \
        --with-gettext \
        --with-jpeg \
        --with-libedit \
        --with-libxml \
        --with-mhash \
        --with-password-argon2 \
        --with-pdo-mysql \
        --with-pdo-pgsql \
        --with-pear \
        --with-pic \
        --with-pgsql \
        --with-sodium=shared \
        --with-webp \
        --with-xsl \
        --with-zlib \
        --with-zip \
    ${PHP_EXTRA_CONFIGURE_ARGS:-} && \
    make -j$(getconf _NPROCESSORS_ONLN); find -type f -name '*.a' -delete && \
    make install; \
    find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true && \
    make clean; \
    cp -v php.ini-* "$PHP_INI_DIR/" && \
    cd / && \
    rm -Rf /usr/src/php && \
    runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-cache $runDeps; \
    \
    apk del --no-network .build-deps; \
    \
    # update pecl channel definitions https://github.com/docker-library/php/issues/443
    pecl update-channels; \
    rm -rf /tmp/pear ~/.pearrc;

# Newrelic agent
RUN curl -L https://download.newrelic.com/php_agent/archive/10.15.0.4/newrelic-php5-10.15.0.4-linux-musl.tar.gz | tar -C /tmp -zx && \
    cd /tmp/newrelic-php5-* && \
    NR_INSTALL_USE_CP_NOT_LN=1 NR_INSTALL_SILENT=1 ./newrelic-install install && \
    mv /etc/newrelic/newrelic.cfg.template /etc/newrelic/newrelic.cfg && \
    mv /etc/php/conf.d/newrelic.ini /etc/php/conf.d/newrelic.ini.template && \
    rm -f /etc/php/conf.d/newrelic.ini.template && \
    rm -Rf /tmp/newrelic-php5-*

# Extension libs
RUN apk add --no-cache rabbitmq-c-dev yaml-dev

# Install PHP extensions
RUN pecl update-channels && \
    pecl install \
        amqp-1.11.0 \
        apcu-5.1.22 \
        ds-1.4.0 \
        igbinary-3.2.14 \
        mongodb-1.16.1 \
        msgpack-2.2.0 \
        oauth-2.0.7 \
        redis-5.3.7 \
        sqlsrv-5.11.0 \
        pdo_sqlsrv-5.11.0 \
        yaml-2.2.3 \
        imagick \
        xdebug-3.2.2 \
        timezonedb \
        grpc

# Install memcached
RUN export MEMCACHED_DEPS='zlib-dev libmemcached-dev cyrus-sasl -dev' && \
    apk add --no-cache memcached libmemcached-libs && \
    apk add --no-cache --virtual .memcached-deps $MEMCACHED_DEPS && \
    pecl install memcached-3.2.0 && \
    echo "extension=memcached.so" > /etc/php/conf.d/memcached.ini && \
    apk del .memcached-deps

# Enable PHP extensions
RUN echo "extension=amqp.so" > /etc/php/conf.d/amqp.ini && \
    echo "extension=apcu.so" > /etc/php/conf.d/apcu.ini && \
    echo "extension=ds.so" > /etc/php/conf.d/ds.ini && \
    echo "extension=igbinary.so" > /etc/php/conf.d/igbinary.ini && \
    echo "extension=imagick.so" > /etc/php/conf.d/imagick.ini && \
    echo "extension=mongodb.so" > /etc/php/conf.d/mongodb.ini && \
    echo "extension=msgpack.so" > /etc/php/conf.d/msgpack.ini && \
    echo "extension=oauth.so" > /etc/php/conf.d/oauth.ini && \
    echo "extension=redis.so" > /etc/php/conf.d/redis.ini && \
    echo "extension=timezonedb.so" > /etc/php/conf.d/timezonedb.ini && \
    echo "extension=grpc.so" > /etc/php/conf.d/grpc.ini && \
    echo "extension=yaml.so" > /etc/php/conf.d/yaml.ini

# Clean up
RUN apk del .phpize-deps wget && \
    rm -rf /tmp/* ~/.pearrc

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root

## Setup Supervisor
RUN mkdir -p /etc/supervisor/conf.d && \
    chown -R default:root /etc/supervisor/conf.d && \
    cp /opt/app-root/etc/supervisor/supervisord.conf /etc/supervisor/supervisord.conf && \
    mkdir -p /var/log/supervisor

RUN mkdir -p /etc/php-fpm.d/ && \
    mkdir -p /etc/php.d/ && \
    cp /opt/app-root/etc/php.ini.template /etc/php.ini.template && \
    cp /opt/app-root/etc/conf.d/php-fpm/fpm.conf /etc/php-fpm.conf && \
    cp /opt/app-root/etc/php.d/libsodium.ini /etc/php/conf.d/libsodium.ini && \
    cp /opt/app-root/etc/supervisor/php-fpm.conf /etc/supervisor/conf.d/php-fpm.conf && \
    cp /opt/app-root/etc/supervisor/nginx.conf /etc/supervisor/conf.d/nginx.conf

RUN mkdir /tmp/sessions && \
    mkdir /tmp/log && \
    mkdir /tmp/proxy && \
    mkdir /tmp/sessions/opcache && \
    mkdir -p /var/log/supervisor

RUN touch /tmp/log/xdebug.log && \
    touch /var/log/newrelic && \
    touch /var/log/nginx/access.log && \
    touch /var/log/nginx/error.log && \
    chown -R default:root /var/log && \
    chown -R default:root /var/log/* && \
    chown -R default:root /var/cache/nginx &&\
    chown -R default:root /opt/app-root && \
    chown -R default:root /etc/php* && \
    chown -R default:root /etc/newrelic && \
    chown -R default:root /tmp

RUN chmod -R a+rwx /tmp && \
    chmod -R a+rwx /var/log/nginx /var/log/newrelic /var/log/supervisor && \
    chmod -R ug+rwx /opt/app-root && \
    chmod -R ug+rwx /etc/php* && \
    chmod +x /opt/app-root/services.sh

ENV PHP_CFLAGS= \
    PHP_CPPFLAGS= \
    PHP_EXTRA_CONFIGURE_ARGS= \
    PHP_LDFLAGS= \
    PHP_SHA256= \
    PHP_URL= \
    PHPIZE_DEPS= \
    BUILD_DEPENDENCIES=

COPY ./bin/ $STI_SCRIPTS_PATH

STOPSIGNAL SIGQUIT

EXPOSE 8080

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/assemble