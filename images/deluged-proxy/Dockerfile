ARG base
FROM $base
RUN apt-get -y update && \
    apt-get -y install \
        socat \
        ssh
COPY fs/ /
CMD ["deluged-proxy"]
