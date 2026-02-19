# Proxmox Installation

Brief description: Download and install Proxmox VE 8.2 directly onto the ZFS pool.

## What You'll Learn

- How to download Proxmox VE 8.2 ISO
- How to boot ISO from rescue environment
- How to install Proxmox on ZFS storage
- How to access Proxmox Web UI

## Prerequisites

- [ ] ZFS pool `rpool` created and healthy
- [ ] Rescue mode SSH access active
- [ ] At least 4GB free space for ISO download

## Estimated Time

45-60 minutes

!!! warning "System Installation"
    This will install a new operating system (Proxmox VE) and replace the rescue environment. Ensure all ZFS setup is complete before proceeding.

## Step-by-Step Instructions

### Step 1: Download Proxmox VE 8.2

Navigate to temp directory:

```bash
cd /tmp
```

Download Proxmox VE 8.2 ISO:

```bash
wget https://enterprise.proxmox.com/iso/proxmox-ve_8.2-2.iso
```

Verify download (optional but recommended):

```bash
ls -lh proxmox-ve_8.2-2.iso
```

Expected size: ~1.1GB

### Step 2: Mount ISO and Prepare Boot

Create mount point:

```bash
mkdir -p /mnt/iso
mount -o loop /tmp/proxmox-ve_8.2-2.iso /mnt/iso
```

Verify mount:

```bash
ls /mnt/iso
```

Expected output:
```
boot  copyright  EULA  proxmox  pve-base.squashfs  pve-installer.squashfs
```

### Step 3: Copy Boot Files

Copy kernel and initrd:

```bash
cp /mnt/iso/boot/linux26 /boot/linux26
cp /mnt/iso/boot/initrd.img /boot/initrd.img
```

### Step 4: Configure GRUB for ISO Boot

Create custom GRUB entry:

```bash
cat > /boot/grub/custom.cfg << 'GRUBCFG'
menuentry "Proxmox VE Installer" {
    linux /boot/linux26 vga=791 video=vesafb:ywrap,mtrr ramdisk_size=16777216 rw quiet
    initrd /boot/initrd.img
}
GRUBCFG
```

Update GRUB configuration:

```bash
update-grub
```

### Step 5: Prepare Installation Target

Set target for Proxmox installation:

```bash
echo "rpool" > /boot/proxmox-install-target
```

### Step 6: Reboot into Installer

!!! caution "Connection Loss"
    This reboot will disconnect your SSH session. The server will boot into the Proxmox installer. You'll need to access the installer via VNC or wait and access the web interface after installation.

Reboot the server:

```bash
reboot
```

Your SSH session will disconnect. Wait 5 minutes for the installation to complete.

### Step 7: Access Proxmox Web UI

After 5 minutes, access the Proxmox Web UI:

```
https://<your-server-ip>:8006
```

!!! danger "SSL Certificate Warning"
    You will see a browser warning about an untrusted certificate. This is normal for the initial installation. Click "Advanced" and proceed.

Login credentials:
- **Username**: root
- **Password**: (set during installation, or rescue password if auto-configured)

### Step 8: Initial Proxmox Configuration

**Configure Repositories:**

1. Click on your node name (left sidebar)
2. Go to **Shell**
3. Disable enterprise repo:

```bash
nano /etc/apt/sources.list.d/pve-enterprise.list
```

Comment out the line:
```
# deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```

Save (Ctrl+O, Enter, Ctrl+X)

4. Add no-subscription repository:

```bash
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" >> /etc/apt/sources.list
```

5. Update system:

```bash
apt update && apt full-upgrade -y
```

### Step 9: Verify ZFS Integration

1. In Proxmox Web UI, click **Datacenter** → **Storage**
2. Verify `rpool` is listed with correct size (~460GB)
3. Check **Disks** section to see both NVMe drives

Test ZFS functionality:

```bash
zpool status
```

Should show both drives ONLINE in mirror configuration.

## Verification

- [ ] Proxmox VE 8.2 ISO downloaded
- [ ] GRUB configured for ISO boot
- [ ] Server rebooted and Proxmox installed
- [ ] Web UI accessible on port 8006
- [ ] Successfully logged in as root
- [ ] Enterprise repository disabled
- [ ] No-subscription repository added
- [ ] System updated
- [ ] ZFS pool visible in Proxmox storage

## Post-Installation Tasks

### Configure Network (if needed)

Check network configuration:

```bash
cat /etc/network/interfaces
```

Verify IP address:

```bash
ip addr show
```

### Set Correct Timezone

```bash
dpkg-reconfigure tzdata
```

Select your timezone from the menu.

### Verify Subscription Status (Optional)

Without a subscription, you'll see a notification. This is normal and doesn't affect functionality.

## Common Issues

### Issue: Cannot access Web UI
**Solution**: 
1. Verify server is pingable: `ping <server-ip>`
2. Check if port 8006 is open: `nmap -p 8006 <server-ip>`
3. Try accessing via IP instead of hostname
4. Clear browser cache or use incognito mode

### Issue: ZFS pool not visible
**Solution**: 
```bash
zpool import rpool
systemctl restart pvedaemon
```

### Issue: Installation hangs
**Solution**: 
- Check server hardware via Hetzner Robot
- Try rescue mode again and verify ZFS pool is healthy
- Contact Hetzner support if hardware issues suspected

## Next Steps

Proceed to [VM Overview](../02-vm-creation/01-overview.md) to prepare for creating your six virtual machines.
