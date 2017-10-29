FROM php:7.1.10-apache
MAINTAINER Nilesh <nilesh@cloudgeni.us>

# Install PHP extensions
RUN apt-get update && apt-get install --no-install-recommends -y \
    cron \
    libc-client-dev \
    libicu-dev \
    libkrb5-dev \
    libmcrypt-dev \
    libssl-dev \
    unzip \
    zip \
    sudo \
    vim \
    git-core \
    curl \
    htop \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    && rm /etc/cron.daily/* \
    && apt-get upgrade -y && apt-get autoremove -y --purge \
    && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-configure imap --with-imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-install imap intl mbstring mcrypt mysqli pdo pdo_mysql zip \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

VOLUME /var/www/html

ENV PHPREDIS_VERSION 3.1.4

RUN mkdir -p /usr/src/php/ext/redis \
    && curl -L https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 \
    && echo 'redis' >> /usr/src/php-available-exts \
    && docker-php-ext-install redis

# Define Mautic version and expected SHA1 signature
ENV MAUTIC_VERSION 2.11.0
ENV MAUTIC_SHA1 7d410f2bb09e06b06e52f0e98eef1ae51a5696d9

ENV MAUTIC_RUN_CRON_JOBS true
ENV MAUTIC_DB_USER root
ENV MAUTIC_DB_NAME mautic

# Download package and extract to web volume
RUN curl -o mautic.zip -SL https://s3.amazonaws.com/mautic/releases/${MAUTIC_VERSION}.zip \
	&& echo "$MAUTIC_SHA1 *mautic.zip" | sha1sum -c - \
	&& mkdir /usr/src/mautic \
	&& unzip mautic.zip -d /usr/src/mautic \
	&& rm mautic.zip \
	&& chown -R www-data:www-data /usr/src/mautic

# Copy init scripts and custom .htaccess
COPY docker-entrypoint.sh /entrypoint.sh
COPY makeconfig.php /makeconfig.php
COPY makedb.php /makedb.php
COPY mautic.crontab /etc/cron.d/mautic
COPY mautic-php.ini /usr/local/etc/php/conf.d/mautic-php.ini
# Enable Apache Rewrite Module
RUN a2enmod rewrite

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
