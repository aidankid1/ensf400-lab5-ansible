# https://github.com/panubo/docker-sshd
FROM alpine:3.16 as alpine-systemd

# Install necessary packages in a single step (Python 3, SSH, and dependencies)
RUN apk --update --no-cache add \
    python3 py3-pip \
    openssh bash git rsync augeas shadow openrc && \
    ln -sf /sbin/openrc-init /sbin/init && \
    ln -sf /usr/bin/python3 /usr/bin/python && \
    rm -rf /var/cache/apk/*  # Clean up package cache

ENTRYPOINT ["/sbin/init"]

# Build SSH server
FROM alpine-systemd as alpine-sshd

RUN apk update && \
    apk add --no-cache \
    bash git openssh rsync augeas shadow nodejs npm && \ 
    deluser $(getent passwd 33 | cut -d: -f1) && \
    delgroup $(getent group 33 | cut -d: -f1) 2>/dev/null || true && \
    mkdir -p ~root/.ssh /etc/authorized_keys && chmod 700 ~root/.ssh/ && \
    augtool 'set /files/etc/ssh/sshd_config/AuthorizedKeysFile ".ssh/authorized_keys /etc/authorized_keys/%u"' && \
    echo -e "Port 2224\n" >> /etc/ssh/sshd_config && \
    cp -a /etc/ssh /etc/ssh.cache && \
    ln -sf /usr/bin/python3 /usr/bin/python && \
    rm -rf /var/cache/apk/*

EXPOSE 2224

COPY docker-entrypoint.sh /entry.sh
RUN chmod a+x /entry.sh
ENTRYPOINT ["/entry.sh"]

CMD ["/usr/sbin/sshd", "-D", "-e", "-f", "/etc/ssh/sshd_config"]
