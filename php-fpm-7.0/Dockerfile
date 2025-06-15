FROM php:7.0-fpm

# 安裝必要的擴展
# Change apt sources to use archive.debian.org for Stretch
RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/archive.debian.org/g' /etc/apt/sources.list \
    && sed -i '/stretch-updates/d' /etc/apt/sources.list \
    && echo "deb http://archive.debian.org/debian stretch-backports main" >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends --allow-unauthenticated \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libwebp-dev \
    libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    libmcrypt-dev \
    libicu-dev \
    libmagickwand-dev \
    librabbitmq-dev \
    redis-tools \
    imagemagick \
    # For common CMS/frameworks
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd pdo pdo_mysql opcache zip mbstring exif bcmath intl soap pcntl \
    && pecl install imagick redis amqp \
    && docker-php-ext-enable pcntl \
    # Manually enable PECL extensions
    && echo "extension=imagick.so" > /usr/local/etc/php/conf.d/20-imagick.ini \
    && echo "extension=redis.so" > /usr/local/etc/php/conf.d/20-redis.ini \
    && echo "extension=amqp.so" > /usr/local/etc/php/conf.d/20-amqp.ini \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

# 設定 php.ini (選用，您可以透過掛載來覆蓋)
COPY php.ini /usr/local/etc/php/conf.d/custom.ini