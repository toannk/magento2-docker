FROM php:7.3-fpm

RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libicu-dev \
    libxml2-dev \
    libxslt-dev \
    libzip-dev \
    git vim unzip cron \
    --no-install-recommends \
    && rm -r /var/lib/apt/lists/*

RUN docker-php-ext-configure gd --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2 \
    && docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-configure intl \
    && docker-php-ext-install -j$(nproc) intl

RUN docker-php-ext-install -j$(nproc) opcache bcmath pdo_mysql soap xsl zip sockets

# Install Xdebug (but don't enable)
RUN pecl install -o xdebug

# Install nvm, node, grunt
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash \
    && . ~/.bashrc \
    && nvm install v12.22.1 \
    && npm i -g grunt-cli

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

# Set magento file permissions
RUN useradd -m -s /bin/bash magento \
    && usermod -a -G www-data magento

RUN cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

ADD ./php-custom.ini $PHP_INI_DIR/conf.d/
# Crontab for user magento
ADD ./magento.cron /var/spool/cron/crontabs/magento
RUN chmod 600 /var/spool/cron/crontabs/magento && chgrp crontab /var/spool/cron/crontabs/magento
# Custom entrypoint to start cron service
ADD custom-entrypoint.sh /usr/local/bin/

ENTRYPOINT [ "/usr/local/bin/custom-entrypoint.sh" ]

CMD ["php-fpm"]

EXPOSE 9000
