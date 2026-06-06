# Distro: Debian 13.x trixie (https://hub.docker.com/_/php)
# Source: https://github.com/docker-library/php/blob/master/8.4/trixie/apache/Dockerfile
FROM php:8.4.22-apache-trixie

# Set environment variables for user and group configuration
ARG APP_USER_NAME=developer 
ARG APP_USER_ID=1000
ARG APP_GROUP_NAME=developer
ARG APP_GROUP_ID=1000

# !!! ONLY FOR TESTING/DEVELOPMENT !!!

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
    && useradd -u $APP_USER_ID -g $APP_GROUP_ID -m -s /bin/bash $APP_USER_NAME
    && echo $APP_USER_NAME ALL=\(root\) NOPASSWD:ALL >/etc/sudoers.d/$APP_USER_NAME
    && chmod 0440 /etc/sudoers.d/$APP_USER_NAME \
    && sed -i "s|\${APACHE_RUN_USER\:=www-data}|\${APACHE_RUN_USER\:=$APP_USER_NAME}|g" /etc/apache2/envvars
	&& sed -i "s|\${APACHE_RUN_GROUP\:=www-data}|\${APACHE_RUN_GROUP\:=$APP_GROUP_NAME}|g" /etc/apache2/envvars

# Set apache to listen on port 8800 instead of 80
RUN sed -i "s|Listen 80|Listen 8800|g" /etc/apache2/ports.conf \
    && sed -i "s|\:80>|\:8800>|g" /etc/apache2/sites-available/default*.conf

WORKDIR /var/www/html
EXPOSE 8800
