# Distro: Debian 13.x trixie (https://hub.docker.com/_/php)
# Source: https://github.com/docker-library/php/blob/master/8.4/trixie/apache/Dockerfile
FROM php:8.4.22-apache-trixie

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

WORKDIR /var/www/html
EXPOSE 80
