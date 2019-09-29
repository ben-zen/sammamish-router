# Setting up the router

Different guides around the internet have vastly different suggestions on how
to handle routing, configuring interfaces, and DNS and DHCP; after trying out
NetworkManager for handling interfaces, it became clear that this was not going
to end well, and instead I moved to using systemd's networkd directly.

## Essential services

### SSH

YOu gotta use SSH. It's the way to talk to a server! Since it's also 2019, we
can use newer standards than RSA-4096 for our keys. Get with the times and use
some elliptic curves! Use Curve25519, it's fine, really. You too can have your
public key on a single line, and read it too.

### DHCP and SLAAC

DHCP is going to be served by dnsmasq on this server, which is a matter of the
configuration of both dnsmasq and firewalld.

#### dnsmasq DHCP configuration changes

These changes are all in `/etc/dnsmasq.conf`; the first one is applicable to
all of the services dnsmasq offers, while the rest are specific to DHCP; if you
have a personal domain name, consider using it here!

- `interface=eno2` : Enable listening for requests on the port I chose for LAN
- `domain=internal.home.net` : Pick a domain name to use for the network!
- `dhcp-range=10.0.1.0,10.0.1.200.12h`: I decided on the `10.0.1.0/24` network

#### firewalld configuration

Of course, having a daemon running to provide DHCP responses and track devices
is only useful if it can receive and respond to those requests! I had to set
interface `eno2` to the `internal` zone, which I enabled DNS and DHCP services
on--i.e., traffic flowing over the standard ports for those protocols.

### DNS

#### dnsmasq DNS configuration changes

Again, there are specific changes to set to make sure dnsmasq is correctly set
up. Pick DNS servers you trust and are useful to you!

- `server=8.8.8.8`, `server=1.1.1.1`: Pick a few public DNS servers to use.
- `no-resolv`: Don't read `/etc/resolv.conf`; I wanted this config file only.
- `address=/router.lan/10.0.1.1`: Add your router's internal domain name and
  ip address, since the router is not going to use DHCP to assign its address,
  and so can't get added to the database automatically.

I also took a chance to make sure I've mitigated a nasty vuln; search CERT in
the config file to learn more.

## Traffic routing

Set up the IPForward rule for all interfaces you want traffic to cross, and
establish any particular routing operations you want to forward; for example,
if there's an internal server that specifically handles HTTP/S or SSH, you can
set that in firewalld.

## Additional services

### Wireguard VPN

One benefit of building a particularly powerful machine to work as a router is
hosting a VPN on it! There's a particular new software stack I've been looking
at to do exactly this, called [Wireguard](https://www.wireguard.com/). It's a
modern, Ethernet-over-UDP VPN protocol that uses up-to-date standards and is
very lightweight. It also has clients for a bunch of platforms, and looks like
a solid option.

#### Installing Wireguard

This is mostly a matter of following Wireguard's instructions; adding the COPR
for Fedora, and installing `wireguard-dkms` and `wireguard-tools`. Next up is
creating a private key and public key for the server; I'm using the suggested
`/etc/wireguard` to store my configurations.

Note: I had not rebooted my router recently, and had pulled down a mismatched
copy of the kernel headers compared to the actual kernel; the headers were for
a newer patch level, and so after running `dnf update` & `shutdown -r`, I was
able to continue.

#### Setting up the VPN

My first impression of Wireguard is that it is still not an approachable tool.
There's a lot of sharp edges, and the software relies on its users having broad
technical backgrounds.
