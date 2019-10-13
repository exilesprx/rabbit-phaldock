#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/phaldock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/phaldock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG PHP_VERSION

FROM laradock/workspace:2.2-${PHP_VERSION}

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

ARG PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

###########################################################################
# phaldock non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# always run apt update when start and after add new source list, then clean up at end.
RUN apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} phaldock && \
    useradd -u ${PUID} -g phaldock -m phaldock -G docker_env && \
    usermod -p "*" phaldock

#
#--------------------------------------------------------------------------
# Mandatory Software's Installation
#--------------------------------------------------------------------------
#
# Mandatory Software's such as ("php-cli", "git", "vim", ....) are
# installed on the base image 'phaldock/workspace' image. If you want
# to add more Software's or remove existing one, you need to edit the
# base image (https://github.com/phaldock/workspace).
#

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/phaldock/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/phaldock/aliases.sh && \
    chown phaldock:phaldock /home/phaldock/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

USER phaldock

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/phaldock/.composer/composer.json

# Make sure that ~/.composer belongs to phaldock
RUN chown -R phaldock:phaldock /home/phaldock/.composer

USER phaldock

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
;fi

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
;fi

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Non-root user : PHPUnit path
###########################################################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER phaldock

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# xDebug:
###########################################################################

USER root

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y php${PHP_VERSION}-xdebug && \
    sed -i 's/^;//g' /etc/php/${PHP_VERSION}/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc \
;fi

# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

RUN curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.deb.sh | bash

RUN apt-get update

RUN apt-get install -y php${PHP_VERSION}-phalcon && \
    git clone git://github.com/phalcon/phalcon-devtools.git && \
    cd phalcon-devtools/ && \
    ./phalcon.sh

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www
