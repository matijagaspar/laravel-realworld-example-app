#
# PHP Dependencies
#
FROM composer:1.7 as vendor

COPY database/ database/

COPY composer.json composer.json
COPY composer.lock composer.lock

RUN composer install \
    --ignore-platform-reqs \
    --no-interaction \
    --no-plugins \
    --no-scripts \
    --prefer-dist

#
# Frontend
#
FROM node:8.11 as frontend

RUN mkdir -p /app/public

COPY package.json webpack.mix.js yarn.lock /app/

WORKDIR /app
#install before source copying means yarn install is not run every change on frontend speeding up the build by 60sec
RUN yarn install

COPY resources/assets/ /app/resources/assets/
RUN yarn production


#
# Application
#
FROM php:7.3.3-apache-stretch

RUN apt-get update && apt-get install -y \
        libmcrypt-dev \
        libzip-dev \
        libxml2-dev \
        # deps for pdo_mysql
        mysql-client

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

RUN pecl install mcrypt-1.0.2 zip

RUN docker-php-ext-enable mcrypt

RUN docker-php-ext-install xmlrpc \
                           pdo_mysql

RUN  apt-get purge -y --auto-remove libxml2-dev libzip-dev

# Use the default production configuration
RUN mkdir -p /var/www/html



ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

COPY . /var/www/html
COPY --from=vendor /app/vendor/ /var/www/html/vendor/
COPY --from=frontend /app/public/js/ /var/www/html/public/js/
COPY --from=frontend /app/public/css/ /var/www/html/public/css/
COPY --from=frontend /app/mix-manifest.json /var/www/html/mix-manifest.json

COPY .env.example /var/www/html/.env

RUN chown -R www-data:www-data /var/www/html/ 

RUN a2enmod rewrite

WORKDIR /var/www/html

CMD php artisan migrate && apache2-foreground