# Rocket.Chat VM Creation (VM-101)

Brief description: Create VM-101 for Rocket.Chat with 2 vCPU, 4GB RAM, and 30GB storage.

## VM Specifications

- **VM ID**: 101
- **Name**: rocketchat
- **CPU**: 2 vCPU
- **RAM**: 4 GiB
- **Disk**: 30 GiB (ZFS)
- **OS**: Ubuntu Server 24.04 LTS
- **Network**: vmbr0
- **Purpose**: Rocket.Chat server

## Prerequisites

- [ ] Proxmox Web UI accessible
- [ ] Ubuntu 24.04 LTS ISO uploaded
- [ ] ZFS storage pool available
- [ ] IP address planned from overview

## Step-by-Step Instructions

### Step 1: Create VM in Proxmox

1. Log into Proxmox Web UI at `https://<proxmox-ip>:8006`
2. Click on your node in the left sidebar
3. Click **Create VM** button (top right)

### Step 2: General Tab

- **Node**: (your node name)
- **VM ID**: 101
- **Name**: rocketchat
- **Start at boot**: (optional, can enable later)

Click **Next**

### Step 3: OS Tab

- **Storage**: local
- **ISO image**: ubuntu-24.04-live-server-amd64.iso
- **Type**: Linux
- **Version**: 6.x - 2.6 Kernel

Click **Next**

### Step 4: System Tab

- **Graphic card**: Default (SPICE)
- **Machine**: Default (i440fx)
- **BIOS**: Default (SeaBIOS)
- **SCSI Controller**: VirtIO SCSI
- **Qemu Agent**: (optional, can enable later)

Click **Next**

### Step 5: Disks Tab

- **Storage**: rpool
- **Disk size (GiB)**: 30
- **Format**: Qemu image format (qcow2)
- **Cache**: Write back
- **Discard**: (optional, for SSD optimization)

Click **Next**

### Step 6: CPU Tab

- **Sockets**: 1
- **Cores**: 2
- **Type**: Default (kvm64)

Click **Next**

### Step 7: Memory Tab

- **Memory (MiB)**: 4096
- **Ballooning Device**: (optional)

Click **Next**

### Step 8: Network Tab

- **Bridge**: vmbr0
- **VLAN Tag**: (none)
- **Firewall**: (unchecked for now)
- **Model**: VirtIO (paravirtualized)

Click **Next**

### Step 9: Confirm Tab

Review settings:
- Name: rocketchat
- VM ID: 101
- CPU: 2 cores
- Memory: 4096 MiB
- Disk: 30 GiB on rpool
- Network: vmbr0

Click **Finish**

### Step 10: Start VM and Install Ubuntu

1. Select VM 101 from the list
2. Click **Start** button
3. Click **Console** to open VNC viewer
4. Press Enter to boot from ISO

### Step 11: Ubuntu Installation

Follow the Ubuntu Server installation wizard:

**Language**: English

**Keyboard Layout**: Your preferred layout

**Network Configuration**:
- DHCP should assign an IP automatically
- **Record this IP address** for later use
- Will be in format: 10.0.0.xxx

**Proxy**: Leave blank (unless required)

**Mirror**: Default is fine

**Storage**:
- Use entire disk
- Accept default partitioning

**Profile**:
- Your name: Administrator
- Server name: rocketchat
- Username: admin
- Password: [choose strong password]
- Confirm password

**SSH**:
- [x] Install OpenSSH server
- [ ] Import SSH identity (skip for now)

**Featured Server Snaps**:
- Skip all (we'll install manually)

**Complete Installation**:
- Wait for installation to finish
- Remove installation medium (Proxmox will handle this)
- Reboot

### Step 12: Post-Installation

After reboot:

1. VM will boot from hard disk
2. Login with your created user
3. Verify IP address:
   ```bash
   ip addr show
   ```
4. Test connectivity:
   ```bash
   ping google.com
   ```
5. Update system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### Step 13: Document IP Address

!!! important "Record IP Address"
    Write down the assigned IP address. You will need it for:
    - Nginx reverse proxy configuration
    - Monitoring setup
    - SSH access

Add to your IP inventory table:
```
rocket-chat | 101 | 10.0.0.101 | chat.example.com | Rocket.Chat Port 3000
```

## Verification

- [ ] VM 101 created with correct specifications
- [ ] Ubuntu 24.04 LTS installed
- [ ] Can login via console
- [ ] IP address assigned via DHCP
- [ ] Internet connectivity confirmed
- [ ] System updated
- [ ] IP address documented

## Post-Creation Configuration

### Disable IPv6 (Optional)

If not using IPv6:

```bash
sudo nano /etc/sysctl.conf
```

Add:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Apply:
```bash
sudo sysctl -p
```

### Set Static IP (Optional)

If you want to set a static IP instead of DHCP:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Edit to:
```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 10.0.0.101/24
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Apply:
```bash
sudo netplan apply
```

## Common Issues

### Issue: VM won't start
**Solution**: Check Proxmox logs: Datacenter → Node → Shell → `cat /var/log/syslog | grep 101`

### Issue: Cannot access console
**Solution**: Try different browser or disable browser extensions blocking WebSocket

### Issue: Network not working
**Solution**: 
```bash
sudo ip link set ens18 up
sudo dhclient ens18
```

### Issue: Installation freezes
**Solution**: Increase VM memory temporarily to 6GB during install, reduce after

## Next Steps

Proceed to:
- [Nextcloud VM](03-nextcloud-vm.md) - VM-102
- Or [Rocket.Chat Installation](../03-software/01-rocketchat.md) if all VMs created
