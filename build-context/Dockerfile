ARG BASE_IMAGE
FROM $BASE_IMAGE

LABEL maintainer="zareldyn" \
      description="A simple HTTP(S) reverse proxy based on Apache and Let's Encrypt."

RUN ["/bin/bash", "-c", " \
    apt-get update && \
    apt-get install -y \
        apache2 \
        cron && \
    apt-get install -t stretch-backports -y \
        python-certbot-apache && \
    a2enmod headers rewrite ssl proxy proxy_http && \
    a2dissite 000-default.conf && \
    sed -i s/www-data/$MAIN_USER_LOGIN/ /etc/apache2/envvars && \
    rm -rf /var/lib/apt/lists/* \
"]

EXPOSE 80 443
