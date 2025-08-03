# 🖥️ Changing a Proxmox VM ID Safely

This guide explains how to safely change a Proxmox VM ID (e.g., from `250` to `30001`) along with its disk image.

> **Applies to**: Proxmox VE

## Manual Method

Use this if you need full control or cannot clone.

### 1. Shutdown the VM
```bash
qm shutdown 250
```

### 2. Rename the Configuration File
```bash
mv /etc/pve/qemu-server/250.conf /etc/pve/qemu-server/30001.conf
```

### 3. Rename Disk Images

#### For directory-based storage:
```bash
mv /var/lib/vz/images/250 /var/lib/vz/images/30001
```

#### For LVM-thin:
Find the logical volume:
```bash
lvs
```

Rename it:
```bash
lvrename pve vm-250-disk-0 vm-30001-disk-0
```

Update the new config file (`/etc/pve/qemu-server/30001.conf`) to reflect the new disk names.

### 4. Start the VM
```bash
qm start 30001
```

---

## 📝 Notes

- Ensure there is **no VM** with ID `30001` before you begin.
- Always back up important VMs before performing operations like renaming or cloning.