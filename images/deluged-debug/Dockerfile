ARG base
FROM $base
RUN apt-get -y update && \
    apt-get -y install \
        gdb \
        gnupg2 \
        less \
        lsb-release \
        strace
RUN echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse"           >/etc/apt/sources.list.d/ddebs.list && \
    echo "deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse"  >>/etc/apt/sources.list.d/ddebs.list && \
    echo "deb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse" >>/etc/apt/sources.list.d/ddebs.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 428D7C01 C8CAB6595FDFF622
RUN apt-get -y update && \
    apt-get -y install \
        libtorrent-rasterbar-dbg
