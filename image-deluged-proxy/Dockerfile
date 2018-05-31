FROM ubuntu
RUN apt-get -y update && \
    apt-get -y install \
        curl \
        iproute2 \
        inetutils-ping \
        socat \
        ssh \
        net-tools
COPY fs/ /
CMD ["deluged-proxy"]
