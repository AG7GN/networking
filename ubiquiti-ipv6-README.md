# Configuring Ubiquiti EdgeRouters to manage an IPv6 prefix delegation from Comcast

VERSION: 20230927

AUTHOR:  Steve Magnuson, AG7GN

# Prerequisites

- Internet service from Comcast
- a Ubiquiti EdgeRouter, such as ER-X, ERLite-3, ER-4, etc.
- You are using the router's DNS service (`dnsmasq`) on your LAN
- `eth3` is the Internet facing interface on the router
- `eth1` is the LAN interface on the router
- In both scenarios described below, you are starting with a 'wizard' configured default configuration, meaning only very basic IPv6 support and DHCPv4 provided by the EdgeRouter's built-in DHCP server and not `dnsmasq`.

## Scenario A (Less Preferred)

- SLAAC ([Stateless Address Autoconfiguration](https://howdoesinternetwork.com/2013/slaac)) is used to assign addresses to clients.
- [DHCPv6](https://en.wikipedia.org/wiki/DHCPv6) is used to assign the IPv6 DNS server (only) address to clients.
- Internet facing interface has a global IPv6 address

1. Configure `dhcpv6-pd` (DHCP for IPv6 with Prefix Delegation feature) to request a prefix delegation of length 60 from Comcast on `eth3`:

		set interfaces ethernet eth3 dhcpv6-pd pd 0 prefix-length 60
		set interfaces ethernet eth3 dhcpv6-pd rapid-commit enable

1. On the LAN interface, enable `slaac`, set the LAN interface host address to ::1, set up a prefix-id and disable DNS (because we'll use `dhcpv6-server` to assign the IPv6 DNS server address):

		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 prefix-id ':1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 service slaac

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

		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 prefix-id ':2'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 service slaac

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
		

## Scenario B (Preferred)

- SLAAC ([Stateless Address Autoconfiguration](https://howdoesinternetwork.com/2013/slaac)) *and* DHCPv6 is used to assign addresses to clients.  All IPv6 hosts support SLAAC. For those client hosts that also support DHCPv6 (most of them), this results in 2 IPv6 address assignments. The SLAAC assignment is made using the EUI-64 process. The other address is assigned via DHCPv6.
- The Internet facing interface does not have a global IPv6 address. Having an external IPv6 address is only needed if the router itself needs to be reachable via IPv6 from anywhere on the Internet. Not having one means one less way for hackers to access your router.

	The advantage to Scenario B is that the hostname learned by the DHCP server via the DHCPv4 assignment to a client is that the DHCPv6 process uses that same name, so you can reach the host using the same name to either it's IPv4 or IPv6 address. It all happens automatically. However, the name does not work for MacOS 11 and Apple IOS devices for IPv6. It works fine for IPv4, though. Don't know why. Linux and Windows hosts work as expected.

1. Configure the external facing interface to request a [prefix delegation](https://en.wikipedia.org/wiki/Prefix_delegation) from the ISP

		set interfaces ethernet eth3 dhcpv6-pd prefix-only
		set interfaces ethernet eth3 dhcpv6-pd rapid-commit enable
		set interfaces ethernet eth3 dhcpv6-pd no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 prefix-length 60

	`dhcpv6-pd` is the prefix delegation configuration.
	`prefix-only` means don't assign global IPv6 address to the Internet facing `eth3`, we only want a prefix delegation that we can use on the internal networks.
	`no-dns` means the router itself won't use the ISP's IPv6 DNS servers. I prefer Cloud Flare's DNS servers and those will be configured later.
	`prefix-length 60` the size of the delegation (the number of network bits in the mask). Comcast allocates 60 bit prefixes.
	
1. Allocate subnets from the prefix delegation to inside network interfaces

	I have 5 inside interfaces that I want to support IPv6: `eth1, eth1.2, eth1.30, eth1.43, eth1.170, eth2`.
	
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1 prefix-id ':1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.2 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.2 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.2 prefix-id ':2'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.30 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.30 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.30 prefix-id ':3'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.43 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.43 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.43 prefix-id ':4'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.170 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.170 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth1.170 prefix-id ':6'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 host-address '::1'
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 no-dns
		set interfaces ethernet eth3 dhcpv6-pd pd 0 interface eth2 prefix-id ':5'

	`host-address ::1` means that that interface will be assigned IPv6 address `<prefix>::1`.
	`no-dns` means that the ISP's DNS servers will not be assigned via SLAAC to hosts on that inside interface.
	`prefix-id ::x` is the prefix assigned to that inside interface. So, say Comcast delegates this prefix to me:
	
		2601:face:cafe:4c80::/60
		
	I can allocate these 16 `/64` subnets out of that prefix:
	
		2601:face:cafe:4c80::/64
		2601:face:cafe:4c81::/64
		2601:face:cafe:4c82::/64
		2601:face:cafe:4c83::/64
		2601:face:cafe:4c84::/64
		2601:face:cafe:4c85::/64
		2601:face:cafe:4c86::/64
		2601:face:cafe:4c87::/64
		2601:face:cafe:4c88::/64
		2601:face:cafe:4c89::/64
		2601:face:cafe:4c8a::/64
		2601:face:cafe:4c8b::/64
		2601:face:cafe:4c8c::/64
		2601:face:cafe:4c8d::/64
		2601:face:cafe:4c8e::/64
		2601:face:cafe:4c8f::/64

	So, `prefix-id ':1'` is the second subnet and I've assigned it to interface `eth1`. Interface `eth1` now has this IP address:
	
		2601:face:cafe:4c81::1/64
	
	and interface `eth1.2` has this IPv6 address:
	
		2601:face:cafe:4c82::1/64	

	etcetera.
	
1. Disable the regular EdgeRouter DHCP server and use `dnsmasq` for DHCP (IPv4 and IPv6) instead

	The standard DHCP server in the EdgeRouter doesn't support all the features we need for IPv6, so we use the `dnsmasq` DHCP servers instead.
	
		delete service dhcp-server
		delete service dhcpv6-server

1. Set up the `dnsmasq` DNS server

	Have the router use `dnsmasq` for it's own DNS and set cache size: 
	
		set service dns forwarding system
		set service dns forwarding cache-size 2048
		
	Tell `dnsmasq` what ports to listen on (inside ports only):
	
		set service dns forwarding listen-on eth1
		set service dns forwarding listen-on eth1.2
		set service dns forwarding listen-on eth1.30
		set service dns forwarding listen-on eth1.43
		set service dns forwarding listen-on vtun0
		set service dns forwarding listen-on eth1.170
		set service dns forwarding listen-on vtun2
		set service dns forwarding listen-on eth2

	Tell `dnsmasq` to use Cloud Flare's IPv4 and IPv6 DNS servers for external name resolution:
	
		set service dns forwarding name-server 1.1.1.1
		set service dns forwarding name-server 1.0.0.1
		set service dns forwarding name-server '2606:4700:4700::1111'
		set service dns forwarding name-server '2606:4700:4700::1001'

	Set some [options](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html):
	
		set service dns forwarding options bogus-priv
		set service dns forwarding options domain-needed
		set service dns forwarding options expand-hosts
		set service dns forwarding options localise-queries
		set service dns forwarding options strict-order
		set service dns forwarding options no-resolv
		set service dns forwarding options selfmx
		set service dns forwarding options filterwin2k
		set service dns forwarding options dns-forward-max=200
		set service dns forwarding options server=/local/

	Set the domain globally and per-interface (they may be different - the per subnet domain designations here are redundant because the domain is the same in all cases):
	
		set service dns forwarding options domain=mydomain.org
		set service dns forwarding options domain=mydomain.org,192.168.1.0/24,local
		set service dns forwarding options domain=mydomain.org,192.168.2.0/24,local
		set service dns forwarding options domain=mydomain.org,192.168.170.0/24,local

	Send requests for names ending in `.local.mesh` to the AREDN router's name server:
	
		set service dns forwarding options server=/local.mesh/10.27.190.1
		
1. Set up the `dnsmasq` DHCP server

	Designate `dnsmasq` as the DHCP server:
	
		set service dns forwarding options dhcp-authoritative
		set service dns forwarding options dhcp-fqdn
		
	Enable additional DHCP event logging to `/var/log/dnsmaq.log`:
	
		set service dns forwarding options log-dhcp
		
	Tell `dnsmasq` to send router advertisements:
	
		set service dns forwarding options enable-ra

	Set IPv4 DHCP options and scopes:

		set service dns forwarding options 'dhcp-option=option:router,0.0.0.0'
		set service dns forwarding options 'dhcp-option=option:dns-server,0.0.0.0'
		set service dns forwarding options 'dhcp-option=option:domain-search,mydomain.org'
		set service dns forwarding options 'dhcp-range=interface:eth1.2,192.168.2.100,192.168.2.200,24h'
		set service dns forwarding options 'dhcp-range=interface:eth1.30,172.20.30.100,172.20.30.200,24h'
		set service dns forwarding options 'dhcp-range=interface:eth1.43,192.168.43.100,192.168.43.200,24h'
		set service dns forwarding options 'dhcp-range=interface:eth1,192.168.1.2,192.168.1.200,24h'
		set service dns forwarding options 'dhcp-range=interface:eth1.170,192.168.170.50,192.168.170.200,24h'
		set service dns forwarding options 'dhcp-range=interface:eth2,172.20.20.100,172.20.20.200,24h'

	Set IPv6 DHCP options and scopes. The `dns-server` setting of `[::]` will cause the DHCP server to send the router's IP address (which is the address the router's DNS server is listening on) to the client.
	
		set service dns forwarding options 'dhcp-option=option6:domain-search,mydomain.org'
		set service dns forwarding options 'dhcp-option=option6:dns-server,[::]'
		set service dns forwarding options 'dhcp-range=::1000,::FFFF,constructor:eth1,ra-names,slaac,64,24h'
		set service dns forwarding options 'dhcp-range=::1000,::FFFF,constructor:eth1.2,ra-names,slaac,64,24h'
		set service dns forwarding options 'dhcp-range=::1000,::FFFF,constructor:eth1.30,ra-names,slaac,64,24h'
		set service dns forwarding options 'dhcp-range=::1000,::FFFF,constructor:eth1.43,ra-names,slaac,64,24h'
		set service dns forwarding options 'dhcp-range=::1000,::FFFF,constructor:eth1.170,ra-names,slaac,64,24h'

	The `dhcp-range` is defined as follows:
	
	`::1000,::FFFF` the scope for this subnet. These are the start and end IP addresses within that subnet.
	`constructor:eth1,ra-names,slaac,64,24`
	`eth1` The remainder of the line applies to the DHCPv6 server behavior on `eth1`.
	
	`ra-names` means gives DNS names to dual-stack hosts which do SLAAC for IPv6. Dnsmasq uses the host's IPv4 lease to derive the name, network segment and MAC address and assumes that the host will also have an IPv6 address calculated using the SLAAC algorithm, on the same network segment. The address is pinged, and if a reply is received, an AAAA record is added to the DNS for this IPv6 address.
	
	`slaac` tells `dnsmasq` to offer Router Advertisement on this subnet and to set the A bit in the router advertisement, so that the client will use SLAAC addresses. When used with a DHCP range or static DHCP address this results in the client having both a DHCP-assigned and a SLAAC address.
	
	`64` is the prefix length for this subnet
	
	`24h` is the DHCPv6 lease duration (24 hours)
	

1. Set up `dnsmasq` lease reservations for certain hosts

	`dhcp-host` guarantees that certain hosts will always be assigned the same IP address. You can specify both the IPv4 and the IPv6 address with one `dhcp-host` command. Note that you only need to specify the host part of the IPv6 address. `dnsmasq` will automatically insert the correct IPv6 prefix. 
	
		set service dns forwarding options 'dhcp-host=b8:27:eb:48:d3:6f,192.168.170.100'
		set service dns forwarding options 'dhcp-host=00:1e:fd:01:a3:f5,192.168.170.50'
		set service dns forwarding options 'dhcp-host=e4:95:6e:41:bb:e0,172.20.20.100'
		set service dns forwarding options 'dhcp-host=f0:18:98:ec:6d:40,192.168.1.10,[::ceb2]'
		set service dns forwarding options 'dhcp-host=a8:20:66:16:0e:12,192.168.1.4,[::de86]'
		set service dns forwarding options 'dhcp-host=buildpi,192.168.1.200,[::c245]'

1. Remove all DHCP-learned IP addresses/hosts from `/etc/hosts`

	Open `/etc/hosts` in an editor and delete the lines that contain "learned via DHCP"
	
	Restart `dnsmasq`:
	
		sudo systemctl restart dnsmasq

1. Check `dnsmasq` lease file

		cat /var/lib/misc/dnsmasq.leases
		
	As a shortcut, add this line to your `~/.bashrc` file for your account and `root`'s account, too:
	
		alias leases="cat /var/lib/misc/dnsmasq.leases"
		
	Then, you can just run `leases` to see a list of the leases.
	
## Lease File Format

The format of the `dnsmasq.leases` file is as follows (from this [dnsmasq lease file format](http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2016q2/010595.html) thread):

### A DHCPv4 lease entry consists of these fields separated by spaces:

- The __expiration time__ (seconds since unix epoch) or duration
  (if dnsmasq is compiled with HAVE_BROKEN_RTC) of the lease.
  0 means infinite.

- The __link address__, in format XX-YY:YY:YY[...], where XX is the ARP
  hardware type.  "XX-" may be omitted for Ethernet.

- The __IPv4 address__

- The __hostname__ (sent by the client or assigned by dnsmasq)
  or '*' for none.

- The __client identifier__ (colon-separated hex bytes)
  or '*' for none.

### A DHCPv6 lease entry has these fields separated by spaces:

- The __expiration time__ or duration

- The __IAID__ ([Identity Association Identifier](https://thirdinternet.com/dhcpv6/)) as a Big Endian decimal number, prefixed by T for IA_TAs (temporary addresses).

- The __IPv6 address__

- The __hostname__ or '*'

- The __client DUID__ ([DHCP Unique Identifier](https://en.wikipedia.org/wiki/DHCPv6), consisting of colon-separated hex bytes) or '*' if unknown. This field seems to be written but is never read back. A bug maybe ?

For DHCPv6, there must also be exactly one special entry indicating
the DUID of the server.  This line contains two fields:

- The __string "duid"__.

- The __DUID of the server__.


