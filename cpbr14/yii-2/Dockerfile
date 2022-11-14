FROM dmstr/php-yii2:latest-nginx

COPY . /app

WORKDIR /app

RUN chmod -R 777 /app/runtime /app/web/assets
RUN chown -R www-data:www-data /app/runtime /app/web/assets

CMD bash -c "composer install --optimize-autoloader --no-dev && supervisord -c /etc/supervisor/supervisord.conf"
