### **Home Lab Setup**

#### **Dell R630 Specifications**

- **CPU**: Dual Intel Xeon E5-2699v3  
- **RAM**: 270 GB DDR4-2133  
- **Disk Configuration**:  
  - **OS**: 600 GB x 2  
  - **Storage**: 1.2 TB x 2 (dedicated for TrueNAS)

---

### PVE network setting
1. vmbr0 for the physical interface to outer wire.
2. vmbr1 for the logical interface act as openwrt lan port.
3. vmbr2 for the logical interface act as the NAopenwrt lan port.

#### OpenWRT BGP

`quagga.conf`

```conf
!
! Zebra configuration saved from vty
!   2024/12/12 13:49:11
!
password zebra
!
router bgp 65000
 bgp router-id 140.115.16.180
 neighbor 10.50.50.101 remote-as 65100
 neighbor 10.50.50.101 description k8smasternode1
 neighbor 10.50.50.102 remote-as 65100
 neighbor 10.50.50.102 description k8smasternode2
 neighbor 10.50.50.103 remote-as 65100
 neighbor 10.50.50.103 description k8smasternode3
!
 address-family ipv6
 exit-address-family
 exit
!
access-list vty permit 127.0.0.0/8
access-list vty deny any
!
line vty
 access-class vty
!
```

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

