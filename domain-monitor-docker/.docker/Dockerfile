# 1. Using PHP 8.2 to avoid "curl_close() deprecated" errors found in 8.5
FROM php:8.2-apache

# 2. Install system dependencies & PHP extensions
RUN apt-get update && apt-get install -y --no-install-recommends \
    libicu-dev libzip-dev libpng-dev libjpeg62-turbo-dev libfreetype6-dev \
    libxml2-dev libcurl4-openssl-dev libonig-dev pkg-config unzip git \
    && docker-php-ext-configure gd --with-jpeg --with-freetype \
    && docker-php-ext-install -j"$(nproc)" \
    pdo pdo_mysql mysqli intl zip gd bcmath mbstring curl xml \
    && a2enmod rewrite headers \
    && rm -rf /var/lib/apt/lists/*

# 3. Install Composer globally
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# 4. Set Working Directory
WORKDIR /var/www/html

# 5. Copy everything from the ROOT (context is '.')
COPY . .

# 6. Copy the specific .env from the subfolder to the app root
RUN cp domain-monitor-docker/.env .env

# 7. Run Composer
RUN composer install --no-interaction --prefer-dist --no-dev

# 8. Set Permissions (Equivalent to bootstrap.sh logic)
RUN chown -R www-data:www-data /var/www/html \
    && find /var/www/html -type d -print0 | xargs -0 chmod 755 \
    && find /var/www/html -type f -print0 | xargs -0 chmod 644 \
    && chmod -R 775 /var/www/html/public/assets/uploads 2>/dev/null || true

# 9. Configure Apache & Fix Redirect Loops
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/000-default.conf

# Add this to handle headers from Coolify's proxy
RUN echo 'SetEnvIf X-Forwarded-Proto "https" HTTPS=on' > /etc/apache2/conf-available/proxy.conf \
    && a2enconf proxy.conf

EXPOSE 80