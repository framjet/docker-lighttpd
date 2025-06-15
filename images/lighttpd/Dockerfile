ARG UPTRACK_SOURCE=alpine:latest
ARG WAIT4X_VERSION=latest
FROM docker.io/atkrad/wait4x:${WAIT4X_VERSION} AS wait4x
FROM ${UPTRACK_SOURCE} AS base
ARG LIGHTTPD_VERSION=1.4.79-r0

LABEL org.opencontainers.image.title="Lighttpd Server"
LABEL org.opencontainers.image.authors="Aurimas Niekis <aurimas@niekis.lt>"
LABEL org.opencontainers.image.source="https://github.com/framjet/docker-lighttpd"
LABEL org.opencontainers.image.licenses="MIT"

RUN set -x \
    && apk add --no-cache \
    lighttpd${LIGHTTPD_VERSION:+=}${LIGHTTPD_VERSION} \
    && rm -rvf /var/cache/apk/* \
    && rm -rvf /etc/lighttpd/* /etc/logrotate.d/lighttpd /var/log/lighttpd /var/www/localhost \
    && mkdir /app \
    ;

COPY --chown=root docker-entrypoint.d/* /docker-entrypoint.d
COPY --chown=root docker-entrypoint.sh /docker-entrypoint.sh
COPY --chown=root lighttpd/ /etc/lighttpd/
COPY --chown=root index.html /app/index.html

ENV HTTP_ROOT=/app

EXPOSE 80/tcp

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["/usr/sbin/lighttpd", "-D", "-f", "/etc/lighttpd/lighttpd.conf"]
