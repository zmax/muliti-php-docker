FROM php:8.3-fpm

# 安裝必要的擴展
RUN apt-get update && apt-get install -y \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libwebp-dev \
    libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    libicu-dev \
    libmagickwand-dev \
    librabbitmq-dev \
    redis-tools \
    # For CraftCMS
    imagemagick \
    # For common CMS/frameworks
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd pdo pdo_mysql opcache zip mbstring exif bcmath intl soap \
    && pecl install imagick redis amqp \
    && docker-php-ext-enable imagick redis amqp \
    && docker-php-ext-install pcntl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

# 設定 php.ini (選用，您可以透過掛載來覆蓋)
COPY php.ini /usr/local/etc/php/conf.d/custom.ini