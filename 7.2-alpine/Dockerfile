FROM php:7.2-fpm-alpine3.7

RUN apk add --no-cache --update unzip libressl bzip2-dev postgresql-dev libzip-dev \
        libmcrypt-dev gmp-dev icu-dev freetype-dev pcre-dev libxml2-dev libxslt-dev \
        libjpeg-turbo-dev libpng-dev sqlite-dev curl gettext-dev icu-dev zlib-dev 

ARG PHP_EXTENSIONS="soap iconv intl bz2 gmp mysqli xsl zip gd pgsql pdo pdo_pgsql pdo_mysql pdo_sqlite json dom hash bcmath gettext mbstring simplexml"
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) ${PHP_EXTENSIONS} \
    && rm -rf /tmp/* /var/tmp/* /usr/local/etc/php/conf.d/*

ARG PHPREDIS_VERSION=3.1.6
ARG APCU_VERSION=5.1.9
ARG XDEBUG_VERSION=2.6.0beta1
RUN apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS \
   && pecl install apcu-$APCU_VERSION xdebug-$XDEBUG_VERSION \
   && pear install DB MDB2 Mail Mail_Mime Mail_mimeDecode Log  HTTP_Request2 XML_RPC2 Date \
   && mkdir -p  /usr/src/php/ext && cd /usr/src/php/ext \
   && curl -sL https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.zip -o phpredis.zip \
   && unzip phpredis.zip && rm phpredis.zip \
   && docker-php-ext-install -j$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) phpredis-$PHPREDIS_VERSION \
   && apk del .phpize-deps && rm -rf  /tmp/* /var/tmp/* /usr/local/etc/php/conf.d/*

# Install php tools (composer / phpunit)
RUN cd /tmp && \
    curl -sL https://getcomposer.org/composer.phar > composer.phar && \
    chmod +x composer.phar && \
    mv composer.phar /usr/local/bin/composer && \
    curl -sL https://phar.phpunit.de/phpunit.phar > phpunit.phar && \
    chmod +x phpunit.phar && \
    mv phpunit.phar /usr/local/bin/phpunit && \
    rm -rf /tmp/* /var/tmp/*

RUN rm -rf /usr/local/lib/php/doc /usr/local/lib/php/test /usr/local/lib/php/build /usr/local/bin/docker*

FROM oondeo/openresty:alpine3.7


ENV SUMMARY="Nginx and php image with standar modules"	\
    DESCRIPTION="The image use scripts and configurations compatible \
        with redhat openshift."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Nginx" \
      io.openshift.s2i.scripts-url=image:///usr/libexec/s2i \
      io.s2i.scripts-url=image:///usr/libexec/s2i \
      com.redhat.component="nginx" \
      name="oondeo/php-nginx" \
      version="7" \
      release="2" \
      maintainer="OONDEO <info@oondeo.es>"

USER root
RUN apk add --no-cache runit

COPY --from=0 /usr/local/bin/ /usr/local/bin/
COPY --from=0 /usr/local/lib/ /usr/local/lib/

ENV PHP_INI_PATH="/opt/app-root/etc/php/php.ini" \
    PHP_CLI_INI_PATH="/opt/app-root/etc/php/php-cli.ini" \
    PHP_FPM_PATH="/opt/app-root/etc/php/php-fpm.conf" \
    PHP_FPMD_PATH="/opt/app-root/etc/php/php-fpm.d" \
    PHP_LIB_PATH="/opt/app-root/lib/php" 

RUN install-deps /usr/local && ln -s ${PHP_INI_PATH} /usr/local/etc/php/ \
    && { \
			echo '[global]'; \
			echo "include=$PHP_FPMD_PATH/*.conf"; \
    } | tee ${PHP_FPM_PATH} \
    && sed -i '/include_path/d' $PHP_INI_PATH \
    && echo "include_path=\".:$HOME/app/include:$HOME/include:$PHP_LIB_PATH:/usr/local/lib/php:/usr/share/php\"" >> /opt/app-root/etc/php.ini

COPY root/ /

RUN /usr/bin/docker-footer

#This is needed in development
ENV NGINXCONF_FASTCGI_READ_TIMEOUT=3600 \
    NGINX_CACHE=0    

EXPOSE 8080 8081 9000

CMD [ "/usr/bin/runsvdir", "/opt/app-root/etc/service" ]

USER 1001 
