# Nginx Proxy VM Creation (VM-103)

Brief description: Create VM-103 for Nginx reverse proxy with 1 vCPU, 2GB RAM, and 10GB storage.

## VM Specifications

- **VM ID**: 103
- **Name**: nginx-proxy
- **CPU**: 1 vCPU
- **RAM**: 2 GiB
- **Disk**: 10 GiB (ZFS)
- **OS**: Ubuntu Server 24.04 LTS
- **Network**: vmbr0
- **Purpose**: Reverse proxy and SSL termination

## Prerequisites

- [ ] Proxmox Web UI accessible
- [ ] Ubuntu 24.04 LTS ISO available
- [ ] ZFS storage pool available
- [ ] IP address planned

## Step-by-Step Instructions

### Step 1: Create VM in Proxmox

1. Click **Create VM** in Proxmox UI

### Step 2: General Tab

- **VM ID**: 103
- **Name**: nginx-proxy

Click **Next**

### Step 3: OS Tab

- **ISO image**: ubuntu-24.04-live-server-amd64.iso
- **Type**: Linux
- **Version**: 6.x - 2.6 Kernel

Click **Next**

### Step 4: System Tab

Accept defaults

Click **Next**

### Step 5: Disks Tab

- **Storage**: rpool
- **Disk size (GiB)**: 10
- **Format**: qcow2
- **Cache**: Write back

!!! note "Minimal Storage"
    Nginx proxy requires minimal storage as it only handles configuration files and certificates. 10GB is sufficient.

Click **Next**

### Step 6: CPU Tab

- **Sockets**: 1
- **Cores**: 1

Click **Next**

### Step 7: Memory Tab

- **Memory (MiB)**: 2048

Click **Next**

### Step 8: Network Tab

- **Bridge**: vmbr0
- **Model**: VirtIO

Click **Next**

### Step 9: Confirm

Review:
- Name: nginx-proxy
- VM ID: 103
- CPU: 1 core
- Memory: 2048 MiB
- Disk: 10 GiB

Click **Finish**

### Step 10: Install Ubuntu

1. Start VM 103
2. Open Console
3. Boot from ISO
4. Follow installation wizard:

**Network**: Record assigned IP

**Profile**:
- Server name: nginx-proxy
- Username: admin
- Password: [strong password]

**SSH**: Install OpenSSH server

**Complete** and reboot

### Step 11: Post-Installation

After reboot:

```bash
# Check IP
ip addr show

# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl vim htop
```

### Step 12: Document IP

Add to inventory:
```
nginx-proxy | 103 | 10.0.0.103 | proxy.example.com | Nginx Port 80/443
```

## Verification

- [ ] VM 103 created with correct specs
- [ ] Ubuntu installed
- [ ] IP address assigned and recorded
- [ ] System updated
- [ ] SSH accessible

## Special Configuration

### Enable IP Forwarding

For proper reverse proxy functionality:

```bash
# Edit sysctl
sudo nano /etc/sysctl.conf
```

Uncomment or add:
```
net.ipv4.ip_forward=1
```

Apply:
```bash
sudo sysctl -p
```

## Common Issues

### Issue: Low resource warnings
**Solution**: Nginx proxy runs fine with 1 vCPU and 2GB RAM. These warnings can be ignored.

### Issue: Cannot reach backend services
**Solution**: Ensure IP forwarding is enabled and firewall allows forwarding.

## Next Steps

- [MkDocs VM](05-mkdocs-vm.md) - VM-104
- Or [Nginx Installation](../03-software/03-nginx-reverse-proxy.md)
