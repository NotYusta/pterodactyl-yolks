FROM        --platform=$TARGETOS/$TARGETARCH php:8.3-cli-bookworm

LABEL       author="Michael Parker" maintainer="parker@pterodactyl.io"

ENV         DEBIAN_FRONTEND=noninteractive

RUN         useradd -m -d /home/container container

STOPSIGNAL  SIGINT

RUN         apt update \
            && apt -y install \
                gnupg \
                ca-certificates \
                curl \
                wget \
                git \
                unzip \
                zip \
                tar \
                tzdata \
                locales \
                iproute2 \
                dnsutils \
                iputils-ping \
                libnss3 \
                tini \
                build-essential \
                libtool \
                ffmpeg \
                sqlite3 \
                libsqlite3-dev \
                libpng-dev \
                libjpeg-dev \
                libfreetype6-dev \
                libonig-dev \
                libxml2-dev \
                libzip-dev \
                libicu-dev \
                libxslt1-dev \
                libffi-dev \
                libedit-dev \
                libreadline-dev \
                python3 \
                python3-dev \
            && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
            && apt -y install nodejs \
            && apt clean \
            && rm -rf /var/lib/apt/lists/*

# Install global npm packages
RUN         npm install --global npm@latest typescript ts-node @types/node

# Install pnpm via corepack
RUN         npm install -g corepack
RUN         corepack enable
RUN         corepack prepare pnpm@latest --activate

# Install PHP extensions matching minetrax reference image
RUN         docker-php-ext-configure gd --with-freetype --with-jpeg \
            && docker-php-ext-install -j$(nproc) \
                bcmath \
                calendar \
                exif \
                ffi \
                ftp \
                gd \
                gettext \
                intl \
                mbstring \
                mysqli \
                pcntl \
                pdo_mysql \
                shmop \
                sockets \
                sysvmsg \
                sysvsem \
                sysvshm \
                xsl \
                zip \
            && pecl install redis igbinary \
            && docker-php-ext-enable redis igbinary

# Install Composer
RUN         curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

COPY        --chown=container:container ./entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh
ENTRYPOINT  ["/usr/bin/tini", "-g", "--"]
CMD         ["/entrypoint.sh"]
