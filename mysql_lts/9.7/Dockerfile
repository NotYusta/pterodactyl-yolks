# -----------------------------------------------------
# MySQL 9.7 LTS Image for Pterodactyl
# -----------------------------------------------------
FROM        --platform=$TARGETOS/$TARGETARCH ghcr.io/notyusta/image-yolks:mysql_9.7

LABEL       author="Michael Parker" maintainer="parker@pterodactyl.io"

RUN         useradd -d /home/container -m container -s /bin/bash

USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

COPY        ../entrypoint.sh /entrypoint.sh
CMD ["/bin/bash", "/entrypoint.sh"]
