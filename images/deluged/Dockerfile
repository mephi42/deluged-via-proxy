ARG base
FROM $base
RUN apt-get -y update && \
    apt-get -y install \
        libffi-dev \
        libssl-dev \
        ppp \
        python3-libtorrent \
        python3-pip \
        rustc \
        socat \
        tzdata
RUN python3 -m pip install deluge
COPY fs/ /
CMD ["deluged-proxied"]
