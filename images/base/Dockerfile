FROM ubuntu
RUN apt-get -y update && \
    apt-get -y install \
        curl \
        iproute2 \
        inetutils-ping \
        net-tools \
        traceroute
COPY fs/ /
