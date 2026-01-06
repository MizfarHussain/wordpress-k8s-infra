FROM openresty/openresty:alpine

RUN apk add --no-cache gcc make musl-dev pcre-dev zlib-dev openssl-dev

RUN ./configure \
 --prefix=/opt/openresty \
 --with-pcre-jit \
 --with-ipv6 \
 --without-http_redis2_module \
 --with-http_iconv_module \
 --with-http_postgres_module -j8

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

EXPOSE 80
