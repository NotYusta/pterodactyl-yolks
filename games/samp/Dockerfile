FROM        --platform=$TARGETOS/$TARGETARCH debian:bookworm-slim

LABEL       author="QuintenQVD" maintainer="josdekurk@gmail.com"

RUN         apt update \
    && apt -y upgrade
RUN         dpkg --add-architecture i386 \
    && apt update \
    && apt upgrade -y \
    && apt install -y libstdc++6 lib32stdc++6 tar wget curl iproute2 openssl fontconfig dirmngr ca-certificates dnsutils tzdata zip tini \
    && apt install -y libtbb12:i386 libtbb-dev:i386 libicu-dev:i386 libatomic1 libatomic1:i386 \
    && useradd -d /home/container -m container

RUN         mkdir -p /run/systemd && echo 'docker' > /run/systemd/container
RUN         rm -rf /var/lib/apt/lists/*

# Temp fix for things that still need libssl1.1 32 bit
RUN         if [ "$(uname -m)" = "x86_64" ]; then \
    wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_i386.deb && \
    dpkg -i libssl1.1_1.1.0g-2ubuntu4_i386.deb && \
    rm libssl1.1_1.1.0g-2ubuntu4_i386.deb; \
    fi

# Temp fix for legacy 32-bit MySQL plugin (libmysqlclient.so.18)
RUN         if [ "$(uname -m)" = "x86_64" ]; then \
                wget http://security.ubuntu.com/ubuntu/pool/main/m/mysql-5.5/libmysqlclient18_5.5.62-0ubuntu0.14.04.1_i386.deb && \
                dpkg -x libmysqlclient18_5.5.62-0ubuntu0.14.04.1_i386.deb /tmp/mysql18 && \
                cp -a /tmp/mysql18/usr/lib/i386-linux-gnu/libmysqlclient.so.18* /usr/lib/i386-linux-gnu/ && \
                rm -rf /tmp/mysql18 libmysqlclient18_5.5.62-0ubuntu0.14.04.1_i386.deb; \
            fi
        
USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

STOPSIGNAL SIGINT

COPY        --chown=container:container ./entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh
ENTRYPOINT    ["/usr/bin/tini", "-g", "--"]
CMD         ["/entrypoint.sh"]
