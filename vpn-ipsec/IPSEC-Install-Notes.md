# INTRO

This is NOT intended to be a full set of instructions to re-create the VPN Server. This is a set of notes which will allow recreation of the test lab.

> **NOTE:** The use of Fedora Server and [Libreswan](https://libreswan.org/) as the IPSec server is NOT a requirement. Other secure gateway hardware that supplies IPSec tunneling may be used, but will not be documented here.

## Requirements

* Start with a Fedora 40 Server image
* Be sure it is fully updated before proceeding

## Configuring the IPSec Server

```
$ dnf install libreswan
$ echo 1 > /proc/sys/net/ipv4/ip_forward
$ systemctl stop firewalld
```

create ipsec configuration file

/etc/ipsec.d/access-vpn.conf
```
conn access-vpn
    # added the following two lines to maybe work with OCP
    ike=aes256-sha2_512;modp2048,aes128-sha2_512;modp2048
    esp=aes_gcm256-null,aes_gcm128-null,aes256-sha2_512,aes128-sha2_512
    # The server's actual IP goes here - not elastic IPs
    left=172.16.10.11
    leftcert=vpn.example.com
    leftid=@vpn.example.com
    leftsendcert=always
    leftsubnet=172.16.0.0/16
    leftrsasigkey=%cert
    # Clients
    right=%any
    # your addresspool to use - you might need NAT rules if providing full internet to clients
    rightaddresspool=172.16.11.100-172.16.11.150
    rightca=%same
    rightrsasigkey=%cert
    #
    # connection configuration
    # DNS servers for clients to use
    modecfgdns=172.16.15.6,172.16.15.7
    narrowing=yes
    # added to test more options
    #leftmodecfgserver=yes
    # recommended dpd/liveness to cleanup vanished clients
    dpddelay=30
    dpdtimeout=120
    dpdaction=clear
    auto=add
    ikev2=insist
    rekey=no
    # ikev2 fragmentation support requires libreswan 3.14 or newer
    fragmentation=yes
```

## Creating the Certificates for use

You will need a Cert Database to work from. This will contain the root CA as well as all server and client certs used to auth to the IPSec server.

### Create a Cert Database and root CA

In your home directory, create a database to generate certificates for this example. This database will hold the private key of the CA and allow you to generate new host certificates.

```
# mkdir ${HOME}/certdb
# certutil -N -d sql:${HOME}/certdb
Enter a password which will be used to encrypt your keys.
The password should be at least 8 characters long,
and should contain at least one non-alphabetic character.

Enter new password: 
Re-enter password: 
Generate the CA certificate with a CA basic constraints extension
# certutil -S -x -n "Example CA" -s "O=Example,CN=Example CA" \
 -k rsa -g 4096 -v 12 -d sql:${HOME}/certdb -t "CT,," -2

A random seed must be generated that will be used in the
creation of your key.  One of the easiest ways to create a
random seed is to use the timing of keystrokes on a keyboard.

To begin, type keys on the keyboard until this progress meter
is full.  DO NOT USE THE AUTOREPEAT FUNCTION ON YOUR KEYBOARD!

Continue typing until the progress meter is full:

|************************************************************|

Finished.  Press enter to continue: 

Generating key.  This may take a few moments...

Is this a CA certificate [y/N]?
y
Enter the path length constraint, enter to skip [<0 for unlimited path]: > 
Is this a critical extension [y/N]?
N
```


### Create new Server IPSec Key

```
Generate the server certificate and assign extensions:
# certutil -S -c "Example CA" -n "vpn.example.com" -s "O=Example,CN=vpn.example.com" \
   -k rsa -g 4096 -v 12 -d sql:${HOME}/certdb -t ",," -1 -6 -8 "vpn.example.com"

A random seed must be generated that will be used in the
creation of your key.  One of the easiest ways to create a
random seed is to use the timing of keystrokes on a keyboard.

To begin, type keys on the keyboard until this progress meter
is full.  DO NOT USE THE AUTOREPEAT FUNCTION ON YOUR KEYBOARD!

Continue typing until the progress meter is full:
   
|************************************************************|
   
Finished.  Press enter to continue:
      
Generating key.  This may take a few moments...
   
                0 - Digital Signature
                1 - Non-repudiation
                2 - Key encipherment
                3 - Data encipherment
                4 - Key agreement
                5 - Cert signing key
                6 - CRL signing key
                Other to finish
 > 0
                0 - Digital Signature
                1 - Non-repudiation
                2 - Key encipherment
                3 - Data encipherment
                4 - Key agreement
                5 - Cert signing key
                6 - CRL signing key
                Other to finish
 > 2
                0 - Digital Signature
                1 - Non-repudiation
                2 - Key encipherment
                3 - Data encipherment
                4 - Key agreement
                5 - Cert signing key
                6 - CRL signing key
                Other to finish
 > 8 
Is this a critical extension [y/N]?
N
                0 - Server Auth
                1 - Client Auth
                2 - Code Signing
                3 - Email Protection
                4 - Timestamp
                5 - OCSP Responder
                6 - Step-up
                7 - Microsoft Trust List Signing
                Other to finish
 > 0
                0 - Server Auth
                1 - Client Auth
                2 - Code Signing
                3 - Email Protection
                4 - Timestamp
                5 - OCSP Responder
                6 - Step-up
                7 - Microsoft Trust List Signing
                Other to finish
 > 1
                0 - Server Auth
                1 - Client Auth
                2 - Code Signing
                3 - Email Protection
                4 - Timestamp
                5 - OCSP Responder
                6 - Step-up
                7 - Microsoft Trust List Signing
                Other to finish
 > 8
Is this a critical extension [y/N]?
N
```

### Create new Client IPSec Key

```
ssh vpnipsec
$ certutil -S -c "Xphyr CA" -n "remotesno2.example.com" -s "O=Xphyrlab,CN=remotesno2.example.com"    -k rsa -g 4096 -v 12 -d sql:${HOME}/certdb -t ",," -1 -6 -8 "remotesno2.example.com"
Enter Password or Pin for "NSS Certificate DB":

A random seed must be generated that will be used in the
creation of your key.  One of the easiest ways to create a
random seed is to use the timing of keystrokes on a keyboard.

To begin, type keys on the keyboard until this progress meter
is full.  DO NOT USE THE AUTOREPEAT FUNCTION ON YOUR KEYBOARD!


Continue typing until the progress meter is full:

|************************************************************|

Finished.  Press enter to continue:


Generating key.  This may take a few moments...

		0 - Digital Signature
		1 - Non-repudiation
		2 - Key encipherment
		3 - Data encipherment
		4 - Key agreement
		5 - Cert signing key
		6 - CRL signing key
		Other to finish
 > 0
		0 - Digital Signature
		1 - Non-repudiation
		2 - Key encipherment
		3 - Data encipherment
		4 - Key agreement
		5 - Cert signing key
		6 - CRL signing key
		Other to finish
 > 2
		0 - Digital Signature
		1 - Non-repudiation
		2 - Key encipherment
		3 - Data encipherment
		4 - Key agreement
		5 - Cert signing key
		6 - CRL signing key
		Other to finish
 > 8
Is this a critical extension [y/N]?
n
		0 - Server Auth
		1 - Client Auth
		2 - Code Signing
		3 - Email Protection
		4 - Timestamp
		5 - OCSP Responder
		6 - Step-up
		7 - Microsoft Trust List Signing
		Other to finish
 > 0
		0 - Server Auth
		1 - Client Auth
		2 - Code Signing
		3 - Email Protection
		4 - Timestamp
		5 - OCSP Responder
		6 - Step-up
		7 - Microsoft Trust List Signing
		Other to finish
 > 1
		0 - Server Auth
		1 - Client Auth
		2 - Code Signing
		3 - Email Protection
		4 - Timestamp
		5 - OCSP Responder
		6 - Step-up
		7 - Microsoft Trust List Signing
		Other to finish
 > 8
Is this a critical extension [y/N]?
n
```

Repeat this process for each remote client you wish to connect to the primary VPN server.

### Export the VPN Server Cert

To configure the VPN server, we need to export the certificate that we created above in [Create new Server IPSec Key](#create-new-server-ipsec-key). This file then needs to be imported into the IPSec SSL database.

```
$ pk12util -o vpn.example.com.p12 -n "vpn.example.com" -d sql:${HOME}/certdb/
Enter password for PKCS12 file:
Re-enter password:  
pk12util: PKCS12 EXPORT SUCCESSFUL
```

Initialize the IPSec Database (this only needs to be done once per configuration)

```
$ sudo ipsec checknss
Initializing NSS database
```

Now import the cert into the IPSec database

```bash
$ sudo ipsec import vpn.example.com.p12 
Enter password for PKCS12 file: 
pk12util: PKCS12 IMPORT SUCCESSFUL
```

### Export the Client Server Cert

```
$ pk12util -o remotesno2.example.com.p12 -n "remotesno2.example.com" -d sql:${HOME}/certdb/
Enter Password or Pin for "NSS Certificate DB":
Enter password for PKCS12 file:
Re-enter password:
pk12util: PKCS12 EXPORT SUCCESSFUL
```

Distribute the .pk12 files as needed

## References

* [Creating an IPSec Server](https://libreswan.org/wiki/VPN_server_for_remote_clients_using_IKEv2) 
