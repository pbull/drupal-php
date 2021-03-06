FROM alpine:3.4
MAINTAINER Wodby <admin@wodby.com>

# Install packages
RUN echo 'http://alpine.gliderlabs.com/alpine/v3.4/main' > /etc/apk/repositories && \
    echo 'http://alpine.gliderlabs.com/alpine/v3.4/community' >> /etc/apk/repositories && \
    echo 'http://alpine.gliderlabs.com/alpine/edge/testing' >> /etc/apk/repositories && \
    echo 'http://alpine.gliderlabs.com/alpine/edge/community' >> /etc/apk/repositories && \

    apk add --update \
        openssl \
        ca-certificates \
        openssh-client \
        rsync \
        git \
        curl \
        wget \
        gzip \
        patch \
        perl \
        pcre \
        imap \
        imagemagick \
        mariadb-client \

        # Temp packages
        build-base \
        autoconf \
        libtool \
        php5-dev \
        pcre-dev \
        imagemagick-dev \

        # PHP packages
        php5 \
        php5-common \
        php5-cli \
        php5-fpm \
        php5-opcache \
        php5-dom \
        php5-xml \
        php5-xmlreader \
        php5-ctype \
        php5-ftp \
        php5-gd \
        php5-json \
        php5-posix \
        php5-curl \
        php5-pdo \
        php5-pdo_mysql \
        php5-sockets \
        php5-zlib \
        php5-mcrypt \
        php5-pcntl \
        php5-mysql \
        php5-mysqli \
        php5-sqlite3 \
        php5-bz2 \
        php5-pear \
        php5-exif \
        php5-phar \
        php5-openssl \
        php5-posix \
        php5-zip \
        php5-calendar \
        php5-iconv \
        php5-imap \
        php5-soap \
        php5-memcache \
        php5-redis \
        php5-xdebug \
        php5-xsl \
        php5-ldap \
        php5-bcmath \
        php5-memcached \
        && \

    # Install uploadprogress and imagick
    sed -ie 's/-n//g' /usr/bin/pecl && \
    yes | pecl install uploadprogress && \
    yes | pecl install imagick && \
    echo 'extension=imagick.so' > /etc/php5/conf.d/imagick.ini && \
    echo 'extension=uploadprogress.so' > /etc/php5/conf.d/uploadprogress.ini && \
    rm -rf /tmp/pear && \

    # Disable Xdebug
    rm /etc/php5/conf.d/xdebug.ini && \

    # Install composer
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \

    # Install PHPUnit
    curl -sSL https://phar.phpunit.de/phpunit.phar -o phpunit.phar && \
        chmod +x phpunit.phar && \
        mv phpunit.phar /usr/local/bin/phpunit && \

    # Install drush
    php -r "readfile('https://s3.amazonaws.com/files.drush.org/drush.phar');" > /usr/local/bin/drush && \
    chmod +x /usr/local/bin/drush && \

    # Install Drupal Console
    curl https://drupalconsole.com/installer -o /usr/local/bin/drupal && \
    chmod +x /usr/local/bin/drupal && \

    # Cleanup
    apk del --purge \
        *-dev \
        build-base \
        autoconf \
        libtool \
        && \

    rm -rf \
        /usr/include/php \
        /usr/lib/php/build \
        /var/cache/apk/* \
        /tmp/* \
        /root/.composer

# Configure php.ini
RUN sed -i \
        -e "s/^expose_php.*/expose_php = Off/" \
        -e "s/^;date.timezone.*/date.timezone = UTC/" \
        -e "s/^memory_limit.*/memory_limit = -1/" \
        -e "s/^max_execution_time.*/max_execution_time = 300/" \
        -e "s/^post_max_size.*/post_max_size = 512M/" \
        -e "s/^upload_max_filesize.*/upload_max_filesize = 512M/" \
        -e "s/^;always_populate_raw_post_data.*/always_populate_raw_post_data = -1/" \
        -e "s/^error_reporting.*/error_reporting = E_ALL/" \
        -e "s/^display_errors.*/display_errors = On/" \
        -e "s/^display_startup_errors.*/display_startup_errors = On/" \
        -e "s/^track_errors.*/track_errors = On/" \
        -e "s/^mysqlnd.collect_memory_statistics.*/mysqlnd.collect_memory_statistics = On/" \
        /etc/php5/php.ini && \

    echo "error_log = \"/proc/self/fd/2\"" | tee -a /etc/php5/php.ini

# Copy PHP configs
COPY opcache.ini /etc/php5/conf.d/
COPY xdebug.ini /etc/php5/conf.d/
COPY php-fpm.conf /etc/php5/

# Add default drush aliases
RUN mkdir -p /etc/drush/site-aliases
COPY default.aliases.drushrc.php /etc/drush/site-aliases/

# Create user www-data
RUN addgroup -g 82 -S www-data && \
	adduser -u 82 -D -S -G www-data www-data

# Create work dir
RUN mkdir -p /var/www/html && \
    chown -R www-data:www-data /var/www

WORKDIR /var/www/html
VOLUME /var/www/html
EXPOSE 9000

# Init www-data user
USER www-data
RUN composer global require hirak/prestissimo:^0.3 --optimize-autoloader && \
    rm -rf ~/.composer/.cache && \
    drupal init --override

USER root
COPY docker-entrypoint.sh /usr/local/bin/
CMD docker-entrypoint.sh
