# MkDocs VM Creation (VM-104)

Brief description: Create VM-104 for MkDocs documentation server with 1 vCPU, 2GB RAM, and 10GB storage.

## VM Specifications

- **VM ID**: 104
- **Name**: mkdocs
- **CPU**: 1 vCPU
- **RAM**: 2 GiB
- **Disk**: 10 GiB (ZFS)
- **OS**: Ubuntu Server 24.04 LTS
- **Network**: vmbr0
- **Purpose**: Documentation server (MkDocs)

## Prerequisites

- [ ] Proxmox Web UI accessible
- [ ] Ubuntu 24.04 LTS ISO available
- [ ] ZFS storage pool available

## Step-by-Step Instructions

### Step 1: Create VM

1. **Create VM** in Proxmox UI

### Step 2: General

- **VM ID**: 104
- **Name**: mkdocs

Click **Next**

### Step 3: OS

- **ISO image**: ubuntu-24.04-live-server-amd64.iso
- **Type**: Linux
- **Version**: 6.x - 2.6 Kernel

Click **Next**

### Step 4: System

Accept defaults

Click **Next**

### Step 5: Disks

- **Storage**: rpool
- **Disk size**: 10 GiB
- **Format**: qcow2

Click **Next**

### Step 6: CPU

- **Cores**: 1

Click **Next**

### Step 7: Memory

- **Memory**: 2048 MiB

Click **Next**

### Step 8: Network

- **Bridge**: vmbr0
- **Model**: VirtIO

Click **Next**

### Step 9: Confirm

Review and click **Finish**

### Step 10: Install Ubuntu

1. Start VM 104
2. Open Console
3. Install Ubuntu:

**Network**: Record IP address

**Profile**:
- Server name: mkdocs
- Username: admin
- Password: [strong password]

**SSH**: Install OpenSSH server

**Complete** installation

### Step 11: Post-Installation

```bash
# Update
sudo apt update && sudo apt upgrade -y

# Install Python and Git
sudo apt install -y python3 python3-pip python3-venv git

# Check versions
python3 --version
pip3 --version
git --version
```

### Step 12: Document IP

Add to inventory:
```
mkdocs | 104 | 10.0.0.104 | docs.example.com | MkDocs Port 8000
```

## Verification

- [ ] VM 104 created
- [ ] Ubuntu installed
- [ ] Python and Git installed
- [ ] IP address recorded

## Next Steps

- [LDAP VM](06-ldap-vm.md) - VM-105
- Or [MkDocs Setup Guide](../06-maintenance/03-doc-update.md)
