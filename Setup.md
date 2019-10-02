# Setting up the router

Different guides around the internet have vastly different suggestions on how
to handle routing, configuring interfaces, and DNS and DHCP; after trying out
NetworkManager for handling interfaces, it became clear that this was not going
to end well, and instead I moved to using systemd's networkd directly.

## Essential services

### SSH

You gotta use SSH. It's the way to talk to a server! Since it's also 2019, we
can use newer standards than RSA-4096 for our keys. Get with the times and use
some elliptic curves! Use Curve25519, it's fine, really. You too can have your
public key on a single line, and read it too.

My specific tweaks to the sshd configuration are as follows:

- Don't use passwords to log in. Only allow authorized keys!
- Don't allow root logins, at all.
- Re-order the checked host keys to put ed25519 as the first key.
- Have your users all use ed25519 where possible. I guess RSA is maybe fine.

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

### DNS

#### dnsmasq DNS configuration changes

Again, there are specific changes to set to make sure dnsmasq is correctly set
up. Pick DNS servers you trust and are useful to you!

- `server=8.8.8.8`, `server=1.1.1.1`: Pick a few public DNS servers to use.
- `no-resolv`: Don't read `/etc/resolv.conf`; I wanted this config file only.
- `address=/router.lan/10.0.1.1`: Add your router's internal domain name and
  ip address, since the router is not going to use DHCP to assign its address,
  and so can't get added to the database automatically.
- `rebind-domain-ok=/plex.direct/`: I have a Plex server behind this firewall,
  and since I've got a symmetric gigabit connection, I want to be able to pull
  streaming content directly from my home network, and the best way to do that
  is port forwarding for IPv4. IPv6 ... doesn't appear to be an issue yet. To
  properly stream directly from that server, however, dnsmasq needs to allow
  rebinding specifically for Plex. (This is maybe unnecessary?)

I also took a chance to make sure I've mitigated a nasty vuln; search CERT in
the config file to learn more.

## Traffic routing

Set up the IPForward rule for all interfaces you want traffic to cross, and
establish any particular routing operations you want to forward; for example,
if there's an internal server that specifically handles HTTP/S or SSH, you can
set that in firewalld.

### Firewall rules

All of the instructions in this section were run with `sudo`, because the rules
for accessing the firewall require root privileges. Additionally, these rules
should all be applied permanently, as well as set for the running instance. You
can either run the commands twice, or run them all with the `--permanent` flag,
then restart firewalld to apply the configuration.

#### Assigning nftables zones

FirewallD on this machine defaults to the FedoraServer zone, which is not very
useful for us. You can set the default if that's useful.

```shell
# Add other interfaces for the internal-facing network here.
firewall-cmd --zone=internal --add-interface=eno2
firewall-cmd --zone=internal --add-interface=docker0 # Yes, docker...
firewall-cmd --zone=external --add-interface=eno1
# Traffic routed out of this server should use this server's IPv4 address.
firewall-cmd --add-masquerade
# Ensure the server is reachable for SSH both inside and outside the network.
firewall-cmd --zone=internal --add-service=ssh
firewall-cmd --zone=external --add-service=ssh
# Let the server handle DHCP and DNS requests - inside the network.
firewall-cmd --zone=internal --add-service=dhcp
firewall-cmd --zone=internal --add-service=dns
```

#### Plex port forwarding

To make sure my local plex server can directly route traffic, we're going to
forward Plex's preferred port, 32400, to my system. In the event of multiple
Plex servers on one network, you'll need a separate port for each server that
is looking to publically route data.

```shell
firewall-cmd --zone=external --add-forward-port=port=32400:proto=tcp:toaddr=192.168.2.3
firewall-cmd --zone=external --add-forward-port=port=32400:proto=udp:toaddr=192.168.2.3
```

#### Other servers and services

Currently, I don't have other services that need specific routing; I don't yet
have plans for a web server or other locally hosted service, but those will be
filled in here as they come online.

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

WireGuard installation is currently ongoing.

### LDAP server

Not yet planned out fully.

### Logging

I'd love to run a logging utility to study my network in a bit more detail

### Controlling the fit-statUSB

There's a small RGB LED USB (can we get a few more 3-letter initialisms?) on
the front of the case because I wanted some sort of blinkenlight. Currently it
pulses between dimmer red and brigher red, because that's what I set it to do,
but ideally it will provide some sort of status update for me based on activity
or system status.

### DDNS updating

I like being able to use a name to connect to my home network, versus trying to
remember my IP address that one time. So, I have DDNS set up-but I currently
manually update my IP every time it changes. It seems changing out how DHCP was
run caused a re-assignment of my home IP, so I had a little fun one afternoon,
which really just showed why I need to add this service. I'm actively searching
for a tool for this purpose.
