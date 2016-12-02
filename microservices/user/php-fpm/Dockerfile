FROM php:7-fpm

RUN apt-get update && apt-get -y install \
        git g++ libcurl4-gnutls-dev libicu-dev libmcrypt-dev libpq-dev libxml2-dev unzip zlib1g-dev \
    && git clone -b php7 https://github.com/phpredis/phpredis.git /usr/src/php/ext/redis \
    && docker-php-ext-install curl intl json mbstring mcrypt pdo pdo_mysql redis xml \
    && apt-get autoremove && apt-get autoclean \
    && rm -rf /var/lib/apt/lists/*


    RUN apt-get update && apt-get upgrade -y && apt-get autoremove -y \
    && apt-get install -y git libmcrypt-dev libpng12-dev libjpeg-dev libpq-dev mysql-client curl \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
    && docker-php-ext-install mcrypt gd mbstring pdo pdo_mysql zip \
    && pecl install xdebug \
    && rm -rf /tmp/pear \
    && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=off\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_port=9000\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.profiler_enable=on\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.profiler_output_dir=/var/www/html/tmp\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.profiler_enable_trigger=on\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer



RUN echo 'date.timezone="Europe/Madrid"' >>  /usr/local/etc/php/conf.d/date.ini
RUN echo 'session.save_path = "/tmp"' >>  /usr/local/etc/php/conf.d/session.ini

ENV CONSUL_TEMPLATE_VERSION 0.16.0
ENV CONSUL_TEMPLATE_SHA1 064b0b492bb7ca3663811d297436a4bbf3226de706d2b76adade7021cd22e156

RUN curl --retry 7 -Lso /tmp/consul-template.zip \
        "https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip" \
    && echo "${CONSUL_TEMPLATE_SHA1}  /tmp/consul-template.zip" | sha256sum -c \
    && unzip /tmp/consul-template.zip -d /usr/local/bin \
    && rm /tmp/consul-template.zip

ENV CONTAINERPILOT_VERSION 2.4.3
ENV CONTAINERPILOT_SHA1 2c469a0e79a7ac801f1c032c2515dd0278134790
ENV CONTAINERPILOT file:///etc/containerpilot.json

RUN curl --retry 7 -Lso /tmp/containerpilot.tar.gz \
         "https://github.com/joyent/containerpilot/releases/download/${CONTAINERPILOT_VERSION}/containerpilot-${CONTAINERPILOT_VERSION}.tar.gz" \
    && echo "${CONTAINERPILOT_SHA1}  /tmp/containerpilot.tar.gz" | sha1sum -c \
    && tar zxf /tmp/containerpilot.tar.gz -C /usr/local/bin \
    && rm /tmp/containerpilot.tar.gz

COPY config/ /etc
COPY scripts/ /usr/local/bin

RUN chmod +x /usr/local/bin/reload.sh

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer

RUN curl -sSL https://phar.phpunit.de/phpunit.phar -o /usr/bin/phpunit && chmod +x /usr/bin/phpunit

ADD ./config/php.ini /usr/local/etc/php/

CMD [ "/usr/local/bin/containerpilot", \
    "php-fpm", \
        "--nodaemonize"]