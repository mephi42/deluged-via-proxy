FROM ubuntu
RUN apt-get -y update && \
    apt-get -y install \
        curl \
        deluged \
        deluge-console \
        iproute2 \
        inetutils-ping \
        net-tools \
        ppp \
        socat \
        tzdata
COPY fs/ /
CMD ["deluged-proxied"]
