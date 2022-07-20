FROM php:7.4-fpm-alpine

LABEL maintainer="Jonathan Eom <jeom@unleashed-technologies.com>"

RUN apk add --no-cache \
    coreutils \
    gettext \
    make \
    curl

# Enable xdebug if $PHP_XDEBUG_MODE is not empty
ARG PHP_XDEBUG_MODE=off
ARG PHP_XDEBUG_CLIENT_PORT=5902
ARG PHP_XDEBUG_CLIENT_HOST=host.docker.internal
COPY xdebug.ini  "$PHP_INI_DIR/xdebug.ini.template"
RUN if [[ "$PHP_XDEBUG_MODE" != "" ]]; then \
        apk add --no-cache php7-pecl-xdebug; \
        export PHP_XDEBUG_MODE=$PHP_XDEBUG_MODE; \
        export PHP_XDEBUG_CLIENT_PORT=$PHP_XDEBUG_CLIENT_PORT; \
        export PHP_XDEBUG_CLIENT_HOST=$PHP_XDEBUG_CLIENT_HOST; \
        envsubst < "$PHP_INI_DIR/xdebug.ini.template" > "$PHP_INI_DIR/xdebug.ini"; \
    fi
RUN rm -f "$PHP_INI_DIR/xdebug.ini.template"

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions gd intl zip pdo_mysql

RUN curl -sS https://getcomposer.org/installer | tee composer-setup.php \
    && php8 composer-setup.php && rm composer-setup.php* \
    && chmod +x composer.phar && mv composer.phar /usr/bin/composer \
    && ln -s /usr/bin/php8 /usr/local/bin/php

ARG SYMFONY_CLI_VERSION
RUN wget https://github.com/symfony/cli/releases/download/v$SYMFONY_CLI_VERSION/symfony_linux_amd64.gz \
    && gzip -d symfony_linux_amd64.gz \
    && mv symfony_linux_amd64 symfony \
    && chmod +x symfony \
    && mv symfony /usr/local/bin/

RUN apk add --update nodejs npm \
    && npm install --global yarn

COPY symfony.ini "$PHP_INI_DIR/"
COPY symfony.ini /usr/local/etc/php/cli/conf.d/
COPY symfony.pool.conf /usr/local/etc/php-fpm.d/

WORKDIR /var/www/symfony

CMD ["php-fpm", "-F"]

EXPOSE 9001
