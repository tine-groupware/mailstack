FROM ubuntu:24.04

RUN apt-get update && \
    apt-get dist-upgrade -y && \
    apt-get install -y \
                postfix ...

# CMD ["/usr/sbin/dovecot", "-c", "/etc/dovecot/dovecot.conf", "-F"]
