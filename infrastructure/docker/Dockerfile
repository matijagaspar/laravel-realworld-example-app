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
COPY resources/assets/ /app/resources/assets/

WORKDIR /app

RUN yarn install && yarn production


#
# Application
#
FROM php:7.3.3-apache-stretch

#RUN curl -o /usr/bin/docker-php-pecl-install -L https://raw.githubusercontent.com/helderco/docker-php/master/template/bin/docker-php-pecl-install \
    #&& chmod +x /usr/bin/docker-php-pecl-install



RUN apt-get update && apt-get install -y \
        libmcrypt-dev \
        libzip-dev \
        libxml2-dev \
        # deps for pdo_mysql
        mysql-client

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

RUN pecl install mcrypt-1.0.2 zip

RUN docker-php-ext-enable mcrypt

#RUN docker-php-ext-install tokenizer 
#RUN docker-php-ext-install mbstring 
#RUN docker-php-ext-install pdo 
#RUN docker-php-ext-install zip
#RUN docker-php-ext-install xml 

#RUN docker-php-ext-install xmlrpc
#RUN docker-php-ext-install pdo_mysql

RUN docker-php-ext-install \                            
                             tokenizer \
                             mbstring \
                             pdo \
                             xml \
                             xmlrpc \
                             zip \
                             pdo_mysql

#RUN  apt-get purge -y --auto-remove libxml2-dev libzip-dev libmcrypt-dev

# Use the default production configuration


#docker-php-ext-install
COPY . /var/www/html
COPY --from=vendor /app/vendor/ /var/www/html/vendor/
COPY --from=frontend /app/public/js/ /var/www/html/public/js/
COPY --from=frontend /app/public/css/ /var/www/html/public/css/
COPY --from=frontend /app/mix-manifest.json /var/www/html/mix-manifest.json

COPY .env.example /var/www/html/.env

#COPY ./infrastructure/docker/apache/vhost.conf /etc/apache2/sites-available/000-default.conf

RUN chown -R www-data:www-data /var/www/html/ && a2enmod rewrite

RUN service apache2 restart

ENV APACHE_DOCUMENT_ROOT /var/www/html/public

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

CMD ["apache2-foreground"]