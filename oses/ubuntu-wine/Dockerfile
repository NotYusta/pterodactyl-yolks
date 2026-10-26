# ---------------------------------------
# Ubuntu + Wine runtime (own build)
# ---------------------------------------
# Replaces hcgcloud/pterodactyl-images:ubuntu-wine. Compared to that image
# (Ubuntu 20.04 + wine 5.0, last rebuilt 2020) this one ships Ubuntu 24.04,
# wine 9, a current toolchain and a 32-bit MySQL client with
# caching_sha2_password support, so plugins built against the legacy
# libmysqlclient.so.18 (SA-MP / MTA MySQL plugins) work with MySQL 8+.

# 32-bit MySQL client library with caching_sha2_password support.
# MariaDB Connector/C is a drop-in replacement for libmysqlclient.so.18 and,
# built with OpenSSL, also does the caching_sha2 RSA full authentication over
# unencrypted connections without the application setting
# MYSQL_OPT_GET_SERVER_PUBLIC_KEY.
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

FROM        --platform=$TARGETOS/$TARGETARCH ubuntu:24.04

LABEL       author="NotYusta" maintainer="NotYusta"

LABEL       org.opencontainers.image.source="https://github.com/NotYusta/pterodactyl-yolks"
LABEL       org.opencontainers.image.licenses=MIT

ENV         DEBIAN_FRONTEND=noninteractive

## Base packages + wine (both architectures)
RUN         dpkg --add-architecture i386 \
            && apt update \
            && apt upgrade -y \
            && apt install -y --no-install-recommends \
                binutils bzip2 ca-certificates cabextract curl file fontconfig gcc g++ git \
                iproute2 libatomic1 libntlm0 locales make net-tools netcat-openbsd \
                tar telnet tini tzdata unzip wget xauth xvfb xz-utils zip \
            && apt install -y wine wine64 wine32:i386 fonts-wine winbind winetricks \
            && apt install -y libssl3t64:i386

## Set up Winetricks
RUN         wget -q -O /usr/sbin/winetricks https://raw.githubusercontent.com/pelican-eggs/winetricks/master/src/winetricks \
            && chmod +x /usr/sbin/winetricks

## Ubuntu's wine 9 has a single binary; keep the wine64 name working because
## eggs (e.g. SA-MP-Windows) call it directly
RUN         ln -sf /usr/bin/wine /usr/bin/wine64

## Configure locale
RUN         update-locale lang=en_US.UTF-8 \
            && dpkg-reconfigure --frontend noninteractive locales

## 32-bit MySQL client with caching_sha2_password support, exposed under the
## legacy libmysqlclient.so.18 soname the SA-MP / MTA plugins were built against
COPY        --from=mysqlclient /opt/mysqlclient/lib/libmariadb.so.3 /usr/lib/i386-linux-gnu/libmariadb.so.3
RUN         ln -sf libmariadb.so.3 /usr/lib/i386-linux-gnu/libmysqlclient.so.18 \
            && ln -sf libmariadb.so.3 /usr/lib/i386-linux-gnu/libmysqlclient.so \
            && ldconfig

## Add container user
RUN         useradd -m -d /home/container -s /bin/bash container
ENV         USER=container HOME=/home/container
USER        container

## Wine defaults
ENV         WINEARCH=win64
ENV         WINEPREFIX=/home/container/.wine64
ENV         WINEDEBUG=-all
ENV         WINEDLLOVERRIDES="mscoree,mshtml="
ENV         DISPLAY=:0
ENV         DISPLAY_WIDTH=1024
ENV         DISPLAY_HEIGHT=768
ENV         DISPLAY_DEPTH=16
ENV         XVFB=0

WORKDIR     /home/container

STOPSIGNAL  SIGINT

COPY        --chown=container:container ./entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh
ENTRYPOINT  ["/usr/bin/tini", "-g", "--"]
CMD         ["/entrypoint.sh"]
