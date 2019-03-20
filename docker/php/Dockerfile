ARG PHP_VERSION

FROM php:${PHP_VERSION}-fpm-alpine

RUN apk update \
    && apk upgrade \
    && apk add --no-cache \
        bash \
        git \
        unzip \
        libpng \
        libpng-dev \
        libjpeg \
        icu \
        icu-dev \
        icu-libs \
        libxml2 \
        libxml2-dev \
        openssl \
        openssl-dev \
    && docker-php-ext-install \
        pdo_mysql \
        mysqli \
        gd \
        mbstring \
        intl \
        xml \
        opcache \
    && docker-php-ext-enable intl mbstring \
    && apk --update --no-cache add autoconf g++ make \
    # xdebugインストール
    && pecl install -f xdebug \
    && docker-php-ext-enable xdebug \
    && apk del --purge autoconf g++ make

# Install composer
RUN curl -sS https://getcomposer.org/installer | php ;mv composer.phar /usr/local/bin/composer;
RUN composer global require hirak/prestissimo \
    && composer global require phpunit/phpunit

# Add User
ARG USER_ID
ARG GROUP_ID

RUN deluser www-data && \
    addgroup -g ${GROUP_ID} -S www-data && \
    adduser -u ${USER_ID} -S -G www-data www-data && \
    echo 'www-data:www-data' | chpasswd
USER www-data:www-data

# Setup working directory
ARG APP_CONTAINER
WORKDIR ${APP_CONTAINER}