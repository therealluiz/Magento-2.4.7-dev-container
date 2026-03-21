FROM php:8.3-fpm-bookworm

ARG USER_ID=1000
ARG GROUP_ID=1000

# System dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    unzip \
    curl \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libicu-dev \
    libxml2-dev \
    libxslt1-dev \
    libzip-dev \
    libsodium-dev \
    libonig-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# PHP extensions required by Magento 2.4.7
RUN docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
    && docker-php-ext-install -j$(nproc) \
        bcmath \
        ctype \
        curl \
        dom \
        fileinfo \
        filter \
        gd \
        iconv \
        intl \
        mbstring \
        pdo_mysql \
        simplexml \
        soap \
        sockets \
        sodium \
        tokenizer \
        xml \
        xmlwriter \
        xsl \
        zip \
        opcache

# Xdebug 3
RUN pecl install xdebug-3.3.2 \
    && docker-php-ext-enable xdebug

# SPX Profiler (free Blackfire alternative with built-in web UI)
# https://github.com/NoiseByNorthwest/php-spx
RUN cd /tmp \
    && git clone --depth=1 https://github.com/NoiseByNorthwest/php-spx.git \
    && cd /tmp/php-spx \
    && phpize \
    && ./configure \
    && make -j$(nproc) \
    && make install \
    && docker-php-ext-enable spx \
    && rm -rf /tmp/php-spx

# Composer 2.7
COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# Magerun2
RUN curl -sL https://files.magerun.net/n98-magerun2.phar -o /usr/local/bin/magerun2 \
    && chmod +x /usr/local/bin/magerun2

# Node.js 18 (for frontend builds)
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# Create magento user
RUN groupadd -g ${GROUP_ID} magento \
    && useradd -m -u ${USER_ID} -g magento -s /bin/bash magento

# PHP config
COPY php.ini /usr/local/etc/php/conf.d/magento.ini
COPY xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
COPY spx.ini /usr/local/etc/php/conf.d/docker-php-ext-spx.ini

# Composer cache directory
RUN mkdir -p /home/magento/.composer && chown -R magento:magento /home/magento

WORKDIR /var/www/html

USER magento
