# Areas for Improvement

## IPv6 Support

This process was tested/deployed with IPv4. It should be re-tested with IPv6 to ensure that all the processes work via IPv6

## Pre-caching of images not directly supported.

Pre-caching of additional images in the IBI image is not directly supported at this time. A workaround is currently documented in the README.md file.

## Disk partitioning is not supported for IBI Clone Clusters

It is not possible to create additional partitions on the install disk for use by LVM.

> **NOTE:** This is partially solved with https://github.com/openshift-kni/lifecycle-agent/pull/677. The PR may be rejected for a better more flexible solution based around the MCO syntax. 

```
# sgdisk -n 6:0:0 --change-name 6:lvm-storage /dev/nvme0n1
# sgdisk -p /dev/nvme0n1
Disk /dev/nvme0n1: 1950351360 sectors, 930.0 GiB
Model: VMware Virtual NVMe Disk
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 00000000-0000-4000-A000-000000000001
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 2048, last usable sector is 1950351326
Partitions will be aligned on 2048-sector boundaries
Total free space is 0 sectors (0 bytes)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  BIOS-BOOT
   2            4096          264191   127.0 MiB   EF00  EFI-SYSTEM
   3          264192         1050623   384.0 MiB   8300  boot
   4         1050624       251658239   119.5 GiB   8300  root
   5       251658240      1300234239   500.0 GiB   8300  varlibcontainers
   6      1300234240      1950351326   310.0 GiB   8300  lvm-storage
```

## Network Name Resolution

DNS name resolution of IPs in the HUB SITE for pods on the OpenShift SDN works because we path the DNS operator to steer DNS lookups for HUB SITE domain names as part of the configuration, however the HOST node does NOT use this configuration, so is unable to do name resolution over the IPSec tunnel.

* the simplest solution is to have public DNS records that resolve to IP addresses on the private network. See `registry.xphyrlab.net`, and `git.xphyrlab.net` for examples.
* Follow the steps here: https://access.redhat.com/solutions/7011361 to modify the DNS entries on the node and add the HUB SITE dns server
  * The issue with this is that ALL DNS requests will go over the IPSec tunnel going forward
* Something else

## Remote management opportunities

* hardware failure/os hang/OS image corrupt - how recover from this
* Disable all access 22, 443, 6443, 80 on Primary network - Possible Solution [Ingress Node Firewall](https://docs.openshift.com/container-platform/4.16/networking/network_security/ingress-node-firewall-operator.html)
* The IPSec tunnel allows for Port 22 (ssh) access initialing outside the cluster, but does not work for the managment ports for the platform (Port 80/443 and 6443). This limits the ability to use `kubectl`, `oc` and other api management tools remotely.

## Software opportunities

* IBI Seed process requires fixed/known disk size 

## Security opportunities

Authentication in clear text on configuration ISO image [https://github.com/rh-telco-tigers/remote_mgmt_ipsec/blob/main/extra-manifests-templates/99-ipsec-endpoint-config.bu#L38](https://github.com/rh-telco-tigers/remote_mgmt_ipsec/blob/main/extra-manifests-templates/99-ipsec-endpoint-config.bu#L38)

## Log Streaming

Need a way to stream logs off the remote server to ACM 

## Alerts/Prometheus data

Need a way to ensure that the speeds and feeds can be accessed seen in acm

