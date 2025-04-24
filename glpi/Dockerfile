FROM ubuntu:20.04

# ARG
ARG PHP_VERSION
ARG REPO_GIT
ARG REPO_TAG

#USER PARAMS
USER root

# SET TIMEZONE TO AMERICA/NEW_YORK
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get install -y tzdata && \
    apt-get install -y nano && \
    ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
    dpkg-reconfigure --frontend noninteractive tzdata

# INSTALL NGINX
RUN apt-get update && apt-get install -y \
    nginx \
    curl \
    iputils-ping \
    software-properties-common \
    patch \
    git \
    gcc \
    g++ \
    zlib1g-dev \
    libxml2-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libicu-dev \
    libonig-dev \
    libssl-dev

WORKDIR /etc/nginx/
RUN mkdir ssl
# INSTALL SSL SELF SIGNED CERTIFICATE FOR DOMAIN TBO
RUN cd ssl \
    && openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt -subj "/C=CO/ST=BOG/L=BOG/O=Test, Inc./OU=IT/CN=localhost" \
    && chmod 600 /etc/nginx/ssl/nginx-selfsigned.*

# INSTALL NVM FOR NODE AND NPM
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
RUN echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc
SHELL ["/bin/bash", "-c"]
RUN source ~/.bashrc 

# INSTALL AND CONFIGURE PHP
RUN add-apt-repository ppa:ondrej/php && apt-get update
RUN apt-get install -y php${PHP_VERSION} php${PHP_VERSION}-fpm php${PHP_VERSION}-ldap php${PHP_VERSION}-bcmath php${PHP_VERSION}-cli php${PHP_VERSION}-mbstring php${PHP_VERSION}-xml php${PHP_VERSION}-opcache php${PHP_VERSION}-intl php${PHP_VERSION}-gd php-pear php${PHP_VERSION}-dev php${PHP_VERSION}-mysqli php${PHP_VERSION}-curl

# INSTALL AND CONFIGURE COMPOSER
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && ln -s /usr/local/bin/composer /usr/local/bin/composer.phar \
    && composer self-update \
    && composer --version

# INSTALL ZIP UNZIP
RUN apt-get install -y zip unzip

# CHANGE HOSTS FILE
COPY ./conf/hosts/hosts /etc/hosts

# LOAD CONFIG FOR NGINX
COPY ./conf/nginx/default.conf /etc/nginx/sites-available/default

# LOAD PHP.INI CUSTOM
COPY ./conf/php/php.ini /etc/php/${PHP_VERSION}/cli/php.ini

WORKDIR /usr/share/nginx/html
RUN rm -rf /usr/share/nginx/html/*

# INSTALL GLPi
RUN curl -L -o glpi-10.0.16.tgz https://github.com/glpi-project/glpi/releases/download/10.0.16/glpi-10.0.16.tgz \
    && mkdir glpi \
    && tar -xzf glpi-10.0.16.tgz -C glpi --strip-components=1 \
    && rm glpi-10.0.16.tgz

# CHANGE ICO AND LOGOS
RUN [ -f /usr/share/nginx/html/glpi/pics/logos/logo-GLPI-250-black.png ] && rm /usr/share/nginx/html/glpi/pics/logos/logo-GLPI-250-black.png; curl -o /usr/share/nginx/html/glpi/pics/logos/logo-GLPI-250-black.png https://raw.githubusercontent.com/IT/pictures/main/logo-GLPI-250-black.png
RUN [ -f /usr/share/nginx/html/glpi/pics/favicon.ico ] && rm /usr/share/nginx/html/glpi/pics/favicon.ico; curl -o /usr/share/nginx/html/glpi/pics/favicon.ico https://raw.githubusercontent.com/ITCSPI/pictures/main/favicon.ico
RUN [ -f /usr/share/nginx/html/glpi/pics/logos/logo-GLPI-100-white.png ] && rm /usr/share/nginx/html/glpi/pics/logos/logo-GLPI-100-white.png; curl -o /usr/share/nginx/html/glpi/pics/logos/logo-GLPI-100-white.png https://raw.githubusercontent.com/IT/pictures/main/logo-GLPI-100-white.png

# INSTALL PLUGIN
WORKDIR /usr/share/nginx/html/glpi/plugins

# PLUGIN OAUTHIMAP
RUN curl -L -o oauthimap-main.zip https://github.com/pluginsGLPI/oauthimap/archive/refs/heads/main.zip \
    && unzip oauthimap-main.zip \
    && mv oauthimap-main oauthimap \
    && rm oauthimap-main.zip

# PLUGIN GPLI AZURE SSO
RUN curl -L -o GLPI-Azure-SSO.zip https://github.com/IT/GLPI-Azure-SSO/archive/refs/heads/main.zip \
    && unzip GLPI-Azure-SSO.zip \
    && mv GLPI-Azure-SSO-main singlesignon \
    && rm GLPI-Azure-SSO.zip
    
WORKDIR /usr/share/nginx/html/glpi

# LOAD local_define CUSTOM
COPY ./conf/php/local_define.php /usr/share/nginx/html/glpi/local_define.php

# SETTING PERMISSIONS FOR CONFIGURATION FILES GLPI
RUN chown -R www-data:www-data /usr/share/nginx/html/glpi/config
RUN chmod -R 755 /usr/share/nginx/html/glpi/config

# SETTING PERMISSIONS FOR GLPi LOGS
RUN chown -R www-data:www-data /usr/share/nginx/html/glpi/files
RUN chmod -R 755 /usr/share/nginx/html/glpi/files

# SETTING PERMISSIONS FOR GLPi LOGS
RUN chown -R www-data:www-data /usr/share/nginx/html/glpi/marketplace
RUN chmod -R 755 /usr/share/nginx/html/glpi/marketplace

# RESTART NGINX AND PHP-FPM
RUN systemctl enable nginx && systemctl enable php${PHP_VERSION}-fpm

# START SERVICES
CMD ["bash", "-c", "service nginx start && service php${PHP_VERSION}-fpm start && tail -f /dev/null"]