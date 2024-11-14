FROM php:8.2-fpm

ARG GIT_EMAIL=${GIT_EMAIL}
ARG GIT_NAME=${GIT_NAME}

WORKDIR "/application"

# Install selected extensions and other stuff
RUN apt-get update \
    && apt-get install -y filter git libicu-dev libonig-dev libsodium-dev \
        libxml2-dev libxslt1-dev libzip-dev openssl unzip zip \
    && apt-get clean

# install php extensions (json and tokenizer already in image base)
RUN docker-php-ext-configure zip -with-zip \
    && docker-php-ext-configure intl
RUN docker-php-ext-install ctype iconv intl mysqli pdo pdo_mysql mbstring session simplexml sodium xml xmlwriter xsl
RUN docker-php-ext-enable intl pdo_mysql

# Install Composer
RUN curl -sS https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer

# Install Symfony CLI
RUN curl -sS https://get.symfony.com/cli/installer | bash \
    && mv /root/.symfony5/bin/symfony /usr/local/bin/symfony

# Install NPM
RUN curl -sL https://deb.nodesource.com/setup_20.x | bash
RUN apt-get install nodejs -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

# Set Paris timezone
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime

# Set git identity to pull Symfony skeleton using CLI
RUN git config --global user.email "${GIT_EMAIL}" \
    && git config --global user.name "${GIT_NAME}"

RUN echo 'alias sf="php bin/console"' >> ~/.bashrc
