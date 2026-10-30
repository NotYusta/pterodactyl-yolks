FROM        --platform=$TARGETOS/$TARGETARCH ubuntu:24.04

LABEL       author="Michael Parker" maintainer="parker@pterodactyl.io"

LABEL       org.opencontainers.image.source="https://github.com/NotYusta/pterodactyl-yolks"
LABEL       org.opencontainers.image.licenses=MIT

ARG         TARGETARCH
ARG         GRAALVM_VERSION=21.2.0
ARG         GRAALVM_RELEASE=vm-21.2.0

ENV         JAVA_HOME=/opt/java/openjdk
ENV         PATH=$JAVA_HOME/bin:$PATH

## libssl3t64 is required by native libraries that link against OpenSSL 3
## (libcrypto.so.3), e.g. Geyser's NetherNet/libdatachannel and many other
## JNI/native plugins. Without it those fail with UnsatisfiedLinkError.
RUN         apt-get update \
            && apt-get install -y --no-install-recommends \
                curl \
                ca-certificates \
                openssl \
                libssl3t64 \
                zlib1g \
                libstdc++6 \
                libatomic1 \
                libfreetype6 \
                fontconfig \
                lsof \
                git \
                tar \
                sqlite3 \
                tzdata \
                iproute2 \
                redis-tools \
                tini \
                zip \
                unzip \
                jq \
            && rm -rf /var/lib/apt/lists/*

## Install GraalVM Community (JDK $GRAALVM_VERSION) instead of pulling a
## third-party prebuilt image, so this image is fully built and owned here.
RUN         set -eux; \
            arch="${TARGETARCH:-$(dpkg --print-architecture)}"; \
            case "$arch" in \
                amd64) graal_arch="amd64" ;; \
                *) echo "GraalVM has no $arch build for Java 8 (amd64 only)" >&2; exit 1 ;; \
            esac; \
            url="https://github.com/graalvm/graalvm-ce-builds/releases/download/${GRAALVM_RELEASE}/graalvm-ce-java8-linux-${graal_arch}-${GRAALVM_VERSION}.tar.gz"; \
            curl -fsSL -o /tmp/graalvm.tar.gz "$url"; \
            curl -fsSL -o /tmp/graalvm.tar.gz.sha256 "$url.sha256"; \
            echo "$(cat /tmp/graalvm.tar.gz.sha256)  /tmp/graalvm.tar.gz" | sha256sum -c -; \
            mkdir -p "$JAVA_HOME"; \
            tar -xzf /tmp/graalvm.tar.gz -C "$JAVA_HOME" --strip-components=1; \
            rm -f /tmp/graalvm.tar.gz /tmp/graalvm.tar.gz.sha256; \
            java -version; \
            javac -version

## Setup user and working directory
RUN         useradd -m -d /home/container -s /bin/bash container
USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

STOPSIGNAL SIGINT

COPY        --chown=container:container ./../entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh
ENTRYPOINT    ["/usr/bin/tini", "-g", "--"]
CMD         ["/entrypoint.sh"]
