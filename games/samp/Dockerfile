# Builds a 32-bit MySQL client library that supports caching_sha2_password.
# The SA-MP MySQL plugin links against the legacy libmysqlclient.so.18
# (MySQL 5.5), which predates caching_sha2_password (MySQL 8 default) and
# fails with "Authentication plugin 'caching_sha2_password' cannot be loaded".
# MariaDB Connector/C is a drop-in replacement for that soname and, when built
# with OpenSSL, performs caching_sha2_password RSA full authentication without
# requiring the application to set MYSQL_OPT_GET_SERVER_PUBLIC_KEY.
FROM        --platform=linux/386 debian:bookworm-slim AS mysqlclient

RUN         apt update \
    && apt install -y --no-install-recommends git cmake make gcc libc6-dev libssl-dev ca-certificates \
    && git clone -q --depth 1 --branch v3.3.21 https://github.com/mariadb-corporation/mariadb-connector-c.git /src \
    && cmake -S /src -B /src/build \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/opt/mysqlclient \
        -DINSTALL_LIBDIR=lib \
        -DWITH_SSL=OPENSSL \
        -DWITH_CURL=OFF \
        -DWITH_ZSTD=OFF \
        -DWITH_UNIT_TESTS=OFF \
        -DCLIENT_PLUGIN_CACHING_SHA2_PASSWORD=STATIC \
        -DCLIENT_PLUGIN_SHA256_PASSWORD=STATIC \
        -DCLIENT_PLUGIN_DIALOG=STATIC \
        -DCLIENT_PLUGIN_MYSQL_CLEAR_PASSWORD=STATIC \
    && cmake --build /src/build -j"$(nproc)" \
    && cmake --install /src/build \
    && rm -rf /src

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

# 32-bit MySQL client with caching_sha2_password support, exposed under the
# legacy libmysqlclient.so.18 soname the SA-MP MySQL plugin was built against
COPY        --from=mysqlclient /opt/mysqlclient/lib/libmariadb.so.3 /usr/lib/i386-linux-gnu/libmariadb.so.3
RUN         if [ "$(uname -m)" = "x86_64" ]; then \
                apt update && apt install -y libssl3:i386 && rm -rf /var/lib/apt/lists/* && \
                ln -sf libmariadb.so.3 /usr/lib/i386-linux-gnu/libmysqlclient.so.18 && \
                ln -sf libmariadb.so.3 /usr/lib/i386-linux-gnu/libmysqlclient.so && \
                ldconfig; \
            fi

USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

STOPSIGNAL SIGINT

COPY        --chown=container:container ./entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh
ENTRYPOINT    ["/usr/bin/tini", "-g", "--"]
CMD         ["/entrypoint.sh"]
