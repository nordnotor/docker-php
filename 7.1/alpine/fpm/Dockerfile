FROM php:7.1-fpm-alpine

# Arg
ARG PUID=1000
ARG PGID=1000
ARG ROOTFS_DIR="./rootfs"

# Container MODE - dev|prod
ENV MODE "prod"

# Php ini
ENV PHP_DATE_TIME_ZONE "Europe/Kiev"
ENV PHP_MEMORY_LIMIT "2G"
ENV PHP_POST_MAX_SIZE "50M"
ENV PHP_MAX_INPUT_TIME 60
ENV PHP_MAX_EXECUTION_TIME 300
ENV PHP_UPLOAD_MAX_FILESIZE "50M"
ENV PHP_FILE_UPLOADS "on"
ENV PHP_SESSION_SAVE_HANDLER "files"
ENV PHP_SESSION_SAVE_PATH ""
ENV PHP_SENDMAIL_PATH ""
ENV PHP_FPM_LISTEN "127.0.0.1:9000"

# SSMTP
ENV SSMTP_ROOT "yourmail@mail.com"
ENV SSMTP_MAILHUB "mailhog:1025"
ENV SSMTP_HOSTNAME "yourserver.example.com"
ENV SSMTP_FROM_LINE_OVERRIDE "YES"

# Add repositories
RUN echo http://dl-2.alpinelinux.org/alpine/edge/community/ >> /etc/apk/repositories

# Install packages
RUN apk add --update --no-cache \
    icu-dev \
    libxslt-dev \
    libpng-dev \
    libmcrypt-dev \
    libjpeg-turbo \
    libjpeg-turbo-dev \
    freetype \
    freetype-dev \
    imagemagick-dev \
    imagemagick \
    libtool \
    git \
    mc \
    curl \
    unzip \
    mysql-client \
    ca-certificates \
    openssl \
    openssh \
    pcre-dev \
    autoconf \
	file \
	g++ \
	gcc \
	libc-dev \
	make \
	pkgconf \
	re2c \
	curl-dev \
	openssl-dev \
	dcron \
	wget \
	rsync \
	shadow \
    ssmtp

# Install gd
RUN docker-php-ext-configure gd \
    --with-gd \
    --with-freetype-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    && NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
    && docker-php-ext-install -j${NPROC} gd \
    && apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev

# Inslall and enable other extensions
RUN docker-php-source extract \
    && pecl install gd mcrypt xdebug redis imagick \
    && docker-php-ext-enable imagick redis xdebug \
    && docker-php-source delete \
    && docker-php-ext-install mcrypt zip mbstring intl xsl exif pdo_mysql soap simplexml opcache \
    && rm -rf /tmp/*

# Copy files
COPY ${ROOTFS_DIR}/command /usr/bin/command
COPY ${ROOTFS_DIR}/php/php-fpm.d /usr/local/etc/php-fpm.d
COPY ${ROOTFS_DIR}/php/php.ini /usr/local/etc/php/php.ini
COPY ${ROOTFS_DIR}/php/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install composer, give access to commands and change PUID, PGID.
RUN usermod -u ${PUID} www-data && groupmod -g ${PGID} www-data \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer \
    && chown -R www-data:www-data /usr/bin/command && chmod -R +x /usr/bin/command/

WORKDIR /var/www/html

EXPOSE 9000

ENTRYPOINT ["/usr/bin/command/entrypoint.sh"]

HEALTHCHECK --interval=10s --timeout=5s --start-period=20s --retries=50 \
        CMD REDIRECT_STATUS=true SCRIPT_NAME=/ping SCRIPT_FILENAME=/ping REQUEST_METHOD=GET cgi-fcgi -bind -connect 127.0.0.1:9000

CMD ["php-fpm", "--nodaemonize"]