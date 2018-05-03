FROM ubuntu:16.04

ENV LANG="en_US.UTF-8" \
    LC_ALL="en_US.UTF-8" \
    LANGUAGE="en_US.UTF-8" \
    TERM="xterm" \
    DEBIAN_FRONTEND="noninteractive" \
    BLACKFIRE_LOG_LEVEL="2" \
    BLACKFIRE_LOG_FILE="stderr" \
    BLACKFIRE_SOCKET="tcp://blackfire:8707"

RUN apt-get update -q && \
    apt-get install -qy software-properties-common language-pack-en-base && \
    export LC_ALL=en_US.UTF-8 && \
    export LANG=en_US.UTF-8 && \
    add-apt-repository ppa:ondrej/php && \
    apt-get update -q && \
    apt-get install --no-install-recommends -qy \
        curl \
        nginx \
        php5.6 \
        php5.6-common \
        php5.6-dom \
        php5.6-fpm \
        php-apcu \
        php-uuid \
        supervisor \
        wget \

    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && cp /usr/share/zoneinfo/Europe/Warsaw /etc/localtime \
    && echo "Europe/Warsaw" > /etc/timezone  \
    && mkdir /run/php

# Blackfire
RUN wget -O - https://packagecloud.io/gpg.key | apt-key add - \
    && echo "deb http://packages.blackfire.io/debian any main" | tee /etc/apt/sources.list.d/blackfire.list \
    && apt-get update \
    && apt-get install blackfire-agent \
    && version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
    && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=${BLACKFIRE_SOCKET}\n" > /etc/php/5.6/cli/conf.d/blackfire.ini \
    && printf "extension=blackfire.so\nblackfire.agent_socket=${BLACKFIRE_SOCKET}\n" > /etc/php/5.6/fpm/conf.d/blackfire.ini

# PHP && nginx config files
COPY pool.conf /etc/php/5.6/fpm/pool.d/www.conf
COPY nginx.conf /etc/nginx/nginx.conf
COPY virtual_hosts.conf /etc/nginx/virtual_hosts.conf

WORKDIR /srv/www

# Supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
CMD ["supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]