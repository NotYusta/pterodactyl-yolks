# -----------------------------------------------------
# MySQL 8.4 LTS Image for Pterodactyl
# -----------------------------------------------------
FROM        --platform=$TARGETOS/$TARGETARCH ghcr.io/notyusta/image-yolks:mysql_8.4

LABEL       author="Michael Parker" maintainer="parker@pterodactyl.io"

RUN         useradd -d /home/container -m container -s /bin/bash

USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

COPY        ../entrypoint.sh /entrypoint.sh
CMD ["/bin/bash", "/entrypoint.sh"]
