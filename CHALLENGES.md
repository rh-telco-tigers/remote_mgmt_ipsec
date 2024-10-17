# Areas for Improvement

## Pre-caching of images not directly supported.

Pre-caching of additional images in the IBI image is not directly supported at this time. A workaround is currently documented in the README.md file.

## Disk partitioning is not supported for IBI Clone Clusters

It is not possible to create additional partitions on the install disk for use by LVM.

## Network Name Resolution

DNS name resolution of IPs in the HUB SITE for pods on the OpenShift SDN works because we path the DNS operator to steer DNS lookups for HUB SITE domain names as part of the configuration, however the HOST node does NOT use this configuration, so is unable to do name resolution over the IPSec tunnel.

* the simplest solution is to have public DNS records that resolve to IP addresses on the private network. See `registry.xphyrlab.net`, and `git.xphyrlab.net` for examples.
* Follow the steps here: https://access.redhat.com/solutions/7011361 to modify the DNS entries on the node and add the HUB SITE dns server
  * The issue with this is that ALL DNS requests will go over the IPSec tunnel going forward
* Something else

## Remote management opportunities

* hardware failure/os hang/OS image corrupt - how recover from this
* Disable all access 22, 443, 6443, 80 - Possible Solution "Ingress Node Firewall"-  [https://docs.openshift.com/container-platform/4.16/networking/network_security/ingress-node-firewall-operator.html](https://docs.openshift.com/container-platform/4.16/networking/network_security/ingress-node-firewall-operator.html)

## Software opportunities

* IBI Seed process requires fixed/known disk size 

## Security opportunities

Authentication in clear text on configuration ISO image [https://github.com/rh-telco-tigers/remote_mgmt_ipsec/blob/main/extra-manifests-templates/99-ipsec-endpoint-config.bu#L38](https://github.com/rh-telco-tigers/remote_mgmt_ipsec/blob/main/extra-manifests-templates/99-ipsec-endpoint-config.bu#L38)

