# ZFS Setup and Configuration

Brief description: Create a mirrored ZFS storage pool on your NVMe drives for Proxmox installation.

## What You'll Learn

- How to install ZFS utilities
- How to create a mirrored ZFS pool
- How to verify ZFS pool health

## Prerequisites

- [ ] Rescue mode activated and SSH connected
- [ ] Both NVMe devices identified (`/dev/nvme0n1` and `/dev/nvme1n1`)
- [ ] Root access in rescue environment

## Estimated Time

20-30 minutes

!!! danger "Data Destruction Warning"
    Creating a ZFS pool will **DESTROY ALL DATA** on the selected drives. Ensure you have no important data on these drives. This is a fresh server setup.

## Step-by-Step Instructions

### Step 1: Install ZFS Utilities

Install ZFS support packages:

```bash
apt install -y zfsutils-linux
```

Verify installation:

```bash
zpool --version
```

Expected output:
```
zfs-2.2.x-x
zfs-kmod-2.2.x-x
```

### Step 2: Identify Devices

Double-check your NVMe devices:

```bash
lsblk -d -o NAME,SIZE,TYPE,MODEL
```

Expected output:
```
NAME     SIZE TYPE MODEL
nvme0n1 476.9G disk NVMe Device
nvme1n1 476.9G disk NVMe Device
```

### Step 3: Create Mirrored ZFS Pool

Create the pool named `rpool` with mirroring:

```bash
zpool create rpool mirror /dev/nvme0n1 /dev/nvme1n1
```

Expected output: (no output on success)

Verify pool creation:

```bash
zpool status
```

Expected output:
```
  pool: rpool
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	rpool       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    nvme0n1 ONLINE       0     0     0
	    nvme1n1 ONLINE       0     0     0

errors: No known data errors
```

### Step 4: Configure ZFS Properties

Set recommended properties for Proxmox:

```bash
# Enable compression
zfs set compression=lz4 rpool

# Disable access time updates for performance
zfs set atime=off rpool

# Set reasonable recordsize for VMs
zfs set recordsize=64k rpool
```

Verify settings:

```bash
zfs get compression,atime,recordsize rpool
```

### Step 5: Create Proxmox Dataset Structure

Create datasets for Proxmox:

```bash
zfs create rpool/data
zfs create rpool/iso
zfs create rpool/template
```

List all datasets:

```bash
zfs list
```

Expected output:
```
NAME               USED  AVAIL     REFER  MOUNTPOINT
rpool              912K  460G       96K  /rpool
rpool/data          96K  460G       96K  /rpool/data
rpool/iso           96K  460G       96K  /rpool/iso
rpool/template      96K  460G       96K  /rpool/template
```

### Step 6: Test ZFS Functionality

Test pool scrub (data integrity check):

```bash
zpool scrub rpool
```

Check scrub status:

```bash
zpool status rpool
```

Wait for scrub to complete (should show "scrub: none requested" or completed status).

## Verification

- [ ] ZFS utilities installed
- [ ] Mirrored pool `rpool` created successfully
- [ ] Both NVMe devices showing ONLINE status
- [ ] ZFS properties configured (compression, atime, recordsize)
- [ ] Proxmox datasets created
- [ ] Pool scrub completed without errors

## Common Issues

### Issue: Device is busy
**Solution**: Ensure drives are not mounted:
```bash
umount /dev/nvme0n1* 2>/dev/null
umount /dev/nvme1n1* 2>/dev/null
zpool create rpool mirror /dev/nvme0n1 /dev/nvme1n1
```

### Issue: Pool already exists
**Solution**: If you need to recreate:
```bash
zpool destroy rpool
zpool create rpool mirror /dev/nvme0n1 /dev/nvme1n1
```

### Issue: One disk shows UNAVAIL
**Solution**: Check physical connections. Contact Hetzner support if hardware issue suspected.

## Next Steps

Proceed to [Proxmox Installation](03-proxmox-install.md) to install Proxmox on your ZFS pool.
