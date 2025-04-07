### **Home Lab Setup**

#### **Dell R630 Specifications**

- **CPU**: Dual Intel Xeon E5-2699v3  
- **RAM**: 80 GB DDR4-2133  
- **Disk Configuration**:  
  - **OS**: 600 GB x 2  
  - **Storage**: 1.2 TB x 2 (dedicated for TrueNAS)

---

### PVE network setting
1. vmbr0 for the physical interface to outer wire.
2. vmbr1 for the logical interface act as openwrt lan port.
3. vmbr2 for the logical interface act as the NAopenwrt lan port.

---

### **Tips**

- For **Ansible**, use the following flags to enable sudo privilege with a password:  
  ```bash
  --ask-become-pass  # Prompt for privilege escalation password
  --ask-pass         # Prompt for SSH user password
  ```

---

### **Virtual Machines**

#### **TrueNAS**

1. Enable disk passthrough for the TrueNAS VM using the following steps:
   - Run the command to identify device IDs:  
     ```bash
     lsblk | awk 'NR==1{print $0" DEVICE-ID(S)"} NR>1{dev=$1; printf $0" "; system("find /dev/disk/by-id -lname \"*"dev"\" -printf \" %p\""); print "";}'
     ```  
     *(This command lists disk device IDs while excluding partitions and logical volumes.)*
   - Configure the VM to passthrough disks:  
     ```bash
     qm set <vm-id> <disk-name> /dev/disk/by-id/<disk-id>
     ```  
   Reference: [How to Install TrueNAS in Proxmox with HDD Passthrough](https://www.youtube.com/watch?v=MkK-9_-2oko)

---

#### **Kubernetes Setup**

#### **Harbor Registry**

##### Deploy

1. Reference this [document](https://goharbor.io/docs/2.0.0/install-config/configure-https/) here to set up the tls.

3. Move the ca to `/usr/local/share/ca-certificate/` and then restart containerd.

3. If mee any problem when pulling. Reference [this](https://goharbor.io/docs/2.0.0/working-with-projects/working-with-images/pulling-pushing-images/).

4. Use `helm install harbor harbor/ -n harbor` to deploy. 


#### **Networking**

- **MetalLB**:  
  Configured in BGP mode to allocate IPs for services.

  1. When configuting the BGP mode remember to open the port on router.

- **Nginx Ingress**:  
  A sample service is accessible at `web.example.com`.

- **Cert Manager**:

1. Use `helm` to installed.
2. After install create a cluster issuser
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
# Add this annotation to ingress
cert-manager.io/cluster-issuer: "letsencrypt"
tls:
  - hosts:
      - "<domain-name>"
    secretName: <domain-name>-tls
```


  



#### **Storage**

- **Longhorn**:  

  Steps to configure:  
  1. Use Ansible to download and install the required packages.  
  2. Enable necessary services for Longhorn.  
  3. Resolve potential multipath issues. *(Currently under investigation.)*  
  <!-- 4. Mount NFS storage and assign it to Longhorn:  
     - Create a mount point:  
       ```bash
       mkdir /mnt/path/to/mount/point
       ```  
     - Add an entry to `/etc/fstab`:  
       ```bash
       <NFS_SERVER_IP_OR_HOSTNAME>:<NFS_SHARE_PATH> /mnt/nfs nfs defaults 0 0
       ```   -->
  4. Can't use NFS as the disk. The current solution is mount the nfs to PVE and then give it to VM so it would treat it as local Storage.


#### service

- **Finance manager**:

1. Helm chart is used to deploy.

---