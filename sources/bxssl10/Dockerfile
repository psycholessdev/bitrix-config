FROM alpine:3.21

RUN apk add --no-cache curl openssl bash mc ca-certificates tzdata

RUN mkdir -pv /opt /ssl

COPY cas.sh /opt/cas.sh
COPY srv.sh /opt/srv.sh
COPY dhparam.pem /ssl/dhparam.pem

RUN chown -R root:root /ssl && chown -R root:root /opt && ln -s /opt/cas.sh /root && ln -s /opt/srv.sh /root

#VOLUME ["/ssl"]

CMD ["tail", "-f", "/dev/null"]
