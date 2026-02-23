# Network Configuration

Brief description: Configure bridged network setup for Proxmox VE on Hetzner.

## What You'll Learn

- How Hetzner's IP/MAC binding works
- How to configure bridged network setup

## Prerequisites

- [ ] Proxmox VE installed and accessible via web UI
- [ ] Server IP address and gateway from Hetzner
- [ ] Additional IPs or subnets (if applicable)
- [ ] SSH access to Proxmox host

## Estimated Time

15-20 minutes

## Understanding Hetzner Network

Hetzner has strict IP/MAC binding for security. Key points:

- **Main IP** and **additional single IPs** are bound to the server's main MAC address
- **Additional subnets** (except failover) can use virtual MAC addresses
- **Failover IPs** and **failover subnets** require routed setup
- Virtual MAC addresses can only be generated for single additional IPs in Robot Panel

---

## Bridged Setup

In bridged mode, the host acts as a transparent bridge. VMs get their own MAC address and appear directly on the network.

### Host Configuration

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto enp0s31f6
iface enp0s31f6 inet manual

auto vmbr0
iface vmbr0 inet static
        address 198.51.100.10/32
        gateway 198.51.100.1
        bridge-ports enp0s31f6
        bridge-stp off
        bridge-fd 0
```

### Guest VM Configuration

#### Static configuration:

```bash
# /etc/network/interfaces

auto ens18
iface ens18 inet static
        address 192.0.2.20/32
        gateway 192.0.2.1
```

#### DHCP configuration:

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto ens18
iface ens18 inet dhcp
        hwaddress ether aa:bb:cc:dd:ee:ff
```

Use the virtual MAC address generated in the Robot Panel.

For LXC containers, configure via Proxmox GUI:
1. Select container → Network
2. Click on the bridge
3. Select DHCP and add the MAC address from Robot Panel

---

## Applying Network Changes

After making changes, apply them:

```bash
systemctl restart networking
```

!!! warning "Running VMs"
    If VMs are running when restarting networking, you must stop and start them again (not restart) to recreate their network connections.

## Verification

- [ ] Network configuration matches your setup
- [ ] Proxmox host is reachable via main IP
- [ ] VMs can communicate as expected

## Next Steps

Proceed to [VM Overview](../02-vm-creation/01-overview.md) to prepare for creating your virtual machines.
