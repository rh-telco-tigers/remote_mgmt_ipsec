# Areas for Improvement



## Remote management opportunites

* hardware failure/os hang/OS image corrupt - how recover from this
* Disable all access 22, 443, 6443, 80 - Possible Solution "Ingress Node Firewall"-  [https://docs.openshift.com/container-platform/4.16/networking/network_security/ingress-node-firewall-operator.html](https://docs.openshift.com/container-platform/4.16/networking/network_security/ingress-node-firewall-operator.html)


## Software opportunites

* IBI Seed process requires fixed/known disk size 

## Security opportunites

Authentication in clear text on configuration ISO image [https://github.com/rh-telco-tigers/remote_mgmt_ipsec/blob/main/extra-manifests-templates/99-ipsec-endpoint-config.bu#L38](https://github.com/rh-telco-tigers/remote_mgmt_ipsec/blob/main/extra-manifests-templates/99-ipsec-endpoint-config.bu#L38)

