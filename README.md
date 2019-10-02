# Sammamish Router

This is the new router for my apartment, a massively overbuilt system based on
a Supermicro server motherboard.

## Functions

There's the core features of a router that I want to support, combined with
some tools that I've wanted to explore for years, namely a VPN, and an LDAP
server. There are also some options I'd like to explore, to do with adding
essentially a NAS, and setting up Let's Encrypt; those will probably be done in
the opposite order.

### Main functions

- Firewall
- DHCP
- DNS
- WireGuard VPN gateway
- LDAP server

### Nice to have

- NAS?
- Let's Encrypt?

## Build

### Case

Supermicro Superchassis CSE-505-203B

- 1U rackmount chassis
- 200W power supply
- Micro-ITX mount

Extra/replacement faceplate: Supermicro MCP-260-00085-0B
Has all the correct ports instead of the one the above ships with.

### Motherboard and CPU

Supermicro MBD-M11SDV-4CT-LN4F-O

- MiniITX formfactor
- AMD EPYC 3101 SOC
- Intel i350 NIC with 4 gigabit ethernet ports
- PCIe 3.0 x16 slot

### RAM

- NEMIX 32GB DDR4-2666 Registered ECC (2 x 16GB)

### Storage

Intel 660p Series SSD

- PCIe NVMe 3.0 x4
- M.2 2280
- 1 Terabyte of storage, for some unexplained reason

### Additional components

- 3x Noctua NF-A4x20
  - Max airflow 9.4m^3/h
  - 40x40mm, 20mm deep
- Supermicro AOM-TPM-9665V
  - TPM 2.0
- Supermicro MCP-220-0004-0N riser
  - Holds 2 2.5" drives in a 3.5" bay location.

## Operating system and software

### Operating system

Multiple options are possible, mostly RedHat derived.

- Fedora 30
- CentOS 8 (not yet released)

In light of the availability of one, but not the other: Fedora 30 it is. This
is far from actually a surprise, and if updates end up being tedious, CentOS
should have shipped at that point.

### Software

#### nftables firewall

As a friend put it, "Well, I have this old setup with iptables and whatnot...
but you, you're building this fresh and new. So *you're* going to try out the
new stuff, like _nftables_."

#### DNSMasq (DNS, DHCP, RAs for IPv6)

I want DNS, I need DHCP for IPv4, and I may need specific limited features of
DHCPv6, such as supplying addresses of DNS servers and the like. DNSMasq has
all of the abilities we're looking for.

#### WireGuard VPN

My concerns about VPNs are generally about sketchy public wifi access points,
and less so about my internet activity being tracked. As such, I think running
a VPN to my home network is reasonable.

#### FreeIPA (which includes 389 Directory Server)

With a small posse of ThinkPads all running essentially the same configuration,
it would be nice to have one login across all of them. With an LDAP server on
the network's gateway, that could be done. Also, I could configure Kerberos for
a reason I do not yet understand. So probably not.

#### Let's Encrypt, possibly with custom code

It would be useful to be able to get key signing abilities established, such as
being able to RDP into a machine without the certificate warning popping up and
interrupting me.

## Build notes

### Case oddities

The case came with a faceplate, but it's one for an older layout of Supermicro
I/O, and as such doesn't work with the new layouts. The normal I/O shield also
doesn't fit, so I'll need to order the correct faceplate.

The PSU came with a full 20-pin ATX power connector, even though this board's
pinout only requires 4 of the pins; I'll correct for this by pulling pins from
the ATX connector and attaching them to the 4-pin piece, then cutting down the
remaining ATX pins.

There is a fan mount adapter that I didn't get, but I think I'll 3d print one,
and add a carbon filter slot behind it.
