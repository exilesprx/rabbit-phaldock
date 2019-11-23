#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'php-fpm' base Image, visit its repository on Github
#    https://github.com/Laradock/php-fpm
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/php-fpm/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG PHP_VERSION

FROM phusion/baseimage:latest

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

ARG PHP_VERSION

USER root

# Add the "PHP 7" ppa
RUN apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:ondrej/php

#
#--------------------------------------------------------------------------
# Software's Installation
#--------------------------------------------------------------------------
#

ENV _APT_OPTIONS -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold"

# Install "PHP Extentions", "libraries", "Software's"
RUN apt-get update && \
    apt-get upgrade -y ${_APT_OPTIONS} && \
    apt-get install -y ${_APT_OPTIONS} --allow-downgrades --allow-remove-essential \
        --allow-change-held-packages \
        php${PHP_VERSION}-cli \
        php${PHP_VERSION}-common \
        php${PHP_VERSION}-curl \
        php${PHP_VERSION}-intl \
        php${PHP_VERSION}-json \
        php${PHP_VERSION}-xml \
        php${PHP_VERSION}-mbstring \
        php${PHP_VERSION}-mysql \
        php${PHP_VERSION}-pgsql \
        php${PHP_VERSION}-sqlite \
        php${PHP_VERSION}-sqlite3 \
        php${PHP_VERSION}-zip \
        php${PHP_VERSION}-bcmath \
        php${PHP_VERSION}-memcached \
        php${PHP_VERSION}-gd \
        php${PHP_VERSION}-dev \
        pkg-config \
        libcurl4-openssl-dev \
        libedit-dev \
        libssl-dev \
        libxml2-dev \
        xz-utils \
        libsqlite3-dev \
        sqlite3 \
        git \
        curl \
        vim \
        nano \
        postgresql-client \
    && apt-get clean

RUN apt-get install -y supervisor


###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install mongo && \
      docker-php-ext-enable mongo \
    ;fi && \
    pecl install mongodb && \
    docker-php-ext-enable mongodb \
;fi

#
#--------------------------------------------------------------------------
# Optional Supervisord Configuration
#--------------------------------------------------------------------------
#
# Modify the ./supervisor.conf file to match your App's requirements.
# Make sure you rebuild your container with every change.
#

COPY supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

RUN apt-get install -y php-phalcon

RUN mkdir /run/php && \
    chown www-data:www-data /run/php

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

WORKDIR /etc/supervisor/conf.d/
