FROM nginx:alpine AS builder

ENV NGINX_VERSION 1.23.1
ENV HTTP_FLV_MODULE_VERSION 1.2.10


RUN wget "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O nginx.tar.gz && \
  wget "https://github.com/winshining/nginx-http-flv-module/archive/refs/tags/v${HTTP_FLV_MODULE_VERSION}.tar.gz" -O http-flv-module.tar.gz



RUN apk add --no-cache --virtual .build-deps \
  gcc \
  libc-dev \
  make \
  openssl-dev \
  pcre-dev \
  zlib-dev \
  linux-headers \
  curl \
  gnupg \
  libxslt-dev \
  gd-dev \
  geoip-dev


RUN rm -rf /usr/src/nginx /usr/src/extra_module && mkdir -p /usr/src/nginx /usr/src/extra_module && \
	tar -zxvC /usr/src/nginx -f nginx.tar.gz && \
    tar -xzvC /usr/src/extra_module -f http-flv-module.tar.gz

WORKDIR /usr/src/nginx/nginx-${NGINX_VERSION}


# Reuse same cli arguments as the nginx:alpine image used to build
RUN CONFARGS=$(nginx -V 2>&1 | sed -n -e 's/^.*arguments: //p') && \
    pwd && \
    ls -al && \
    sh -c "./configure --with-compat $CONFARGS --add-dynamic-module=/usr/src/extra_module/*" && make && make install

FROM nginx:alpine
# Extract the dynamic module NCHAN from the builder image
# COPY --from=builder /usr/local/nginx/modules/ngx_http_flv_live_module.so /usr/local/nginx/modules/ngx_http_flv_live_module.so

COPY --from=builder /usr/src/nginx/nginx-${NGINX_VERSION}/objs/*_module.so /usr/local/nginx/modules/

RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
EXPOSE 1935

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
