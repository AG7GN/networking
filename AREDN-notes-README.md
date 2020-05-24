# Notes on Customizing AREDN

VERSION 20200419

AUTHOR Steve Magnuson, AG7GN

## Adding hostnames to the DNS configuration on an AREDN node

Note that names you add in this way will only be resolvable by your local node, not by other nodes on the mesh.  I have a submitted an [enhancement request](https://github.com/aredn/aredn_ar71xx/issues/516) to the AREDN team to add this feature so it'll work on the entire mesh.

1. SSH to your AREDN node (TCP port 2222, remember).

1. Add your hostnames to `/etc/hosts.user`.  Example:

		10.27.190.3	w7ecg-meet
		10.27.190.3	w7ecg-info
		10.27.190.3	w7ecg-help
		
1. Add this line to `/etc/dnsmasq.conf`:

		addn-hosts=/etc/hosts.user
		
1. Restart `dnsmasq`:

		service dnsmasq restart
		



