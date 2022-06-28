FROM alpine:3.10

# Install packages
RUN apk -U --no-cache add \
    build-base \
    file \
    git \
    make \
    php7 \
    php7-dev \
    php7-tokenizer \
    python3 \
    python3-dev \
    re2c

# Install BFR
COPY ./bfr /opt/BFR
WORKDIR /opt/BFR
RUN phpize7 \
    && ./configure --with-php-config=/usr/bin/php-config7 --enable-bfr \
    && make \
    && make install
WORKDIR /
RUN rm -rf /opt/BFR /tmp/* /var/tmp/* \
    && echo "zend_extension = "$(find /usr -name bfr.so) >> /etc/php7/php.ini

# Install PHP Sandbox
COPY ./phpox /opt/phpox
WORKDIR /opt/phpox
RUN python3 -m pip install -r requirements.txt \
    && make

# Cleanup
RUN apk del --purge \
    build-base \
    git \
    php7-dev \
    python3-dev \
    && rm -rf /root/* \
    && rm -rf /var/cache/apk/*

# Set workdir and start phpsandbox
USER nobody:nobody
WORKDIR /opt/phpox
CMD ["python3", "sandbox.py"]
