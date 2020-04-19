# Notes on Configuring a Private Jitsi Server

VERSION 20200419

AUTHOR Steve Magnuson, AG7GN

## Prerequisites

- Read the [Jitsi Meet Quick Install](https://github.com/jitsi/jitsi-meet/blob/master/doc/quick-install.md) guide, then read the rest of this document for some important additions/changes to the instructions provided by the Quick Install. 

## Certificate Requirements and Installation Notes

1. If deploying your own non-self-signed certificate, the certificate must have these attributes:

	- The certificate validity, the difference between "Not After" and "Not Before", must within 39 months.
	- The Common Name (CN) must be the same as the host subcomponent of the [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier).
	- The CN must ALSO be in the Subject Alternate Name (SAN) of the certificate.
	- Extended Key Usage must contain "TLS Web Server Authentication"

1. Have your certificate (public key), private key and CA certificate ready to go before starting the installation.  The installation script described in the [Jitsi Meet Quick Install](https://github.com/jitsi/jitsi-meet/blob/master/doc/quick-install.md) guide will prompt you to specify the location of the private key and certificate.  I put the private key in `/etc/ssl/private` and the cert in `/etc/ssl`.

1. Importing the CA Certificate to the server's Trust Store.

	- The CA certificate must be Base64 encoded, also known as Privacy Enhanced Mail (PEM) format.
	- Copy the certificate to `/usr/local/share/ca-certificates`.  Give it a `.crt` extension.
	- Run this command:
	
			sudo update-ca-certificates
			
		This will create a symlink in `/etc/ssl/certs` to the CA certificate in `/usr/local/share/ca-certificates`.

### Certificate Example

~~~
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4101 (0x1005)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = WA, O = Whatcom Emergency Communication Group - WECG AREDN, OU = AREDN, OU = OU1, OU = OU2, L = Seattle, CN = WECG AREDN Root, description = rootCA
        Validity
            Not Before: Apr 17 22:39:11 2020 GMT
            Not After : Apr 17 00:00:00 2022 GMT
        Subject: C = US, ST = WA, L = Seattle, O = Whatcom Emergency Communication Group - WECG AREDN, OU = AREDN, OU = OU1, OU = OU2, CN = w7ecg-web.local.mesh, description = serverCert
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b8:ca:b1:45:3a:34:4a:2d:b6:30:2a:a3:44:69:
                    ed:a2:06:80:d8:b8:72:15:1c:3b:a6:7f:38:a3:b2:
                    5a:d6:bd:d7:96:c0:c7:76:cc:c9:e1:d0:96:5b:19:
                    0e:6c:73:96:41:f9:bf:27:b1:bc:69:93:4c:e0:d6:
                    fd:0c:ca:3c:5f:23:68:1a:72:2b:da:7c:97:93:b2:
                    a9:94:17:3d:c9:b5:1b:b1:31:36:81:83:06:1b:8c:
                    d8:bf:71:55:88:59:db:5e:58:56:18:c8:df:d2:87:
                    db:f6:69:9e:6d:da:b7:2b:e6:99:e1:4f:37:27:98:
                    ce:e7:f2:92:77:10:08:45:08:07:d3:76:14:da:63:
                    cd:84:ea:3b:6d:07:b2:8f:5f:c1:df:67:54:e3:fb:
                    c5:a5:f8:07:5c:46:f2:b0:11:a8:1c:03:86:72:f4:
                    e5:3d:0b:12:68:66:a0:73:46:0b:a6:03:af:95:14:
                    54:10:dd:d0:be:60:cc:ed:48:e1:dd:02:3a:b8:03:
                    f7:85:02:8e:d5:02:e4:67:d8:29:ba:dc:ae:b2:4e:
                    1a:96:b3:be:d6:52:4d:89:09:91:a0:3f:c4:19:75:
                    0a:d8:12:49:77:3b:dd:51:b1:7a:dd:39:51:37:10:
                    ea:6d:fd:10:e7:fd:4f:84:ac:3b:d8:a3:fb:30:a7:
                    06:99
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier: 
                61:41:DB:43:D1:68:69:CF:1A:74:AC:E0:FB:41:E2:44:7A:70:A8:F6
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Key Agreement
            X509v3 Authority Key Identifier: 
                keyid:B9:E1:E3:0B:7A:7D:AE:24:3B:7E:12:BA:BA:9A:8F:AD:57:9F:91:A9
                DirName:/C=US/ST=WA/O=Whatcom Emergency Communication Group - WECG AREDN/OU=AREDN/OU=OU1/OU=OU2/L=Seattle/CN=WECG AREDN Root/description=rootCA
                serial:48:4F:84:52:E1:DF:AC:C0:CF:CE:E7:7F:DF:9D:66:AB:59:67:9D:4A

            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            Authority Information Access: 
                CA Issuers - URI:http://w7ecg-web.local.mesh:8080/crl/rootCA.WECGAREDN-01tier-sha256.adelie-cert.pem

            X509v3 CRL Distribution Points: 

                Full Name:
                  URI:http://w7ecg-web.local.mesh:8080/crl/rootCA.WECGAREDN-01tier-sha256.adelie-crl.pem

            Netscape Comment: 
                serverCert
            X509v3 Subject Alternative Name: 
                DNS:w7ecg-jitsi.local.mesh, DNS:w7ecg-jitsi-meet.local.mesh, DNS:w7ecg-meet.local.mesh, DNS:w7ecg-web.local.mesh
    Signature Algorithm: sha256WithRSAEncryption
         6a:6b:0d:9e:a6:ab:9e:95:ef:df:56:9c:3c:97:be:a9:42:da:
         a1:bb:bc:89:0d:c9:32:e4:40:a0:48:ae:55:8a:53:f6:ac:ca:
         a2:41:31:e7:5c:c7:42:db:a3:86:01:dc:50:8f:df:8a:2b:a4:
         db:68:94:1e:57:b8:22:48:99:b6:e3:2a:28:14:9d:7f:36:2d:
         71:89:8e:61:05:45:48:5f:00:55:ae:b3:d5:d1:eb:42:e5:4d:
         75:87:3a:52:f4:a8:e7:79:ca:33:10:ff:35:7a:09:f0:2c:e7:
         3a:5a:e1:10:42:87:94:0b:82:09:dc:2d:7a:2a:10:76:70:75:
         a5:22:f4:4f:52:ab:98:37:e1:d1:d4:3b:1e:e1:59:38:03:f5:
         54:72:91:29:de:43:2d:33:79:15:1e:c5:ee:a1:3a:5c:2f:2e:
         3e:9d:ce:b1:6d:1f:3b:9c:39:82:00:9a:47:14:15:5e:38:8c:
         b7:50:9b:b1:f2:55:d7:96:94:0a:89:f1:f9:99:bd:73:a2:5d:
         8d:4a:b2:51:02:40:25:d6:d5:36:23:6b:6e:36:ef:4d:5e:7f:
         f5:59:20:bf:f7:20:87:61:50:a3:92:46:a0:2d:bf:fb:87:d1:
         57:44:14:54:ec:7f:2e:9c:97:95:a6:b2:6c:67:64:61:5b:1a:
         01:e7:5a:b0:d0:71:0b:01:cf:1a:5b:b2:cc:5a:3e:a4:ac:1f:
         02:9b:b9:6b:a2:43:d3:73:b4:75:d5:bc:93:13:57:7e:d3:d6:
         93:db:26:30:56:c7:3d:b5:b4:f3:51:b1:36:31:62:de:0e:d6:
         42:e5:e5:dd:f6:0a:c8:88:22:0b:c1:79:43:7f:26:43:4f:bb:
         62:39:9f:04:d0:33:3a:22:b6:77:15:6c:8d:b8:b2:1c:13:49:
         5f:c0:4e:a1:57:4b:2c:db:b7:e6:b5:b2:ed:71:d3:7b:ea:3b:
         f6:94:2e:c1:8d:92:b5:b9:1a:88:07:e1:02:7b:79:25:9f:47:
         2d:6d:cc:68:d2:15:23:35:3f:18:38:84:8a:49:0b:37:af:d1:
         a2:1a:5a:0e:5c:17:ff:90:ab:d8:86:7e:4d:1f:d6:c9:1d:b3:
         9c:95:2c:e5:31:41:16:55:bd:4f:1a:c8:ce:79:67:5a:ab:f5:
         31:7f:ca:f9:91:b8:e3:33:94:49:12:4b:7f:34:32:9b:4e:6f:
         b1:30:eb:a4:01:9b:74:9c:46:00:ca:78:42:66:42:13:70:e1:
         bf:df:82:8f:83:d0:db:4a:30:9f:ad:81:29:d0:92:d4:61:92:
         fc:3a:37:14:82:2b:d9:a8:18:83:0d:c7:0f:90:88:a2:50:39:
         ad:49:8a:e9:d3:15:9d:8a
~~~

### Importing the CA Certificate into the *Client's* Trust Store

Windows

- Follow [these instructions](https://docs.microsoft.com/en-us/skype-sdk/sdn/articles/installing-the-trusted-root-certificate).

Mac

- Follow [these instructions](https://www.eduhk.hk/ocio/content/faq-how-add-root-certificate-mac-os-x).  Note that although the instructions say the cert must have a `.cer` extension, it also works with certs with a `.crt` extension.

Linux

- The CA certificate must be Base64 encoded, also known as Privacy Enhanced Mail (PEM) format.
- Copy the certificate to `/usr/local/share/ca-certificates`.  Give it a `.crt` extension.
- Run this command:
	
			sudo update-ca-certificates
			
	This will create a symlink in `/etc/ssl/certs` to the CA certificate in `/usr/local/share/ca-certificates`.

## NGINX Configuration Notes

The Jitsi install script described in the [Jitsi Meet Quick Install](https://github.com/jitsi/jitsi-meet/blob/master/doc/quick-install.md) guide will automatically install `nginx` if it's not automatically installed.

## Uninstall (or Starting Over)

Use these instructions to uninstall and start over, not the ones in the [Jitsi Meet Quick Install](https://github.com/jitsi/jitsi-meet/blob/master/doc/quick-install.md) guide.

		sudo apt-get --purge remove jigasi jitsi-meet jitsi-meet-web-config jitsi-meet-prosody jitsi-meet-turnserver jitsi-meet-web jicofo jitsi-videobridge2	prosody
		sudo rm -rf /var/lib/prosody
	