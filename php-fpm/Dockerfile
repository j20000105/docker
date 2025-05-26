FROM php:fpm

RUN apt-get update && apt-get install -y \
        libfreetype-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
	libmagickwand-dev \
    && pecl install redis \
    	&& pecl install imagick \
	&& docker-php-ext-configure gd --with-freetype --with-jpeg \
	&& docker-php-ext-install -j$(nproc) gd zip pcntl pdo pdo_mysql \
	&& docker-php-ext-enable redis \
 	&& docker-php-ext-enable imagick

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer

# disable log
RUN echo "access.log = /dev/null" >> /usr/local/etc/php-fpm.d/www.conf

# max_children
RUN echo 'pm.max_children = 100' >> /usr/local/etc/php-fpm.d/zz-docker.conf && \
    echo 'pm.max_requests = 500' >> /usr/local/etc/php-fpm.d/zz-docker.conf
