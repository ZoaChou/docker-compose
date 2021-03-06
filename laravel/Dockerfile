# docker hub:https://hub.docker.com/_/php/
FROM php:7.2-fpm-alpine

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && apk update \
    && apk add tzdata \
        supervisor \
        nginx \
        git \
        $PHPIZE_DEPS \
        libressl \
        libressl-dev \
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" \
    && git clone https://github.com/tideways/php-xhprof-extension.git \
    && cd php-xhprof-extension \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && docker-php-ext-enable tideways_xhprof \
    && pecl install mongodb \
    && docker-php-ext-enable mongodb \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime


COPY ./nginx/conf.d/ /etc/nginx/conf.d/
COPY ./supervisor/supervisor.d/ /etc/supervisor.d/

EXPOSE 80

RUN apk add --no-cache --virtual .build-deps \
        imagemagick-dev \
        libtool \
        postgresql-dev \
    && apk add --no-cache \
        imagemagick \
        postgresql-libs \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install \
        pdo_mysql \
        pdo_pgsql \
        pcntl \
    && curl -s https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer \
    && apk del -f .build-deps \
    && composer config -g repo.packagist composer https://packagist.laravel-china.org \
    && composer global require laravel/laravel -vvv

# Install PHP Extensions (igbinary & memcached)
RUN apk add --no-cache --update libmemcached-libs zlib
RUN set -xe && \
    cd /tmp/ && \
    apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS && \
    apk add --no-cache --update --virtual .memcached-deps zlib-dev libmemcached-dev cyrus-sasl-dev && \
    pecl install igbinary && \
    ( \
        pecl install --nobuild memcached && \
        cd "$(pecl config-get temp_dir)/memcached" && \
        phpize && \
        ./configure --enable-memcached-igbinary && \
        make -j$(nproc) && \
        make install && \
        cd /tmp/ \
    ) && \
    docker-php-ext-enable igbinary memcached && \
    rm -rf /tmp/* && \
    apk del .memcached-deps .phpize-deps

RUN docker-php-ext-install bcmath

VOLUME /www/app
WORKDIR /www/app

CMD ["supervisord","-n","-j","/tmp/supervisord.pid","--user","root","-y","0"]
