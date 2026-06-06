# Distro: Debian 13.x trixie (https://hub.docker.com/_/php)
# Source: https://github.com/docker-library/php/blob/master/8.4/trixie/apache/Dockerfile
FROM php:8.4.22-apache-trixie

# Set environment variables for user and group configuration
ARG APP_USER_NAME=developer 
ARG APP_USER_ID=1000
ARG APP_GROUP_NAME=developer
ARG APP_GROUP_ID=1000

ARG WEB_DOCUMENT_ROOT=/var/www/html/public
ARG WEB_SERVER_ADMIN=webmaster@localhost
ARG WEB_SERVER_NAME=localhost
ARG WEB_SERVER_ALIAS=www.localhost

# !!! ONLY FOR TESTING/DEVELOPMENT !!!

# Install additional packages (git, make, vim)
RUN apt-get update \
    && apt-get install -y --no-install-recommends git make vim

# Set root and add sudo support 
RUN apt-get update \
    && apt-get install -y --no-install-recommends sudo \
    && rm -rf /var/lib/apt/lists/* \
    # Configuration sudoers
    && echo "Defaults env_keep=\"http_proxy https_proxy no_proxy\"" > /etc/sudoers.d/defaults \
    && chmod 0440 /etc/sudoers.d/defaults \
    # Set root password: pass
    && echo "root:pass" | chpasswd

# Set application user and group
RUN groupadd -g $APP_GROUP_ID $APP_GROUP_NAME \
    && useradd -u $APP_USER_ID -g $APP_GROUP_ID -m -s /bin/bash $APP_USER_NAME \
    && echo $APP_USER_NAME ALL=\(root\) NOPASSWD:ALL >/etc/sudoers.d/$APP_USER_NAME \
    && chmod 0440 /etc/sudoers.d/$APP_USER_NAME \
    && sed -i "s|\${APACHE_RUN_USER\:=www-data}|\${APACHE_RUN_USER\:=$APP_USER_NAME}|g" /etc/apache2/envvars \
	&& sed -i "s|\${APACHE_RUN_GROUP\:=www-data}|\${APACHE_RUN_GROUP\:=$APP_GROUP_NAME}|g" /etc/apache2/envvars

# Setup Apache
RUN echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf \
    && a2enconf servername \
    # Enable module(s)
    && a2enmod rewrite \
    # Configure hosts file(s)
    ###############################################################################################
    && a2dissite 000-default \
    && mv /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/default.conf \
    && sed -i -e "s|/var/www/html|$WEB_DOCUMENT_ROOT|g" /etc/apache2/sites-available/default*.conf \
    && sed -i -e "s|ServerAdmin webmaster@localhost|ServerAdmin $WEB_SERVER_ADMIN\n\n\tServerName $WEB_SERVER_NAME\n\tServerAlias $WEB_SERVER_ALIAS|g" /etc/apache2/sites-available/default*.conf \
    # Configure ports
    && sed -i "s|Listen 80|Listen 8080|g" /etc/apache2/ports.conf \
    && sed -i "s|\:80>|\:8080>|g" /etc/apache2/sites-available/default*.conf \
    # Enable site(s)
    && a2ensite default


# https://github.com/php/pie/releases
ARG PIE_VERSION="1.4.0"

# PHP extensions: gd, intl, pdo_mysql, zip
# https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/Administration/Installation/SystemRequirements/Index.html
###################################################################################################
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ghostscript \
        graphicsmagick \
        libavif-dev \
        libfreetype-dev \
        libjpeg62-turbo-dev \
        libicu-dev \
        libpng-dev \
        libwebp-dev \
        libxpm-dev \
        libzip-dev \
    && rm -rf /var/lib/apt/lists/* \
    && curl -L https://github.com/php/pie/releases/download/${PIE_VERSION}/pie.phar -o /usr/local/bin/pie \
    && chmod +x /usr/local/bin/pie \
    && docker-php-ext-configure gd --with-jpeg --with-xpm --with-webp --with-freetype --with-avif \
    && docker-php-ext-install gd \
    && docker-php-ext-install intl \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install zip


# https://getcomposer.org/download/ (Latest LTS)
ARG COMPOSER_VERSION="2.10.1"
# https://packagist.org/packages/xdebug/xdebug
ARG PHP_XDEBUG_VERSION="3.5.1"
# https://github.com/aptible/supercronic/releases
ARG SUPERCRONIC_VERSION="0.2.34"
# Target architecture setup by buildx (linux/amd64, linux/arm64, linux/arm/v7)
ARG TARGETARCH

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        default-mysql-client \
        make \
        mc \
        openssh-client \
        sqlite3 \
        unzip  \
        vim \
        wget \
        zsh \
    && rm -rf /var/lib/apt/lists/* \
    # Update PHP setup
    && pie install xdebug/xdebug:${PHP_XDEBUG_VERSION} \
    # Copy php ini (php.ini-development|php.ini-production)
    && cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini" \
    # Add custom ini parameter
    && printf "[PHP]\nmemory_limit = 512M\nmax_execution_time = 3600\nmax_input_vars = 1500\npcre.jit = 1\nupload_max_filesize = 50M\npost_max_size = 50M\n\n[Date]\ndate.timezone = Europe/Berlin\n" > "$PHP_INI_DIR/conf.d/additional.ini" \
    # Fetch & install composer binary
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --version=${COMPOSER_VERSION} --filename=composer \
    # Supercronic
    && curl -fsSLO "https://github.com/aptible/supercronic/releases/download/v${SUPERCRONIC_VERSION}/supercronic-linux-${TARGETARCH}" \
    && chmod +x "supercronic-linux-${TARGETARCH}" \
    && mv "supercronic-linux-${TARGETARCH}" /usr/local/bin/supercronic

EXPOSE 8080

WORKDIR /var/www/html
