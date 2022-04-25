# Abstract

Run [`deluged`](https://deluge-torrent.org/) in an isolated environment, so
that it cannot make direct connections.

A very simple and reliable measure forces all traffic through the proxy.

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
  Firewall &rarr; Custom Rules or, alternatively:

  * Network &rarr; Interfaces &rarr; Add new interface
    * Name: `deluged`
    * Protocol: `Unmanaged`
    * Custom Interface: `ppp-deluged`
  * Network &rarr; Firewall &rarr; General Settings &rarr; Zones &rarr; Add
    * Name: `deluged`
    * Covered networks: `deluged`
    * Inter-Zone Forwarding (both destination and source): `wan`

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

      deluge-via-proxy$ ./restart

* Start the client:

      deluge-via-proxy$ ./deluge-console

# Port forwarding

* If seeding performs badly, set the following in
  `~/var-lib-deluged/config/core.conf`:

      "random_port": false

      "listen_ports": [
        6881,
        6891
      ]

  and forward them to `192.168.77.2` on the server.

# How does it work

* `deluged` container is connected only to the internal `deluge` network.
  * Daemon cannot connect to the Internet directly - all packets are blocked,
    including pings, DNS requests and regular traffic.
  * Besides `deluged` itself, this container also runs `pppd`
    * One end of `pppd` is connected to a local interface. A default route
      points to a remote peer of this interface.
    * The other end is connected to port `7777` in `deluged-proxy` container.
* `deluged-proxy` container is connected to both `deluge` network and LAN.
  * Incoming connections to port `7777` are bound to new `ssh` connection to
    the server, which runs `pppd` command.
  * The effect is that `pppd` instances inside the `deluged` container and on
    the server talk to each other through the `ssh` tunnel.
  * SOCKS proxies are not used, because their way of relaying UDP
    is [incompatible with `ssh`](https://stackoverflow.com/questions/41967217),
    and `ssh`'s own `-D` option is not used, because it [does not support UDP](
    https://superuser.com/questions/639425).
* `deluged-public` container is also connected to both `deluge` network and
  LAN. It publishes `deluged`'s port 58846 in order for GUI clients to connect.
  It is needed, because `deluged` must not be attached to the host network, and
  therefore its ports cannot be published directly.
* `deluge-console` is `docker exec`ed inside the `deluged` container.

# Misc

* There appears to be a [bug with connection limiting in deluged](
  https://askubuntu.com/a/744411). Set `max_connections_global = -1` as a
  workaround.

* When running deluged on slow hardware, deleting unneeded torrents
  helps with performance.

# Debugging

* To debug [libtorrent](https://www.libtorrent.org), use:

      deluge-via-proxy$ ./debug

   and then either

       # strace \
           -f \
           -o strace.out \
           -s 4096 \
           -ttt \
           deluged \
               --do-not-daemonize \
               --config=/var/lib/deluged/config \
               --loglevel=debug &
       # less strace.out

   or

       # gdb --args \
           python $(which deluged) \
               --do-not-daemonize \
               --config=/var/lib/deluged/config \
               --loglevel=debug
