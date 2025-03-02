FROM node:20 AS builder

ENV PUSH=push-server.tar.gz

RUN mkdir -p /opt/push-server

COPY push-server-0.4.0.tar.gz /opt/push-server/$PUSH

WORKDIR /opt/push-server

RUN tar xzvvf $PUSH && rm -f $PUSH

RUN npm install --omit=dev

FROM node:20-alpine AS production

RUN apk --no-cache add shadow && usermod -u 6006 node && groupmod -g 6006 node

WORKDIR /opt/push-server

RUN mkdir -p /usr/local/bin && apk add --no-cache bash gettext libintl && mv /usr/bin/envsubst /usr/local/bin/envsubst

COPY --from=builder /opt/push-server /opt/push-server

RUN chown -R node:node /opt/push-server && \
    [[ ! -d /tmp/push-server ]] && mkdir /tmp/push-server && \
    chown -R node:node /tmp/push-server && \
    [[ ! -d /etc/push-server ]] && mkdir /etc/push-server

COPY docker-entrypoint.sh /usr/local/bin/
COPY push-server-pub.json /etc/push-server/push-server-pub.json
COPY push-server-sub.json /etc/push-server/push-server-sub.json

RUN chown -R node:node /etc/push-server

RUN chmod 0755 /usr/local/bin/docker-entrypoint.sh

RUN mkdir -p /var/log/push-server

RUN chown -R node:node /var/log/push-server

# VOLUME /var/log/push-server

USER node:node

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
