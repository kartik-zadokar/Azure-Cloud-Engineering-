# Day 02 – Azure Virtual Machine Disks & Storage Management

## 🎯 Objective

Learn Azure Managed Disks and Linux storage administration by attaching an additional data disk to an existing Azure Ubuntu Virtual Machine.

The objective was to provision a managed disk, identify it inside Linux, create a partition, format it, mount it and configure persistent mounting across VM reboots.

---

## ☁️ Azure Services & Technologies Used

- Microsoft Azure Virtual Machines
- Azure Managed Disks
- Azure Portal
- Ubuntu Server 24.04 LTS
- Linux Disk Management
- NVMe Storage Devices
- ext4 Filesystem
- Linux `/etc/fstab`
- SSH

---

## 🏗️ Infrastructure Configuration

| Component | Configuration |
|---|---|
| Virtual Machine | vm-ubuntu-lab01 |
| Operating System | Ubuntu Server 24.04 LTS |
| OS Disk | 30 GB |
| Additional Data Disk | 10 GB |
| Disk Type | Standard SSD |
| Disk Device | /dev/nvme0n2 |
| Partition | /dev/nvme0n2p1 |
| Filesystem | ext4 |
| Mount Point | /data |

---

## 🔧 Implementation Steps

### 1. Created and Attached Azure Managed Disk

- Opened the Azure Portal.
- Navigated to the existing Virtual Machine.
- Opened Settings → Disks.
- Created a new 10 GB Standard SSD managed disk.
- Attached the disk to the Ubuntu VM.

### 2. Identified the New Disk

Executed:

```bash
lsblk
```

Identified the newly attached 10 GB disk:

```text
nvme0n2       10G  disk
```

The existing OS disk was:

```text
nvme0n1       30G  disk
```

Important: Identified the correct disk before making any partition or formatting changes.

### 3. Created a Partition

Executed:

```bash
sudo fdisk /dev/nvme0n2
```

Created a primary partition using the following fdisk commands:

```text
n
p
1
Enter
Enter
w
```

Verified the partition:

```bash
lsblk
```

Result:

```text
nvme0n2
└── nvme0n2p1
```

### 4. Created an ext4 Filesystem

Formatted the new partition:

```bash
sudo mkfs.ext4 /dev/nvme0n2p1
```

This created an ext4 filesystem on the partition.

### 5. Created a Mount Point

Created the directory:

```bash
sudo mkdir -p /data
```

Mounted the partition:

```bash
sudo mount /dev/nvme0n2p1 /data
```

Verified the mount:

```bash
df -h
```

### 6. Tested Data Storage

Created a test file:

```bash
echo "My first Azure Data Disk" | sudo tee /data/test.txt
```

Verified the file:

```bash
cat /data/test.txt
```

### 7. Configured Persistent Mounting

Retrieved the filesystem UUID:

```bash
sudo blkid /dev/nvme0n2p1
```

Created a backup of the existing fstab configuration:

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

Edited the fstab configuration:

```bash
sudo nano /etc/fstab
```

Added the following entry using the actual filesystem UUID:

```text
UUID=YOUR-UUID /data ext4 defaults,nofail 0 2
```

Validated the configuration:

```bash
sudo mount -a
```

Verified the mount:

```bash
findmnt /data
```

### 8. Verified Mount Persistence

Rebooted the VM:

```bash
sudo reboot
```

After reconnecting through SSH, verified the disk:

```bash
lsblk
df -h
```

Confirmed that the additional data disk remained mounted at `/data` after reboot.

---

## 📚 Key Learnings

### Azure Concepts
- Azure Managed Disks
- OS Disk vs Data Disk
- Standard SSD
- Attaching additional storage to Azure VMs

### Linux Administration
- Identifying storage devices using `lsblk`
- Inspecting disks using `fdisk`
- Creating disk partitions
- Formatting partitions with ext4
- Mounting filesystems
- Understanding filesystem UUIDs
- Configuring persistent mounts using `/etc/fstab`

### Troubleshooting
- Identifying newly attached disks
- Verifying partition creation
- Checking filesystem mounts
- Validating persistent mounting configuration

---

## 💡 Important Commands Reference

| Command | Purpose |
|---|---|
| `lsblk` | List block devices |
| `sudo fdisk -l` | Inspect disk partitions |
| `sudo fdisk /dev/nvme0n2` | Manage disk partitions |
| `sudo mkfs.ext4` | Create ext4 filesystem |
| `sudo mount` | Mount a filesystem |
| `df -h` | Check disk space |
| `sudo blkid` | Retrieve filesystem UUID |
| `sudo nano /etc/fstab` | Configure persistent mounting |
| `sudo mount -a` | Validate and mount fstab entries |
| `findmnt /data` | Verify mount information |

---

## 🔐 Security & Best Practices

- Always identify the correct disk before formatting.
- Never format the operating system disk accidentally.
- Use UUIDs in `/etc/fstab` for reliable filesystem identification.
- Back up configuration files before modifying them.
- Validate fstab entries before rebooting.
- Avoid storing sensitive information in GitHub.

---

## ✅ Project Status

**Completed – Day 02: Azure Managed Disk & Linux Storage Administration**

Successfully attached, partitioned, formatted and mounted an additional Azure managed disk and configured persistent mounting across VM reboots.

---

## 🚀 Next Learning Objective

**Day 03 – Azure High Availability**

- Availability Sets
- Fault Domains
- Update Domains
- Availability Zones
- Virtual Machine Scale Sets
- High Availability Architecture

