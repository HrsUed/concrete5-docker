FROM php:7.3.1-cli-stretch

ARG site_name=hello_world
ARG c5user_pass=concrete5

RUN apt-get update && apt-get install -y --no-install-recommends \
    # To unzip concrete5 assets
    unzip \
    # To localize JP
    gettext \
  && rm -rf /var/lib/apt/lists/*

# Requirements for concrete5
RUN apt-get update && apt-get install -y --no-install-recommends \
    mysql-client \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libzip-dev \
    libpng-dev \
    libxml2-dev \
    libmcrypt-dev \
  && rm -rf /var/lib/apt/lists/*

# PHP Extention required by concrete5
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd
RUN docker-php-ext-install pdo pdo_mysql zip
RUN docker-php-ext-install dom
RUN docker-php-ext-install iconv
RUN docker-php-ext-install fileinfo
RUN docker-php-ext-install mbstring
RUN docker-php-ext-install simplexml

RUN pecl install mcrypt-snapshot
RUN docker-php-ext-enable mcrypt

# Install PHP composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

###################################
# Deploy concrete5 source files
###################################
RUN mkdir -m 755 /public

COPY concrete5-8.4.4.zip /tmp

WORKDIR /tmp
RUN unzip -q concrete5-8.4.4.zip \
    && mv concrete5-8.4.4 /public/${site_name} \
    && rm concrete5-8.4.4.zip

RUN useradd -m -d /home/concrete5 -s /bin/bash concrete5 \
    && echo "concrete5:${c5user_pass}" | chpasswd \
    && chown -R concrete5:concrete5 /public/${site_name} \
    && chmod 755 /public/${site_name} \
    && su - concrete5 \
    && cd /public/${site_name} \
    && chmod -R 777 application/files \
    && chmod -R 755 application/config \
    && chmod -R 755 packages \
    && chmod -R 755 updates

###################################
# Patches
###################################
COPY --chown=concrete5:concrete5 patches/composer.json /public/${site_name}
COPY --chown=concrete5:concrete5 patches/UnitOfWork.php /public/${site_name}/concrete/vendor/doctrine/orm/lib/Doctrine/ORM
COPY --chown=concrete5:concrete5 patches/simple_html_dom.php /public/${site_name}/concrete/vendor/sunra/php-simple-html-dom-parser/Src/Sunra/PhpSimple/simplehtmldom_1_5
COPY --chown=concrete5:concrete5 patches/FileGenerator.php /public/${site_name}/concrete/vendor/zendframework/zend-code/src/Generator/
COPY --chown=concrete5:concrete5 patches/MethodReflection.php /public/${site_name}/concrete/vendor/zendframework/zend-code/src/Reflection/
COPY --chown=concrete5:concrete5 patches/ArrayObject.php /public/${site_name}/concrete/vendor/zendframework/zend-stdlib/src/

WORKDIR /public/${site_name}
RUN chmod 664 composer.json

WORKDIR /public/${site_name}/concrete/vendor/doctrine/orm/lib/Doctrine/ORM
RUN chmod 664 UnitOfWork.php

WORKDIR /public/${site_name}/concrete/vendor/sunra/php-simple-html-dom-parser/Src/Sunra/PhpSimple/simplehtmldom_1_5
RUN chmod 664 simple_html_dom.php

WORKDIR /public/${site_name}/concrete/vendor/zendframework/zend-code/src/Generator/
RUN chmod 664 FileGenerator.php

WORKDIR /public/${site_name}/concrete/vendor/zendframework/zend-code/src/Reflection/
RUN chmod 664 MethodReflection.php

WORKDIR /public/${site_name}/concrete/vendor/zendframework/zend-stdlib/src/
RUN chmod 664 ArrayObject.php

###################################
# Create concrete5 project
###################################
USER concrete5
WORKDIR /public/${site_name}
RUN composer create-project -n concrete5/composer ${site_name}

###################################
# Install concrete5
###################################
ENV DB_SERVER=172.17.0.2
ARG DB_USERNAME=root
ARG DB_PASSWORD=secret
ARG DB_DATABASE=${site_name}
ARG SITE=${site_name}
ARG STARTING_POINT=elemental_full
ARG ADMIN_PASSWORD=concrete5
ARG LANGUAGE=ja_JP
ARG SITE_LOCALE=ja_JP

# Create Database
RUN mysql -h ${DB_SERVER} -u ${DB_USERNAME} --password=${DB_PASSWORD} \
    -e "CREATE DATABASE ${DB_DATABASE};"

RUN ./concrete/bin/concrete5 c5:install \
    --db-server ${DB_SERVER} \
    --db-username ${DB_USERNAME} \
    --db-password ${DB_PASSWORD} \
    --db-database ${DB_DATABASE} \
    --site ${SITE} \
    # --canonical-url=${CANONICAL_URL} \
    # --canonical-ssl-url=${CANONICAL_SSL_URL} \
    --starting-point ${STARTING_POINT} \
    # --admin-email=${ADMIN_EMAIL} \
    --admin-password ${ADMIN_PASSWORD} \
    # --demo-username=${DEMO_USERNAME} \
    # --demo-password=${DEMO_PASSWORD} \
    # --demo-email=${DEMO_EMAIL} \
    --language ${LANGUAGE} \
    --site-locale ${SITE_LOCALE}
    # --config=${CONFIG} \
    # --attach \
    # --force-attach

USER root

# For debugging
RUN apt-get update && apt-get install -y --no-install-recommends \
    iputils-ping \
    net-tools \
    procps \
    gettext \
  && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["docker-php-entrypoint"]
WORKDIR /public/${site_name}
CMD ["php", "-S", "0.0.0.0:8000"]
