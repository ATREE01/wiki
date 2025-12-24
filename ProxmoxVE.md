# 🖥️ Changing a Proxmox VM ID Safely

> **Applies to**: Proxmox VE

## Manual Method

Follow these steps for full control or if cloning is not an option.

### 1. Shutdown the VM
```bash
qm shutdown 250
```

### 2. Rename Disk Images

#### For directory-based storage:
```bash
mv /var/lib/vz/images/250 /var/lib/vz/images/30001
```

#### For LVM-thin:
1. Find the logical volume:
  ```bash
  lvs
  ```
2. Rename it:
  ```bash
  lvrename pve vm-250-disk-0 vm-30001-disk-0
  ```
3. Update the new config file (`/etc/pve/qemu-server/30001.conf`) to reflect the new disk names.

### 3. Rename the Configuration File
```bash
mv /etc/pve/qemu-server/250.conf /etc/pve/qemu-server/30001.conf
```

### 4. Start the VM
```bash
qm start 30001
```

---

## **Virtual Machines**

### **TrueNAS**

1. Enable disk passthrough for the TrueNAS VM using these steps:
  - Identify device IDs:
    ```bash
    lsblk | awk 'NR==1{print $0" DEVICE-ID(S)"} NR>1{dev=$1; printf $0" "; system("find /dev/disk/by-id -lname \"*"dev"\" -printf \" %p\"); print "";}'
    ```
    *(This lists disk device IDs while excluding partitions and logical volumes.)*
  - Configure the VM for disk passthrough:
    ```bash
    qm set <vm-id> <disk-name> /dev/disk/by-id/<disk-id>
    ```
  - Reference: [How to Install TrueNAS in Proxmox with HDD Passthrough](https://www.youtube.com/watch?v=MkK-9_-2oko)
