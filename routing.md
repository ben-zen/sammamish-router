# Routing configuration

This network will be routed using both IPv4 and IPv6.

## NAT for IPv4

`nft add table nat`
`nft add chain nat prerouting { type nat hook prerouting priority 0 \; }`
`nft add chain nat postrouting { type nat hook postrouting priority 100 \; }`

### Source NAT

`nft add rule nat postrouting ip saddr 10.0.0.0/8 oif eth0 snat $(ipv4_addr)`

### Destination NAT for a web server

`nft add rule nat prerouting iif eth0 tcp dport { 80, 443 } dnat $(ipv4_addr)`

## IP address filtering (IPv4)

`nft add table ip filter`

## IPv6

`conf/all/forwarding` should be set to 1

### Router Advertisements & Solicitations

- Turn on RAs for eno2-4, since those are LAN ports
- Turn on RSes for eno1, since that's the WAN port
