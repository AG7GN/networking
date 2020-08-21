# Notes on Installing and Operating a TFTP server on the Nexus

VERSION 20200821

AUTHOR Steve Magnuson, AG7GN

1. Install the latest __hampi-iptables__ rules.

	- Click __Raspberry > Hamradio > Update Pi and Ham Software__.
	- Select __hampi-iptables__, click __OK__.

1. Install the TFTP server. Open a Terminal, then run:
	
		sudo apt update
		sudo apt -y install tftpd-hpa atftp
		sudo mkdir -p /srv/tftp
		sudo chown nobody:nogroup /srv/tftp
		sudo chmod 777 /srv/tftp
		sudo cp /etc/default/tftpd-hpa /etc/default/tftpd-hpa.original

1. As root (`sudo`), open `/etc/default/tftpd-hpa` in an editor and change it so it looks like this:

		# /etc/default/tftpd-hpa

		RUN_DAEMON="yes"
		TFTP_USERNAME="nobody"
		TFTP_DIRECTORY="/srv/tftp"
		TFTP_ADDRESS=":69"
		TFTP_OPTIONS="--secure -4 -p -c"
		
	__EXAMPLE__: From the Terminal, run `sudo leafpad /etc/default/tftpd-hpa`, then make the above changes.
	
	Save the file and close the editor.

1.	Restart the TFTP server. In the Terminal, run:

		sudo systemctl restart tftpd-hpa.service
		
1.	Copy files you want to serve to tftp clients into `/srv/tftp`

	- The files you serve must be readable by all.
	- Files that are "put" to the tftp server will also be in `/srv/tftp`.
	
1. The TFTP server will automatically start when the Pi is rebooted.

1. __If the TFTP *CLIENT* is Linux__ and running `iptables`, run these commands on the client to enable TFTP connection tracking (or disable the firewall - not recommended)

		sudo modprobe nf_conntrack_tftp
		sudo iptables -t raw -I OUTPUT -j CT -p udp -m udp --dport 69 --helper tftp

	- Needed because the TFTP server's SOURCE port changes for return packets (it's not 69).
	
