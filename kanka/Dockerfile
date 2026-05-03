FROM node:20-alpine AS asset-builder
WORKDIR /app
COPY src .
RUN npm install --legacy-peer-deps && npm run build

FROM ghcr.io/serversideup/php:8.5-frankenphp

USER root

RUN install-php-extensions gd intl bcmath zip

RUN docker-php-serversideup-set-id www-data 1000:1000 && \
    docker-php-serversideup-set-file-permissions --owner 1000:1000 --service frankenphp

RUN install-php-extensions intl dom json zip

WORKDIR /var/www/html

COPY --chown=1000:1000 src .
# Copy compiled assets from the first stage
COPY --from=asset-builder --chown=1000:1000 /app/public /var/www/html/public

RUN chmod -R 775 storage bootstrap/cache

USER www-data

RUN composer install --no-dev --optimize-autoloader --no-interaction

RUN php artisan view:cache && \
    php artisan config:cache && \
    php artisan route:cache

LABEL org.opencontainers.image.source https://github.com/Alanaktion/self-contained
EXPOSE 8080
