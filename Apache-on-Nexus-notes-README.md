# Notes on Running Apache on the Nexus DR-X Image

VERSION 20200817

AUTHOR Steve Magnuson, AG7GN

The Nexus DR-X configures `/var/log/` as a [tmpfs](https://wiki.archlinux.org/index.php/Tmpfs), so it exists in memory rather than on the micro SD card. This is done to prolong the life of the micro SD card by reducing the number of "writes" to the card. The disadvantage of tmpfs is that it (and `/var/log`) is removed when the pi is powered off or rebooted. Some programs, like Apache, expect that certain folders exist or the program won't start.

Here's how to fix that problem for Apache (this assumes Apache is installed):

1. As root, edit `/lib/systemd/system/apache2.service`. The file looks something like this:

		[Unit]
		Description=The Apache HTTP Server
		After=network.target remote-fs.target nss-lookup.target
		
		[Service]
		Type=forking
		Environment=APACHE_STARTED_BY_SYSTEMD=true
		ExecStart=/usr/sbin/apachectl start
		ExecStop=/usr/sbin/apachectl stop
		ExecReload=/usr/sbin/apachectl graceful
		PrivateTmp=true
		Restart=on-abort

		[Install]
		WantedBy=multi-user.target

1. Locate the `Environment=...` line in the `[Service]` section. After that line, add these lines:

		PermissionsStartOnly=true
		ExecStartPre=-/bin/mkdir /var/log/apache2
		ExecStartPre=-/bin/chmod 750 /var/log/apache2
		ExecStartPre=-/bin/chown root:adm /var/log/apache2

1.	Save the file and exit the editor.

1. Run these commands in the Terminal:

		sudo systemctl daemon-reload
		sudo systemctl restart apache2

1.	Reboot the Pi and verify that Apache has started.
