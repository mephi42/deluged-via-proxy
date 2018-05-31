# Abstract

Run [`deluged`](https://deluge-torrent.org/) via proxy in an isolated
environment, so that it cannot make direct connections.

A very simple and reliable measure must force all traffic through the proxy.

# Prerequisites

* Client
  * [Docker](https://www.docker.com/)
* Server:
  * running `sshd`
    * both [OpenSSH](https://www.openssh.com/) and [Dropbear](
      https://matt.ucc.asn.au/dropbear/dropbear.html) work
    * enabled [`GatewayPorts`](
      http://www.snailbook.com/faq/gatewayports.auto.html)
    * enabled [public key authentication](
      https://wiki.archlinux.org/index.php/SSH_keys) for user with
      [`NET_ADMIN`](http://man7.org/linux/man-pages/man7/capabilities.7.html)
      rights
  * installed [`pppd`](https://linux.die.net/man/8/pppd)

# Disclaimer

* Use at your own risk

# Usage

* On the server, add the following [iptables](
  https://linux.die.net/man/8/iptables) rule:

      # iptables -I FORWARD 1 -i ppp-deluged -j ACCEPT
  
  On [OpenWrt](https://openwrt.org/) it can be added through Network &rarr;
  Firewall &rarr; Custom Rules

* Add the following to `~/.ssh/config`:

      Host deluged-proxy
              Hostname <ssh server ip>
              Port <ssh server port>
              User <ssh server user>
              IdentityFile <ssh private key>

  The private key should be also in `~/.ssh` directory, otherwise it would not
  be shared with the container.

* Put `deluged` files into `~/var-lib-deluged`, so that it looks like this:

      ~/var-lib-deluged
      ├── Downloads
      └── config
          ├── auth
          └── core.conf

* Start the daemon and the proxy:

      deluge-via-proxy$ ./restart-deluged

* Make sure all traffic is really proxied:

      deluge-via-proxy$ curl ifconfig.co
      deluge-via-proxy$ docker exec -it deluged curl ifconfig.co

  The first command should print the local IP. The second command should print
  the `<ssh proxy ip>`.

* Start the client:

      deluge-via-proxy$ ./deluge-console

# How does it work

* `deluged` container is connected only to the internal `deluge` network.
  * Daemon cannot connect to the Internet directly - all packets are blocked,
    including pings, DNS requests and regular traffic.
  * Besides `deluged` itself, this container also runs `pppd`
    * One end of `pppd` is connected to a local interface. A default route
      points to a remote peer of this interface.
    * The other end is connected to port `7777` in `deluged-proxy` container.
* `deluged-proxy` container is connected to both `deluge` network and
  the Internet.
  * Incoming connections to port `7777` are bound to new `ssh` connection to
    the server, which runs `pppd` command.
  * The effect is that `pppd` instances inside the `deluged` container and on
    the server talk to each other through the `ssh` tunnel.
  * SOCKS proxies are not used, because their way of relaying UDP
    is [incompatible with `ssh`](https://stackoverflow.com/questions/41967217),
    and `ssh`'s own `-D` option is not used, because it [does not support UDP](
    https://superuser.com/questions/639425).
* `deluge-console` is `docker exec`ed inside the `deluged` container.

# Debugging

* To debug [libtorrent](https://www.libtorrent.org), add the following to
  `image-deluged/Dockerfile`:

      RUN apt-get -y install gdb gnupg2 less lsb-release strace
      RUN echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse"           >/etc/apt/sources.list.d/ddebs.list && \
          echo "deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse"  >>/etc/apt/sources.list.d/ddebs.list && \
          echo "deb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse" >>/etc/apt/sources.list.d/ddebs.list
      RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 428D7C01 C8CAB6595FDFF622
      RUN apt-get -y update && apt-get -y install libtorrent-rasterbar-dbg

   Then start the container:

      $ docker run \
          --interactive \
          --tty \
          --privileged \
          --volume="$(realpath ~/var-lib-deluged):/var/lib/deluged" \
          --network=deluge \
          "$(./docker-build image-deluged)" \
          bash

   and use either

       # strace \
           -f \
           -o strace.out \
           -s 4096 \
           -ttt \
           deluged \
               --do-not-daemonize \
               --config=/var/lib/deluged/config \
               --loglevel=debug &
       less strace.out

   or

       # gdb --args \
           python $(which deluged) \
               --do-not-daemonize \
               --config=/var/lib/deluged/config \
               --loglevel=debug
