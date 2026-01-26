ARG DOCKER_HUB_PROXY=""

FROM ${DOCKER_HUB_PROXY}hairyhenderson/gomplate:stable-alpine AS gomplate

FROM ${DOCKER_HUB_PROXY}library/ubuntu:noble AS postfix
COPY --from=gomplate /bin/gomplate /usr/bin/gomplate

RUN apt-get update \
 && apt-get install -y \
    busybox \
    postfix \
    postfix-mysql
RUN mkdir /usr/local/busybox \
 && busybox --install /usr/local/busybox
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/busybox

RUN mkdir -p /etc/gomplate

COPY images/postfix/entrypoint /usr/local/entrypoint

CMD ["/usr/local/entrypoint"]