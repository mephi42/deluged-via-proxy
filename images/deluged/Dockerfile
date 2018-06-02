ARG base
FROM $base
RUN apt-get -y update && \
    apt-get -y install \
        deluged \
        deluge-console \
        ppp \
        socat \
        tzdata
COPY fs/ /
CMD ["deluged-proxied"]
