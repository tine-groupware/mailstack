FROM ubuntu:24.04

RUN apt-get update && \
    apt-get dist-upgrade -y && \
    apt-get install -y \
                dovecot-core \
                dovecot-mysql mysql-client \
                dovecot-managesieved dovecot-lmtpd dovecot-imapd dovecot-pop3d \
                dovecot-antispam

CMD ["/usr/sbin/dovecot", "-c", "/etc/dovecot/dovecot.conf", "-F"]
