FROM alpine
LABEL mainteiner="SergSha"
RUN apk add --update --no-cache php-fpm
RUN mkdir -p /var/lib/nginx/html
RUN sed -i "s/listen = 127.0.0.1:9000/listen = [::]:9000/" /etc/php8/php-fpm.d/www.conf
#RUN echo "catch_workers_output = Yes" >> /etc/php8/php-fpm.d/www.conf
#COPY www.conf /etc/php-fpm/php-fpm.d/www.conf
EXPOSE 9000
CMD ["php-fpm8","-F"]

