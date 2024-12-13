# LoadBalancer

##  MetalLB

``` bash
$ helm repo add metallb https://metallb.github.io/metallb
$ helm install metallb metallb/metallb
```

### Using BGP mode

可以將以下檔案放在同一個yaml file 然後使用`kubectl apply -f metallb.yaml`

#### BGPPeer
``` yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: bpg-peer
  namespace: metallb-system
spec:
  myASN: 65100 # The ASN number of metallb
  peerASN: 65000 # The ASN number of router
  peerAddress: 10.50.50.254
```
#### IP Pool
``` yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ipv4-pool
  namespace: metallb-system
spec:
  addresses:
  # 可分配的 IP 地址,可以指定多个，包括 ipv4、ipv6
  - 10.50.50.106-10.50.50.110
```
#### BPG Advertisement
```yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: bgp-advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
    - ipv4-pool
```

記得注意如果Master Node有 `node.kubernetes.io/exclude-from-extern`的Label可以用這個指令把它移掉
```bash
$ kubectl label node <Node-Name> node.kubernetes.io/exclude-from-external-load-balancers-
```

#### 額外注意事項
1. 確保`MasterNode`可以跟`Router`透過`TCP port179`通信