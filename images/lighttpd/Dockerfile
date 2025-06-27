ARG UPTRACK_SOURCE=framjet/alpine:latest
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

COPY --chown=root lighttpd/ /etc/lighttpd/
COPY --chown=root index.html /app/index.html

# Configure entrypoint
ENV DOCKER_ENTRYPOINT_COMMANDS="/usr/sbin/lighttpd"
ENV HTTP_ROOT=/app

EXPOSE 80/tcp

CMD ["/usr/sbin/lighttpd", "-D", "-f", "/etc/lighttpd/lighttpd.conf"]
