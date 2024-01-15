FROM alpine:3.16.2
LABEL maintainer="Iúri Gustavo <gustavo.iuri@gmail.com>"

ADD root /

ENV STI_SCRIPTS_PATH /usr/libexec/s2i
ENV HOME /opt/app-root/src
ENV PATH /opt/app-root/src/bin:/opt/app-root/bin:$PATH

RUN chmod 777 /tmp
RUN mkdir -p /opt/app-root && chown -R 1001:0 /opt/app-root
RUN mkdir -p /opt/app-root/src && chown -R 1001:0 /opt/app-root/src
RUN adduser -u 1001 -S -G root -h ${HOME} -s /sbin/nologin default
RUN apk update && \
    apk add --no-cache \
        icu-data-full \
        libcrypto3 \
        openssl3 \
        tzdata

ENV TZ America/Sao_Paulo

RUN apk add --update tzdata

RUN chmod 777 /etc/ssl/openssl.cnf

WORKDIR ${HOME}

ENTRYPOINT ["container-entrypoint"]