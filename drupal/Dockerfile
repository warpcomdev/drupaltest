FROM php:8.1-apache

RUN apt-get update -q && apt-get install -y wget zip git

# See https://www.drupal.org/docs/7/configuring-clean-urls/enable-clean-urls#dedicated
RUN a2enmod rewrite

WORKDIR /var/www

# See https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md
# https://github.com/composer/getcomposer.org/commits/main
# This hash belongs to conposer release 2.4.2
RUN wget https://raw.githubusercontent.com/composer/getcomposer.org/d366ce4e77501dd11a28baace02e702dd9f1c203/web/installer -O - -q | \
    php -- --install-dir=/usr/local/bin --filename=composer

# See https://hub.docker.com/_/php
RUN apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-install -j$(nproc) opcache

# See https://stackoverflow.com/questions/57183109/how-to-enable-phps-mysql-extensions-in-docker-container
RUN docker-php-ext-install pdo pdo_mysql

# See https://www.drupal.org/docs/develop/using-composer/manage-dependencies#download-core
RUN composer create-project drupal/recommended-project:9.4.8 asicamera && \
    chown -R www-data:www-data asicamera

WORKDIR /var/www/asicamera
USER www-data

# See https://www.drupal.org/docs/comparison-of-contributed-modules/comparison-of-mapping-modules/list-of-d8d9-mapping-modules
RUN composer require --dev drush/drush
RUN composer require 'drupal/geofield:^1.44'
RUN composer require 'drupal/geofield_map:^2.88'
RUN composer require 'drupal/geolocation:^3.10'

# DB must be configured for this to work
# RUN ./vendor/bin/drush en geofield
# RUN ./vendor/bin/drush en geofield_map
# RUN ./vendor/bin/drush en geolocation

USER root
# Copy files last, so a change in the file does not invalidate
# the docker build cache for all the layers above
COPY opcache.ini /usr/local/etc/php/conf.d/opc

# See https://hub.docker.com/_/php
ENV APACHE_DOCUMENT_ROOT /var/www/asicamera/web
RUN sed -ri -e "s!/var/www/html!${APACHE_DOCUMENT_ROOT}!g" /etc/apache2/sites-available/*.conf
RUN sed -ri -e "s!/var/www/!${APACHE_DOCUMENT_ROOT}!g" /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

