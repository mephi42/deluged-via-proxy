ARG base
FROM $base
RUN apt-get -y update && \
    apt-get -y install \
        socat
COPY fs/ /
CMD ["deluged-public"]
