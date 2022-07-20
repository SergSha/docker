FROM alpine
LABEL mainteiner="SergSha"
RUN apk add --update --no-cache nginx && mkdir -p /run/nginx
#COPY ./nginx/index.html /var/lib/nginx/html/index.html
COPY ./nginx/default.conf /etc/nginx/http.d/default.conf
EXPOSE 80
CMD ["nginx","-g","daemon off;"]

