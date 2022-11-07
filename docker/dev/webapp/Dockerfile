FROM node:18.4-alpine as node
FROM nginx:1.18-alpine as nginx
FROM php:8.0-fpm-alpine
# SHELL ["/bin/ash", "-oeux", "pipefail", "-c"]

# timezone environment
ENV TZ=UTC \
  # locale
  LANG=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LC_ALL=en_US.UTF-8 \
  # composer environment
  COMPOSER_ALLOW_SUPERUSER=1 \
  COMPOSER_HOME=/composer

COPY --from=composer:2.0 /usr/bin/composer /usr/bin/composer

RUN apk update && apk upgrade
RUN apk add --no-cache \
    bash \
    git \
    icu-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libzip-dev \
    libpng-dev \
    zip \
    unzip \
    gcc \
    g++ \
    make \
    supervisor \
    autoconf
RUN mkdir /var/run/php-fpm
RUN mkdir /var/log/nginx /var/cache/nginx
RUN docker-php-ext-install intl pdo_mysql zip bcmath
RUN docker-php-ext-configure gd \
    --with-freetype=/usr/include/ \
    --with-jpeg=/usr/include/ && \
    NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
    docker-php-ext-install -j${NPROC} gd
RUN composer config -g process-timeout 3600
RUN composer config -g repos.packagist composer https://packagist.org

# phpredis
RUN pecl install redis
RUN echo 'extension=redis.so' > /usr/local/etc/php/conf.d/docker-php-ext-redis.ini

COPY --from=node /usr/local/bin /usr/local/bin
COPY --from=node /usr/local/lib /usr/local/lib
COPY --from=node /opt /opt
COPY --from=nginx /etc/nginx /etc/nginx
COPY --from=nginx /usr/lib /usr/lib
COPY --from=nginx /usr/sbin /usr/sbin

WORKDIR /var/www/app

COPY ./docker/dev/webapp/php/php-fpm.d/zzz-www.conf /usr/local/etc/php-fpm.d/zzz-www.conf
COPY ./docker/dev/webapp/php/php.ini /etc/php.ini
COPY ./docker/dev/webapp/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./docker/dev/webapp/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY ./docker/dev/webapp/supervisor/app.ini /etc/supervisor.d/app.ini


EXPOSE 3001 80
CMD ["/usr/bin/supervisord","-n"]