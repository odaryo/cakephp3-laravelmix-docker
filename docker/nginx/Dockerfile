ARG NGINX_VERSION

FROM nginx:${NGINX_VERSION}-alpine

# ルートディレクトリを作成
RUN mkdir -p /var/www

ARG APP_CONTAINER
ARG USER_ID

# ツールをインストール
RUN apk update \
    && apk upgrade \
    && apk add --no-cache \
        bash \
        openssl \
    && adduser -D -H -u ${USER_ID} -s /bin/bash www-data

# default.confのルートディレクトリを変更
ADD ./conf.d/ /etc/nginx/conf.d/
RUN sed -i -e "s|APP_CONTAINER|${APP_CONTAINER}|g" /etc/nginx/conf.d/default.conf

# # 自己証明書を発行
# RUN openssl genrsa 2048 > server.key \
#  && openssl req -new -key server.key -subj "/C=JP/ST=Hiroshima/L=Naka-ku/O=UP-net/OU=Ltd/CN=localhost" > server.csr \
#  && openssl x509 -in server.csr -days 3650 -req -signkey server.key > server.crt \
#  && cp server.crt /etc/nginx/server.crt \
#  && cp server.key /etc/nginx/server.key \
#  && chmod 755 -R /var/www \
#  && chmod 400 /etc/nginx/server.key

WORKDIR ${APP_CONTAINER}