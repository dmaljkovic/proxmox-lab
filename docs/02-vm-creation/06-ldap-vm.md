# OpenLDAP VM Creation (VM-105)

Brief description: Create VM-105 for OpenLDAP directory server with 1 vCPU, 2GB RAM, and 10GB storage.

## VM Specifications

- **VM ID**: 105
- **Name**: openldap
- **CPU**: 1 vCPU
- **RAM**: 2 GiB
- **Disk**: 10 GiB (ZFS)
- **OS**: Ubuntu Server 24.04 LTS
- **Network**: vmbr0
- **Purpose**: LDAP directory services

## Prerequisites

- [ ] Proxmox Web UI accessible
- [ ] Ubuntu 24.04 LTS ISO available
- [ ] ZFS storage pool available

## Step-by-Step Instructions

### Step 1: Create VM

Click **Create VM** in Proxmox

### Step 2: General

- **VM ID**: 105
- **Name**: openldap

Click **Next**

### Step 3: OS

- **ISO image**: ubuntu-24.04-live-server-amd64.iso

Click **Next**

### Step 4: System

Accept defaults

Click **Next**

### Step 5: Disks

- **Storage**: rpool
- **Disk size**: 10 GiB

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

Click **Finish**

### Step 10: Install Ubuntu

1. Start VM 105
2. Open Console
3. Install Ubuntu:

**Network**: Record IP

**Profile**:
- Server name: openldap
- Username: admin
- Password: [strong password]

**SSH**: Install OpenSSH server

**Complete** installation

### Step 11: Post-Installation

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install LDAP utilities
sudo apt install -y ldap-utils
```

### Step 12: Document IP

Add to inventory:
```
openldap | 105 | 10.0.0.105 | ldap.example.com | LDAP Port 389/636
```

## Verification

- [ ] VM 105 created
- [ ] Ubuntu installed
- [ ] IP address assigned
- [ ] System updated

## Security Note

!!! warning "LDAP Security"
    This VM will store user credentials. Ensure proper firewall rules are configured (only Keycloak should access LDAP ports).

## Next Steps

- [Keycloak VM](07-keycloak-vm.md) - VM-106
- Or [OpenLDAP Installation](../03-software/04-keycloak-oidc.md)
