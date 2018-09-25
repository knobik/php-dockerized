# workspace forked from dev/workspace

ARG PHP_VERSION

FROM knobik/workspace:latest-${PHP_VERSION}

ARG PHP_VERSION

# Start as root
USER root

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# always run apt update when start and after add new source list, then clean up at end.
RUN apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} dev && \
    useradd -u ${PUID} -g dev -m dev -G docker_env && \
    usermod -p "*" dev
	
#
#--------------------------------------------------------------------------
# Mandatory Software's Installation
#--------------------------------------------------------------------------
#
# Mandatory Software's such as ("php-cli", "git", "vim", ....) are
# installed on the base image 'knobik/workspace' image. If you want
# to add more Software's or remove existing one, you need to edit the
# base image (https://github.com/knobik/workspace).
#

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# Composer
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/dev/.composer/composer.json

# Make sure that ~/.composer belongs to dev
RUN chown -R dev:dev /home/dev/.composer

USER dev

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=true
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
USER dev

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/code/vendor/bin:$PATH"' >> ~/.bashrc
	
###########################################################################
# xDebug:
###########################################################################

USER root

ARG INSTALL_XDEBUG=true

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y php${PHP_VERSION}-xdebug && \
    sed -i 's/^;//g' /etc/php/${PHP_VERSION}/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /code/vendor/bin/phpunit'" >> ~/.bashrc \
;fi

###########################################################################
# ssh:
###########################################################################

ARG INSTALL_WORKSPACE_SSH=true

COPY insecure_id_rsa /tmp/id_rsa
COPY insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
    rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
        && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
        && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
        && rm -f /tmp/id_rsa* \
        && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
    && cp -rf /root/.ssh /home/dev \
    && chown -R dev:dev /home/dev/.ssh \
;fi


###########################################################################
# Check PHP version:
###########################################################################

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /code