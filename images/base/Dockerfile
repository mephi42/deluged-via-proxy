FROM ubuntu:23.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -y update && \
    apt-get -y install \
        curl \
        iproute2 \
        inetutils-ping \
        net-tools \
        traceroute
COPY fs/ /
