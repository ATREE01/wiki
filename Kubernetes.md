# Kubernetes Resources

## Networking Solutions

### MetalLB

```bash
$ helm repo add metallb https://metallb.github.io/metallb
$ helm install metallb metallb/metallb
```

#### Using BGP Mode

You can place the following configurations in a single YAML file and apply with `kubectl apply -f metallb.yaml`:

##### BGPPeer
```yaml
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

##### IP Pool
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ipv4-pool
  namespace: metallb-system
spec:
  addresses:
  # Available IP addresses, can specify multiple, including ipv4 and ipv6
  - 10.50.50.106-10.50.50.110
```

##### BGP Advertisement
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

#### Important Notes
- If Master Nodes have the `node.kubernetes.io/exclude-from-extern` label, remove it with:
  ```bash
  $ kubectl label node <Node-Name> node.kubernetes.io/exclude-from-external-load-balancers-
  ```
- Ensure `MasterNode` can communicate with the `Router` via `TCP port 179`

### Nginx Ingress
A sample service is accessible at `web.example.com`.

### Cert Manager

1. Install using `helm`
2. After installation, create a cluster issuer:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    email: anyu9898@yahoo.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: nginx

---
# Add these annotations to your ingress
# cert-manager.io/cluster-issuer: "letsencrypt"
# tls:
#   - hosts:
#       - "<domain-name>"
#     secretName: <domain-name>-tls
```
---
## Storage

### Longhorn

Configuration steps:
1. Use Ansible to download and install required packages.
2. Enable necessary services for Longhorn.
3. Resolve potential multipath issues. *(Currently under investigation.)*
4. Cannot use NFS as the disk directly. Current solution: mount the NFS to PVE and then allocate it to VM so it's treated as local storage.

---

## Services

### Harbor Registry

#### Deployment

1. Reference this [document](https://goharbor.io/docs/2.0.0/install-config/configure-https/) to set up TLS.
2. Move the CA to `/usr/local/share/ca-certificate/` and then restart containerd.
3. If encountering problems when pulling, reference [this guide](https://goharbor.io/docs/2.0.0/working-with-projects/working-with-images/pulling-pushing-images/).
4. Use `helm install harbor harbor/ -n harbor` to deploy.

---

### Grafana

#### Deployment

1. Pull the chart
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm pull prometheus-community/kube-prometheus-stack
```
2. Modify values.
```conf
grafana:
  ingress:
    enabled: true
    hosts:
      - grafana.lan

  persistence:
    enabled: true
    type: sts
    storageClassName: "longhorn"
    accessModes:
      - ReadWriteOnce
    size: 20Gi
```
3. 
```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace -f values.yaml

```


---

### Argo CD

#### Deployment

1. Create namespace
```bash
kubectl create namespace argocd
```
2. 設定`values.yaml`
```bash
helm show values argo-cd/argo-cd > values.yaml
```
```conf
configs:
  params:
    server.insecure: true
global:
  # -- Default domain used by all components
  ## Used for ingresses, certificates, SSO, notifications, etc.
  domain: argocd.lan
---
ingress:
    # -- Enable an ingress resource for the Argo CD server
    enabled: true
    # -- Specific implementation for ingress controller. One of `generic`, `aws` or `gke`
    ## Additional configuration might be required in related configuration sections
    controller: generic
    # -- Additional ingress labels
    labels: {}
    # -- Additional ingress annotations
    ## Ref: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
    annotations: {}
      # nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
      # nginx.ingress.kubernetes.io/ssl-passthrough: "true"

    # -- Defines which ingress controller will implement the resource
    ingressClassName: "nginx"

    # -- Argo CD server hostname
    # @default -- `""` (defaults to global.domain)
    hostname: ""

    # -- The path to Argo CD server
    path: /

    # -- Ingress path type. One of `Exact`, `Prefix` or `ImplementationSpecific`
    pathType: Prefix

    # -- Enable TLS configuration for the hostname defined at `server.ingress.hostname`
    ## TLS certificate will be retrieved from a TLS secret `argocd-server-tls`
    ## You can create this secret via `certificate` or `certificateSecret` option
    tls: false
```

3. Deploy
```bash
 helm install argo-cd argo-cd/argo-cd -f values.yaml -n argocd
```

4. Login with password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -drd}" | base64 -d
```


---

### ARC(Action Runner Controller)

#### Delpoyment

Install the arc.
```bash
NAMESPACE="arc-systems"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

這篇文章的下面有跟你說可以用PAT授權

https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/authenticate-to-the-api

剩下的就照著這篇文章去做就好了，我是創了一個organization，不然就是只能針對單一repo。

https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/deploy-runner-scale-sets

Install the scale set
```bash
INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="https://github.com/TREE-FARM"
GITHUB_PAT="<PAT>"
helm install "${INSTALLATION_NAME}" \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set

```

### Finance Manager
- Deployed using Helm chart.

