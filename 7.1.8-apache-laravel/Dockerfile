FROM php:7.1.8-apache
LABEL maintainer="contact@jorgemurta.com"

# APP Directory
RUN mkdir /srv/app
WORKDIR /srv/app

# Install Dependencies
RUN apt-get update && apt-get install -y \
        libmcrypt-dev \
        git \
        zlib1g-dev \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*

# Install Laravel PHP Dependencies
RUN docker-php-ext-install \
        mcrypt \
        mbstring \
        tokenizer \
        zip

# Enable Apache Modules
RUN a2enmod rewrite

# Copy Apache Configuration
ADD ./apache.conf /etc/apache2/sites-enabled/000-default.conf

# Install Composer
RUN curl --silent --show-error --fail --location \
      --header "Accept: application/tar+gzip, application/x-gzip, application/octet-stream" \
      "https://getcomposer.org/installer" \
    | php -- --install-dir=/usr/bin --filename=composer --with-zlib

# Copy Composer require files
ONBUILD COPY composer.json composer.lock artisan /srv/app/
ONBUILD COPY database /srv/app/database/

# Install Laravel Packages
ONBUILD RUN composer install --prefer-dist --optimize-autoloader --no-scripts --ignore-platform-reqs

# Copy Application
ONBUILD COPY . /srv/app

# Optimize Laravel
ONBUILD RUN php artisan clear-compiled && \
php artisan optimize && \
php artisan config:clear

# Change Bootstrap Cache Directory Permissions
ONBUILD RUN chgrp -R www-data storage /srv/app/bootstrap/cache && \
chmod -R ug+rwx storage /srv/app/bootstrap/cache

# Change Storage Directory Permissions
ONBUILD RUN chgrp -R www-data storage /srv/app/storage && \
chmod -R ug+rwx storage /srv/app/storage
