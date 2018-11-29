FROM php:7.2.12-fpm-alpine3.8

MAINTAINER CodeAnti <1428491590@qq.com>

ENV php_conf /usr/local/etc/php-fpm.conf
ENV fpm_conf /usr/local/etc/php-fpm.d/www.conf
ENV php_vars /usr/local/etc/php/conf.d/docker-vars.ini

# Install bcmath
RUN docker-php-ext-install bcmath

# php-redis
ENV PHPREDIS_VERSION 3.0.0

# create group
RUN addgroup -S www

# create user
RUN adduser -D -S -h /var/cache/www -s /sbin/nologin -G www www



# create dir
RUN rm -Rf /var/www/* && \
	rm -Rf /data/www/* && \
	mkdir -p /data/www/ && \
	mkdir -p /var/www/

# install php & extension
RUN echo @testing http://nl.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories && \
    echo /etc/apk/respositories && \
    apk update && \
    apk add --no-cache bash \
    openssh-client \
    openssl \
    wget \
    supervisor \
    curl \
    libcurl \
    git \
    python \
    python-dev \
    py-pip \
    augeas-dev \
    openssl-dev \
    ca-certificates \
    dialog \
    gcc \
    musl-dev \
    linux-headers \
    libmcrypt-dev \
    libpng-dev \
    icu-dev \
    libpq \
    libxslt-dev \
    libffi-dev \
    freetype-dev \
    sqlite-dev \
    libjpeg-turbo-dev && \
    docker-php-ext-configure gd \
      --with-gd \
      --with-freetype-dir=/usr/include/ \
      --with-png-dir=/usr/include/ \
      --with-jpeg-dir=/usr/include/ && \
    #php redis
    mkdir -p /usr/src/php/ext/redis && \
    curl -L https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 && \
    echo 'redis' >> /usr/src/php-available-exts && \
    #curl iconv session
    pecl install mcrypt-1.0.1 && \
    docker-php-ext-enable mcrypt && \
    docker-php-ext-install pdo_mysql pdo_sqlite mysqli gd exif intl xsl json soap dom zip opcache redis && \
    docker-php-source delete

# install composer
RUN EXPECTED_COMPOSER_SIGNATURE=$(wget -q -O - https://composer.github.io/installer.sig) && \
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php -r "if (hash_file('SHA384', 'composer-setup.php') === '${EXPECTED_COMPOSER_SIGNATURE}') { echo 'Composer.phar Installer verified'; } else { echo 'Composer.phar Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" && \
    php composer-setup.php --install-dir=/usr/bin --filename=composer && \
    php -r "unlink('composer-setup.php');"

# change composer packagist
RUN composer config -g repo.packagist composer https://packagist.laravel-china.org

# composer speed up
RUN composer global require "hirak/prestissimo:^0.3"

# delete extra package
RUN apk del gcc musl-dev linux-headers libffi-dev augeas-dev python-dev

# Fix iconv bug
RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Add supervisord conf
RUN mkdir -p /etc/supervisor/conf.d
ADD supervisor/supervisord.conf /etc/supervisord.conf
ADD supervisor/conf.d/ /etc/supervisor/conf.d/

# Add supervisord log
RUN mkdir -p /var/log/supervisor

# tweak php-fpm config
RUN echo "cgi.fix_pathinfo=0" > ${php_vars} &&\
    echo "upload_max_filesize = 100M"  >> ${php_vars} &&\
    echo "post_max_size = 100M"  >> ${php_vars} &&\
    echo "variables_order = \"EGPCS\""  >> ${php_vars} && \
    echo "memory_limit = 128M"  >> ${php_vars} && \
    sed -i \
        -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" \
        -e "s/pm.max_children = 5/pm.max_children = 10/g" \
        -e "s/pm.start_servers = 2/pm.start_servers = 5/g" \
        -e "s/pm.min_spare_servers = 1/pm.min_spare_servers = 1/g" \
        -e "s/pm.max_spare_servers = 3/pm.max_spare_servers = 9/g" \
        -e "s/;pm.max_requests = 500/pm.max_requests = 200/g" \
        -e "s/user = www/user = www/g" \
        -e "s/group = www/group = www/g" \
        -e "s/;listen.mode = 0660/listen.mode = 0666/g" \
        -e "s/;listen.owner = www/listen.owner = www/g" \
        -e "s/;listen.group = www/listen.group = www/g" \
        -e "s/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm.sock/g" \
        -e "s/^;clear_env = no$/clear_env = no/" \
        ${fpm_conf}

# Modify Timezone
ENV TZ Asia/Shanghai
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add Vim
RUN apk add --no-cache vim && \
    rm -f /usr/bin/vi && \
    cp /usr/bin/vim /usr/bin/vi

# Add Cron
COPY scripts/schedule.sh /
COPY cron/schedule /etc/cron.d/

# Modify Cron Configs
RUN chown www.www /schedule.sh && \
    chmod +x /schedule.sh && \
    crontab /etc/cron.d/schedule && \
    mkdir -p /var/log/cron && \
    touch /var/log/cron/cron.log && \
    touch /var/log/cron.log && \
    touch /tmp/cron.log && \
    chown www.www /tmp/cron.log /var/log/cron/cron.log

# Optimize bash
ADD bash/.bashrc /root/.bashrc
ADD bash/.inputrc /root/.inputrc

# Add Scripts
ADD scripts/start.sh /start.sh

# Add Start Script Priviliages
RUN chmod +x /start.sh

# copy in code
ADD src/ /data/www/

WORKDIR /data/www

EXPOSE 80

CMD ["/start.sh"]