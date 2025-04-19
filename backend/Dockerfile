FROM php:8.3.11-fpm

# Update package list and install dependencies
RUN apt-get update && apt-get install -y \
    libzip-dev \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    postgresql-client \
    libpq-dev \
    nodejs \
    npm \
    unzip \
    git \
    curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions including pdo_mysql for MySQL support
RUN docker-php-ext-install pdo pdo_mysql pgsql pdo_pgsql gd bcmath zip \
    && pecl install redis \
    && docker-php-ext-enable redis

# Install Composer (latest stable)
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

ENV COMPOSER_ALLOW_SUPERUSER=1

# Set working directory
WORKDIR /usr/share/nginx/html/

# Copy project files into the container
COPY . ./

# Install composer dependencies (production)
RUN sed 's_@php artisan package:discover_/bin/true_;' -i composer.json \
    && composer install --ignore-platform-req=php --no-dev --optimize-autoloader \
    && composer clear-cache \
    && php artisan package:discover --ansi \
    && chmod -R 775 storage \
    && chown -R www-data:www-data storage \
    && mkdir -p storage/framework/sessions storage/framework/views storage/framework/cache

# Copy entrypoint script
COPY ./scripts/php-fpm-entrypoint /usr/local/bin/php-entrypoint

# Make entrypoint executable
RUN chmod a+x /usr/local/bin/php-entrypoint

ENTRYPOINT ["/usr/local/bin/php-entrypoint"]

CMD ["php-fpm"]
