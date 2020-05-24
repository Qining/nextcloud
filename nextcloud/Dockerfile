FROM nextcloud:18.0-apache
# FROM nextcloud:fpm

RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        ffmpeg \
        libmagickcore-6.q16-6-extra \
        procps \
        smbclient \
        supervisor \
#       libreoffice \
    ; \
    rm -rf /var/lib/apt/lists/*

RUN set -ex; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libbz2-dev \
        libc-client-dev \
        libkrb5-dev \
        libsmbclient-dev \
    ; \
    \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
    docker-php-ext-install \
        bz2 \
        imap \
    ; \
    pecl install smbclient; \
    docker-php-ext-enable smbclient; \
    \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p \
    /var/log/supervisord \
    /var/run/supervisord \
;

COPY supervisord.conf /

ENV NEXTCLOUD_UPDATE=1

# Install and config aria2
RUN apt-get update && \
    apt-get install -y aria2 wget sudo fatrace && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/log/aria2c && \
    mkdir -p /var/local/aria2c && \
    touch /var/log/aria2c/aria2c.log && \
    touch /var/local/aria2c/aria2c.sess && \
    chown www-data.www-data -R /var/log/aria2c /var/local/aria2c && \
    chmod 770 -R /var/log/aria2c /var/local/aria2c

# aria2 will be started in the wrapper, then the real server
COPY wrapper.sh /
RUN chmod +x /wrapper.sh

# youtube-dl
RUN curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
RUN chmod a+rx /usr/local/bin/youtube-dl

# Set ram limit to 1G
RUN echo "memory_limit=2G" > /usr/local/etc/php/conf.d/memory-limit.ini
RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini

# Set a smaller number of workers for Apache2
# Not necessary for fpm-php image
RUN mv /etc/apache2/mods-available/mpm_prefork.conf /etc/apache2/mods-available/mpm_prefork.conf.orig
COPY etc/apache2/mods-available/mpm_prefork.conf /etc/apache2/mods-available/mpm_prefork.conf


# Not necessary for fpm-php image
CMD ["/bin/bash", "-c", "/wrapper.sh"]
# # CMD ["/usr/bin/supervisord", "-c", "/supervisord.conf"]


# For fpm-php image (not working)
# RUN su www-data aria2c --enable-rpc --rpc-allow-origin-all -c -D \
  # --log=/var/log/aria2c/aria2c.log \
  # --check-certificate=false \
  # --save-session=/var/local/aria2c/aria2c.sess \
  # --save-session-interval=2 \
  # --continue=true \
  # --input-file=/var/local/aria2c/aria2c.sess \
  # --rpc-save-upload-metadata=true \
  # --force-save=true\
  # --log-level=warn
