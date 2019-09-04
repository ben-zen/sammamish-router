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

### Motherboard and CPU

Supermicro MBD-M11SDV-4CT-LN4F-O

- MiniITX formfactor
- AMD EPYC 3101 SOC
- Intel i350 NIC with 4 gigabit ethernet ports

### RAM

- NEMIX 32GB DDR4-2666 Registered ECC (2 x 16GB)

### Storage

Intel 660p Series SSD

- PCIe NVMe 3.0 x4
- M.2 2280
- 500GB is more than enough space

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

### Software

#### nftables firewall

As a friend put it, "Well, I have this old setup with iptables and whatnot...
but you, you're building this fresh and new. So *you're* going to try out the
new stuff, like _nftables_."

#### DNSMasq (DNS, DHCP, RAs for IPv6)

#### WireGuard VPN

My concerns about VPNs are generally about sketchy public wifi access points,
and less so about my internet activity being tracked. As such, I think running
a VPN to my home network is reasonable.

#### 389 Directory Server (as a Red Hat library?)

With a small posse of ThinkPads all running essentially the same configuration,
it would be nice to have one login across all of them. With an LDAP server on
the network's gateway, that could be done.

#### Let's Encrypt, possibly with custom code

It would be useful to be able to get key signing abilities established, such as
being able to RDP into a machine without the certificate warning popping up and
interrupting me.
