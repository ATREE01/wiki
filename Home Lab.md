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

### **Subnet Settings Description**

- **172.16.0.0/24**:  
  Local network

#### VLAN
- **10.50.30.0/24**:  
- **10.50.50.0/24**:  
  Most machine are here for now.
- **10.50.60.0/24**:  


### **Tips**

- For **Ansible**, use the following flags to enable sudo privilege with a password:  
  ```bash
  --ask-become-pass  # Prompt for privilege escalation password
  --ask-pass         # Prompt for SSH user password
  ```
---

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



