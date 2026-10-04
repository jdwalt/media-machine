FROM automaticrippingmachine/automatic-ripping-machine:2.24.3

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        intel-media-va-driver-non-free \
        vainfo \
        libmfx-tools \
    && rm -rf /var/lib/apt/lists/*

COPY install_handbrake-1.9.2.sh /tmp/install_handbrake-1.9.2.sh

RUN grep -qx 'HANDBRAKE_VERSION="1.9.2"' \
        /tmp/install_handbrake-1.9.2.sh \
    && bash /tmp/install_handbrake-1.9.2.sh \
    && HandBrakeCLI --version 2>&1 | grep -F 'HandBrake 1.9.2' \
    && test -e /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so \
    && rm -f /tmp/install_handbrake-1.9.2.sh

ENV LIBVA_DRIVER_NAME=iHD
