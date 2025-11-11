# Testing VPN

You can test the network setup prior to deploying SNO clusters by following these steps:

## Install required software

```
# dnf-update
# dnf install libreswan
```

## Import the certificate

```
$ sudo ipsec checknss
$ sudo ipsec import vpn.example.com.p12 
Enter password for PKCS12 file: 
pk12util: PKCS12 IMPORT SUCCESSFUL
```

### Create IPSec config file

Create the following file and place in /etc/ipsec.d/aws-vpn.conf

```
conn aws-vpn
    ikev2=insist
    # pick up our dynamic IP
    left=<your clients local IP address>
    leftsubnet=0.0.0.0/0
    leftcert=<match the name of the cert you imported above>
    leftid=%fromcert
    leftmodecfgclient=yes
    # right can also be a DNS hostname
    right=<vpn server name>
    # if access to the remote LAN is required, enable this, otherwise use 0.0.0.0/0
    #rightsubnet=10.10.0.0/16
    require-id-on-certificate=no
    rightsubnet=0.0.0.0/0
    fragmentation=yes
    # trust our own Certificate Agency
    rightca=%same
    authby=rsasig
    # allow narrowing to the server’s suggested assigned IP and remote subnet
    narrowing=yes
    # support (roaming) MOBIKE clients (RFC 4555)
    mobike=yes
    # initiate connection
    auto=start
```

## Start IPSec

```
# systemctl enable ipsec --now
# ipsec status
```
