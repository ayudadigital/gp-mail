FROM alpine:3

LABEL org.opencontainers.image.source=https://github.com/ayudadigital/gp-mail
LABEL org.opencontainers.image.description="Generic Platform - Mail service"
LABEL org.opencontainers.image.licenses=MIT

# Packages
RUN apk add --no-cache postfix mailx rsync

# Allow mail send from all private network segments
RUN echo "mailbox_size_limit= 0 \
mynetworks = 127.0.0.0/8 192.168.0.0/16 10.0.0.0/8 172.16.0.0/12 169.254.0.0/16 \
inet_interfaces = all \
recipient_delimiter = + " >> /etc/postfix/main.cf

# Postfix configuration
RUN postconf -e 'myhostname = mail' && \
    postconf -e 'mydomain = disc-order.com' && \
    postconf -e 'myorigin = $mydomain' && \
    postconf -e 'smtp_generic_maps = hash:/etc/postfix/generic' && \
    postconf -e 'mynetworks = 127.0.0.1/32 192.168.0.0/16 172.16.0.0/12 10.0.0.0/8 [::1]' && \
    postconf -e 'mynetworks_style = class' && \
    postconf -e 'inet_interfaces = all' && \
    postconf -e 'relayhost = ' && \
    echo "postmaster@disc-order.com postmaster@disc-order.com" > /etc/postfix/generic && \
    postmap /etc/postfix/generic

# Log to stdout
RUN echo "postlog   unix-dgram n  -       n       -       1       postlogd" >> /etc/postfix/master.cf && \
    echo "maillog_file = /dev/stdout" >> /etc/postfix/main.cf

# Copy backup/cleanup scripts
COPY backup /usr/local/bin/backup
COPY cleanup /usr/local/bin/cleanup

# GP service identification
RUN echo "mail" > /gp.service

# Container command
CMD postfix start-fg
