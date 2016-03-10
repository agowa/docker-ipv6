# Docker IPv6
This respository contains various scripts and tools to enable Docker to
use a IPv6 prefix obtained through DHCPv6 Prefix Delegation.

This allows containers to use native IPv6 without any need for proxies.

The scripts and tooling in this repository were tested on Ubuntu 14.04

# Prefix Delegation
Prefix Delegation (PD) is a mechanism which allows a client to request a
IPv6 prefix to be routed towards that host.

Usually a client will obtain a prefix larger than a /64 to allow splitting
up this prefix in multiple smaller prefixes.

# Docker and IPv6
Docker can use IPv6 for it's containers if you start the docker daemon with
the --ipv6 flag.

For example:

```docker daemon --ipv6 --fixed-cidr-v6=2001:db8:100::/80```

IPv6 addresses of Docker containers are based on the MAC address. A /80 subnet
combined with the 48-bits of a MAC address sums op to 128-bits for a full IPv6
address.

# dhclient
dhclient is part of ISC DHCP and can request a IPv6 Prefix through DHCPv6.

In order to do so, dhclient should be run with these flags:

```dhclient -6 -P eth0```

It will request a prefix through DHCPv6.

## Upstart
Under Ubuntu 14.04 you can run dhclient using upstart (in the future systemd) to
request the prefix and renew then needed.

See the upstart file in the repository to do so.

## Docker IPv6 hook
The 'docker-ipv6' dhclient hook in this repository should be placed in
/etc/dhcp/dhclient-enter-hooks.d/ where it will be executed after dhclient
obtains a prefix.

The hook will get the first /80 subnet out of the delegated prefix and
write it to /etc/docker/ipv6.prefix

The Docker daemon is then restarted so it will use the new subnet as the fixed
IPv6 cidr.

In order for this to work the DOCKER_OPTS in /etc/default/docker should be set to:

```DOCKER_OPTS="--ipv6 --fixed-cidr-v6=`cat /etc/docker/ipv6.prefix`"```
