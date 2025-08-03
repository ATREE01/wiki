# Rancher

##  traefik
### Installation Steps
1. During installation, add the annotation `cloudprovider.harvesterhci.io/ipam: dhcp` to `.Service.annotations` to automatically obtain an IP address from `harvester`.

2. For `kube-vip DaemonSets` configuration:
    ```yaml
    enable_service_security: false
    vip_interface: enp1s0  # Primary network interface
    vip_servicesinterface: enp2s0
    EGRESS_CLEAN: true
    ```

## nginx
`Rancher`本身就會提供nginx，可以直接使用要expose的話可以用`fortigate`

### Expose

1. `Fortigate`建立一個`external connector`，在`rancher cluster`建一個`service account`然後建一個`cluster role`跟`role binding`設定一下權限然後把`token`設定到`fortigate`裡。詳情參考[這個](https://docs.fortinet.com/document/fortigate-private-cloud/7.6.0/kubernetes-administration-guide/718577)