ARG base
FROM $base
RUN apt-get -y update && \
    apt-get -y install \
        cargo \
        libffi-dev \
        libssl-dev \
        pkg-config \
        ppp \
        python3-cryptography \
        python3-libtorrent \
        python3-pip \
        rustc \
        socat \
        tzdata
RUN python3 -m pip install --break-system-packages --upgrade pip setuptools
RUN python3 -m pip install --break-system-packages --upgrade deluge
COPY fs/ /
CMD ["deluged-proxied"]
