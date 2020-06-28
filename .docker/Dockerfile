FROM php:fpm-buster

ENV COMPOSER_ALLOW_SUPERUSER=1

ARG INSTALL_OPCACHE=true
ENV INSTALL_OPCACHE $INSTALL_OPCACHE

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libmemcached-dev \
        libz-dev \
        libpq-dev \
        libssl-dev \
        libmcrypt-dev \
        openssh-server \
        libmagickwand-dev \
        git \
        cron \
        nano \
        libxml2-dev \
        libzip-dev \
        unzip \
        nginx \
        && rm -r /var/lib/apt/lists/*

# Install the PHP extention
RUN docker-php-ext-install pcntl zip bcmath intl

# Install xdebug and copy config file
RUN set -eux \
    && if [ ${INSTALL_OPCACHE} = true ]; then \
        docker-php-ext-install opcache; \
    fi

COPY ./.docker/opcache.ini /usr/local/etc/php/conf.d/opcache.ini

# Install composer and add its bin to the PATH.
RUN curl -s http://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer
# Source the bash
RUN . ~/.bashrc

#install parallel composer install
RUN composer global require hirak/prestissimo

COPY . /var/www/html

WORKDIR /var/www/html

RUN rm /etc/nginx/sites-enabled/default

COPY ./.docker/deploy.conf /etc/nginx/conf.d/default.conf

RUN mv /usr/local/etc/php-fpm.d/www.conf /usr/local/etc/php-fpm.d/www.conf.backup

COPY ./.docker/www.conf /usr/local/etc/php-fpm.d/www.conf

RUN usermod -a -G www-data root
RUN chgrp -R www-data storage

RUN chown -R www-data:www-data ./storage
RUN chmod -R 0777 ./storage

RUN chmod +x .docker/run

ENTRYPOINT ["sh", ".docker/run"]

EXPOSE 80
