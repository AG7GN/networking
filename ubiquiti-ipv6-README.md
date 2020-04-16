# Configuring Ubiquiti Routers to manage an IPv6 prefix delegation from Comcast

VERSION: 20200415
AUTHOR:  Steve Magnuson, AG7GN

# Prerequisites

- Internet service from Comcast
- a Ubiquiti EdgeRouter, such as ER-X, ERLite-3, ER-4, etc.
- You are using the router's DNS service (`dnsmasq`) on your LAN

## Scenario

- `eth0` is the Internet facing interface on the router
- `eth1` is the LAN interface on the router

SLAAC (Stateless Address Autoconfiguration) is used to assign addresses to clients.
DHCPv6 is used to assign the IPv6 DNS server address to clients.

1. Establish a CLI connection to your router.

1. Configure `dhcpv6-pd` (DHCP for IPv6 with Prefix Delegation feature) to request a prefix delegation of length 60 from Comcast on `eth0`:

		set interfaces ethernet eth0 dhcpv6-pd pd 0 prefix-length 60
		set interfaces ethernet eth0 dhcpv6-pd rapid-commit enable

1. On the LAN interface, enable `slaac`, set the LAN interface host address to ::1, set up a prefix-id and disable DNS (because we'll use `dhcpv6-server` to assign the IPv6 DNS server address):

		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth1 host-address '::1'
		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth1 no-dns
		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth1 prefix-id ':1'
		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth1 service slaac

1. Configure RA (router advertisement) parameters on the LAN interface.  The `other-config-flag` tells the host to use SLAAC to get the IPv6 address and DHCPv6 to get other parameters.  The LAN interface will get a `/64` although our delegation was `/60`.  The other `/64` subnets in that `/60` can be assigned to other LAN interfaces. A `/60` contains 16 `/64` subnets.

		set interfaces ethernet eth1 ipv6 dup-addr-detect-transmits 1
		set interfaces ethernet eth1 ipv6 router-advert cur-hop-limit 64
		set interfaces ethernet eth1 ipv6 router-advert link-mtu 0
		set interfaces ethernet eth1 ipv6 router-advert managed-flag false
		set interfaces ethernet eth1 ipv6 router-advert max-interval 600
		set interfaces ethernet eth1 ipv6 router-advert other-config-flag true
		set interfaces ethernet eth1 ipv6 router-advert prefix '::/64' autonomous-flag true
		set interfaces ethernet eth1 ipv6 router-advert prefix '::/64' on-link-flag true
		set interfaces ethernet eth1 ipv6 router-advert prefix '::/64' preferred-lifetime 14400
		set interfaces ethernet eth1 ipv6 router-advert prefix '::/64' valid-lifetime 86400
		set interfaces ethernet eth1 ipv6 router-advert reachable-time 0
		set interfaces ethernet eth1 ipv6 router-advert retrans-timer 0
		set interfaces ethernet eth1 ipv6 router-advert send-advert true

1. Add a secondary IPv6 local-scope address to the LAN interface.  

		set interfaces ethernet eth1 address 'fd00::1/48'

1. Configure the `dhcpv6-server`.  The subnet is required by dhcpv6 and is used to determine on which interfaces the dhcpv6 server will listen.  Assigning a secondary local address to the interface (previous step) and using that as the required subnet value makes the configuration portable. Otherwise, you need to put the "scope link" address of the LAN interface with a netmask of `/128` instead of `/64` as the subnet.

		set service dhcpv6-server shared-network-name DHCPV6_STATELESS name-server 'fd00::1'
		set service dhcpv6-server shared-network-name DHCPV6_STATELESS subnet 'fd00::1/128'
		
1. For each additional LAN interface, repeat the `slaac` configuration above, except increase the `prefix-id` by one.  

	Example:  Say we have another LAN interface `eth2`.  The `slaac` configuration would be:

		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth2 host-address '::1'
		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth2 no-dns
		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth2 prefix-id ':2'
		set interfaces ethernet eth0 dhcpv6-pd pd 0 interface eth2 service slaac

	Add the RA commands for `eth2` as well:

		set interfaces ethernet eth2 ipv6 dup-addr-detect-transmits 1
		set interfaces ethernet eth2 ipv6 router-advert cur-hop-limit 64
		set interfaces ethernet eth2 ipv6 router-advert link-mtu 0
		set interfaces ethernet eth2 ipv6 router-advert managed-flag false
		set interfaces ethernet eth2 ipv6 router-advert max-interval 600
		set interfaces ethernet eth2 ipv6 router-advert other-config-flag true
		set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' autonomous-flag true
		set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' on-link-flag true
		set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' preferred-lifetime 14400
		set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' valid-lifetime 86400
		set interfaces ethernet eth2 ipv6 router-advert reachable-time 0
		set interfaces ethernet eth2 ipv6 router-advert retrans-timer 0
		set interfaces ethernet eth2 ipv6 router-advert send-advert true
		
	Finally, include the local scope IP address subnet in the `dhcpv6-server` configuration.  To find out what it is, run:

		ip addr show dev eth2 | grep "scope link"

	Example:

		steve@gateway:~$ ip addr show dev eth2 | grep "scope link"
	    inet6 fe80::e236:daff:fe28:9694/64 scope link 

	Then change the mask from `/64` to `/128` and add it to the `dhcpv6-server` configuration:

		set service dhcpv6-server shared-network-name DHCPV6_STATELESS subnet 'fe80::e236:daff:fe28:9694/128'
		
1. Finish

		commit;save
		
		
