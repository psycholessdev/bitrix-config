FROM nginx:1.26.3-alpine-slim

RUN apk add git && \
    git clone https://github.com/cyrax13/nginx-modules.git && \
    echo "/root/nginx-modules/packages/1.26.3/alpine/v3.20/main" >> /etc/apk/repositories && \
    apk --allow-untrusted update && \
    apk --allow-untrusted add nginx-module-xslt nginx-module-geoip nginx-module-image-filter nginx-module-njs nginx-module-perl nginx-module-brotli nginx-module-geoip2 nginx-module-ndk nginx-module-lua nginx-module-headers-more nginx-module-zip && \
    apk del git && \
    rm -rf /root/nginx-modules
