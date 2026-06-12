# Resize AWS EBS Storage on Linux

This guide explains how to increase storage capacity on an AWS EC2 instance after expanding the attached EBS volume.

Increasing the EBS volume size in AWS does **not automatically increase usable disk space inside the operating system**. After resizing the EBS volume, you must:

1. Verify the new disk size
2. Extend the partition
3. Extend the Logical Volume (LVM)
4. Grow the filesystem

This example assumes:

- Linux server using LVM
- XFS filesystem
- Root and `/var` are separate logical volumes

---

# Architecture Overview

Before Expansion:

```text
EBS Volume (50 GB)
        │
        ▼
Partition
        │
        ▼
LVM Volume Group
        │
 ┌──────┴──────┐
 ▼             ▼
Root LV      Var LV
20 GB        20 GB
```

After Expansion:

```text
EBS Volume (100 GB)
        │
        ▼
Partition Expanded
        │
        ▼
LVM Volume Group
        │
 ┌──────┴──────┐
 ▼             ▼
Root LV      Var LV
40 GB        50 GB
```

---

# Step 1: Verify Current Disk Layout

Check available disks and partitions:

```bash
lsblk
```

Example Output:

```text
NAME               SIZE
nvme0n1             100G
├─nvme0n1p1           1G
├─nvme0n1p2           1G
├─nvme0n1p3           2G
└─nvme0n1p4          46G
   ├─RootVG-rootVol  20G
   └─RootVG-varVol   20G
```

Purpose:

- Verify EBS size increase
- Identify partition number
- Identify logical volumes

---

# Step 2: Expand Partition

After increasing the EBS volume in AWS, the partition still uses the old size.

Use:

```bash
sudo growpart /dev/nvme0n1 4
```

Breakdown:

```text
/dev/nvme0n1
```

Disk name.

```text
4
```

Partition number.

This expands:

```text
nvme0n1p4
```

to consume newly available space.

---

# Verify Partition Expansion

Run:

```bash
lsblk
```

Expected:

```text
nvme0n1p4
```

should now show the increased size.

---

# Step 3: Check Volume Group Free Space

Verify available space inside the Volume Group:

```bash
sudo vgs
```

Example:

```text
VG      #PV #LV VSize   VFree
RootVG    1   2  96.00g 50.00g
```

Meaning:

```text
50 GB available
for allocation
```

---

# Step 4: Extend Root Logical Volume

Increase root volume:

```bash
sudo lvextend -L +20G /dev/RootVG/rootVol
```

Breakdown:

```text
+20G
```

Add 20 GB.

Current:

```text
20 GB
```

After extension:

```text
40 GB
```

---

# Step 5: Extend /var Logical Volume

Increase `/var` volume:

```bash
sudo lvextend -L +30G /dev/RootVG/varVol
```

Current:

```text
20 GB
```

After extension:

```text
50 GB
```

---

# Verify Logical Volumes

Run:

```bash
sudo lvs
```

Example:

```text
LV       VG      Size
rootVol  RootVG  40G
varVol   RootVG  50G
```

---

# Step 6: Resize Root Filesystem

Logical volume size changes do not automatically increase filesystem size.

For XFS filesystems:

```bash
sudo xfs_growfs /
```

Purpose:

```text
Expand filesystem
to use newly allocated space
```

---

# Step 7: Resize /var Filesystem

Extend filesystem:

```bash
sudo xfs_growfs /var
```

Purpose:

```text
Expand /var filesystem
to use additional storage
```

---

# Verify Filesystem Expansion

Check final usage:

```bash
df -h
```

Example:

```text
Filesystem            Size
/dev/mapper/rootVol    40G
/dev/mapper/varVol     50G
```

---

# Complete Resize Workflow

```text
Increase EBS Volume
        │
        ▼
Verify Disk
(lsblk)
        │
        ▼
Expand Partition
(growpart)
        │
        ▼
Extend LVM Volumes
(lvextend)
        │
        ▼
Grow Filesystems
(xfs_growfs)
        │
        ▼
Verify Capacity
(df -h)
```

---

# Useful Verification Commands

## View Block Devices

```bash
lsblk
```

---

## View Volume Groups

```bash
vgs
```

---

## View Logical Volumes

```bash
lvs
```

---

## View Filesystem Usage

```bash
df -h
```

---

## View Partition Layout

```bash
fdisk -l
```

---

# Real DevOps Use Cases

## Elasticsearch Storage Expansion

When Elasticsearch disk utilization exceeds:

```text
80%
```

expand EBS storage without rebuilding the server.

---

## Database Storage Growth

Increase space for:

- MySQL
- PostgreSQL
- MongoDB

without downtime.

---

## Kubernetes Worker Nodes

Expand node storage for:

- Container images
- Logs
- Persistent data

---

## Log Management Servers

Increase `/var` space for:

- ELK Stack
- Splunk
- Fluentd
- Filebeat

---

# Best Practices

✅ Take EBS Snapshot before resizing

✅ Verify Volume Group free space

✅ Extend Logical Volumes before filesystems

✅ Verify filesystem type (XFS vs EXT4)

✅ Monitor disk utilization

✅ Resize during maintenance windows

---

# Common Mistakes

❌ Expanding EBS but forgetting partition resize

❌ Extending LVM but not filesystem

❌ Running EXT4 commands on XFS

❌ Not taking backups before changes

---

# Benefits of Online EBS Expansion

- No server rebuild
- Minimal downtime
- Flexible storage growth
- Cost efficient
- Production-friendly

---

# Why This Procedure Is Important

Storage expansion is a common operational task in cloud environments.

Understanding:

- EBS Volumes
- Partitions
- LVM
- Filesystems

is essential for:

- DevOps Engineers
- Linux Administrators
- Platform Engineers
- Cloud Engineers
- Site Reliability Engineers (SRE)