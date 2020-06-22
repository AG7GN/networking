# Notes on Customizing AREDN

VERSION 20200621

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
		
## AREDN OLSR "Under the Hood"

### OLSR State JSON Output

[All Services Available on the Mesh](http://localnode.local.mesh/cgi-bin/sysinfo.json?services=1)

[Local Node's Services](http://localnode.local.mesh/cgi-bin/sysinfo.json?services_local=1)

[Nodes on the Mesh](http://localnode.local.mesh:8080/cgi-bin/sysinfo.json?hosts=1)

[Routes](http://localnode.local.mesh:9090/routes)

[OLSR State](http://localnode.local.mesh:9090/all)

### Services Cache

You might notice that other nodes have services listed in their mesh status that your node advertised at one time, but that you have since removed or changed.  [This thread](https://www.arednmesh.org/content/services-not-being-deleted) explains what is going on.  Excerpt:

"...OLSR uses a timeout to clear the cache on all nodes of a service that is no longer advertising itself.  As long as a node is alive on the mesh, the caches will not clear.  If the node is off the mesh for a period of about 10 mins (ie. not advertising itself to all other nodes), then, other nodes realize that the node (and all of it's services) has gone offline and removes all entries for that node (including services)."

So, to force all other nodes on the mesh to flush the list of services that they learned from your node, you must remove your node from the mesh for 10+ minutes.

Of course, as an alternative, you could contact the owners of all the other nodes and have them reboot their nodes to flush their cache, but that would be a lot of work!







