FROM alpine:3.21

RUN addgroup -g 9999 sphinx && \
    adduser -D -u 9999 -G sphinx sphinx

RUN apk add --no-cache bash expat libgcc libpq libstdc++ mariadb-connector-c musl unixodbc zlib sphinx sphinx-doc sphinx-php curl openssl ca-certificates tzdata && \
    mkdir -pv /opt/sphinx/conf /opt/sphinx/log /opt/sphinx/run /opt/sphinx/dict /opt/sphinx/lib /opt/sphinx/binlogs

COPY start.sh /opt/sphinx/
COPY *.pak /opt/sphinx/dict/
COPY sample_sphinx.conf /opt/sphinx/conf/sphinx.conf

RUN ln -sf /dev/stdout /opt/sphinx/log/searchd.log && \
    ln -sf /dev/stderr /opt/sphinx/log/query.log && \
    chown -R sphinx:sphinx /opt/

# VOLUME ["/opt/sphinx/conf", "/opt/sphinx/log", "/opt/sphinx/dict", "/opt/sphinx/lib"]

USER sphinx:sphinx

EXPOSE 9312 9306

CMD ["bash", "/opt/sphinx/start.sh"]
