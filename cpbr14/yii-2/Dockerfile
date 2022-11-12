FROM dmstr/php-yii2:latest-nginx

ENV TZ=America/Sao_Paulo

RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y --no-install-recommends libc-client-dev libkrb5-dev unzip

COPY . /app

WORKDIR /app

RUN chmod -R 777 /app/runtime /app/web/assets
RUN chown -R www-data:www-data /app/runtime /app/web/assets
CMD bash -c "composer install --optimize-autoloader --no-dev && supervisord -c /etc/supervisor/supervisord.conf"
