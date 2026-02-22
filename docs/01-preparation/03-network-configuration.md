# Network Configuration

Brief description: Configure your network setup for Proxmox VE on Hetzner. This covers routed, bridged, vSwitch, cloud network, and NAT configurations.

## What You'll Learn

- How Hetzner's IP/MAC binding works
- How to configure different network setups
- How to enable IP forwarding for routed setups
- How to set up NAT/masquerading for VMs

## Prerequisites

- [ ] Proxmox VE installed and accessible via web UI
- [ ] Server IP address and gateway from Hetzner
- [ ] Additional IPs or subnets (if applicable)
- [ ] SSH access to Proxmox host

## Estimated Time

30-45 minutes

## Understanding Hetzner Network

Hetzner has strict IP/MAC binding for security. Key points:

- **Main IP** and **additional single IPs** are bound to the server's main MAC address
- **Additional subnets** (except failover) can use virtual MAC addresses
- **Failover IPs** and **failover subnets** require routed setup
- Virtual MAC addresses can only be generated for single additional IPs in Robot Panel

!!! warning "Bridged Setup Warning"
    In bridged mode, packets will have the VM's MAC address. Without a virtual MAC, traffic may be flagged as abuse and could lead to server blocking.

## IP Forwarding

For routed setups, enable IP forwarding:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
```

Apply permanently:

```bash
systemctl restart systemd-sysctl
```

Verify:

```bash
sysctl net.ipv4.ip_forward
sysctl net.ipv6.conf.all.forwarding
```

## Network Configuration Options

Choose one of the following setups:

- [Routed Setup](#1-routed-setup)
- [Bridged Setup](#2-bridged-setup)
- [vSwitch with Public Subnet](#3-vswitch-with-public-subnet)
- [Hetzner Cloud Network](#4-hetzner-cloud-network)
- [Masquerading (NAT)](#5-masquerading-nat)

---

## 1. Routed Setup

In routed configuration, the host bridge IP serves as the gateway. Use `/32` mask for additional IPs not in the same subnet.

### Example Network Data

Replace these with your actual values:
- Main IP: `198.51.100.10/24`
- Gateway: `198.51.100.1`
- Additional Subnet: `203.0.113.0/24`
- Additional single IP (foreign subnet): `192.0.2.20/24`
- Additional single IP (same subnet): `198.51.100.30/24`
- Failover IP: `172.17.234.100/32`
- IPv6: `2001:DB8::/64`

### Host Configuration

Edit `/etc/network/interfaces`:

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

iface lo inet6 loopback

auto enp0s31f6
iface enp0s31f6 inet static
        address 198.51.100.10/32
        gateway 198.51.100.1

        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up echo 1 > /proc/sys/net/ipv6/conf/all/forwarding

# IPv6 for main interface
iface enp0s31f6 inet6 static
        address 2001:db8::2/128
        gateway fe80::1

# Bridge for single IPs
auto vmbr0
iface vmbr0 inet static
        address 198.51.100.10/32
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up ip route add 192.0.2.20/32 dev vmbr0     # Foreign subnet IP
        post-up ip route add 198.51.100.30/32 dev vmbr0  # Same subnet IP
        post-up ip route add 172.17.234.100/32 dev vmbr0 # Failover IP

# IPv6 for bridge
iface vmbr0 inet6 static
       address 2001:db8::3/64

# Additional Subnet
auto vmbr1
iface vmbr1 inet static
        address 203.0.113.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

# Failover Subnet
auto vmbr2
iface vmbr2 inet static
        address 172.17.234.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
```

!!! warning "Network Restart"
    Restart networking after applying changes. If VMs are running, stop and start them again (not restart) to recreate point-to-point connections.

Apply changes:

```bash
systemctl restart networking
```

### Guest VM Configuration (Debian/Ubuntu)

#### Additional IP from same subnet:

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto ens18
iface ens18 inet static
        address 198.51.100.30/32
        gateway 198.51.100.10

# IPv6
iface ens18 inet6 static
        address 2001:DB8::4/64
        gateway 2001:DB8::3
```

#### Additional IP from foreign subnet:

```bash
auto ens18
iface ens18 inet static
        address 192.0.2.20/32
        gateway 198.51.100.10
```

#### IP from additional subnet:

```bash
auto ens18
iface ens18 inet static
        address 203.0.113.10/24
        gateway 203.0.113.1
```

#### IP from failover subnet:

```bash
auto ens18
iface ens18 inet static
        address 172.17.234.10/24
        gateway 172.17.234.1
```

#### Single failover IP:

```bash
auto ens18
iface ens18 inet static
        address 172.17.234.100/32
        gateway 198.51.100.10
```

---

## 2. Bridged Setup

In bridged mode, the host acts as a transparent bridge. You **must** request virtual MAC addresses for each additional IP in the Robot Panel.

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

## 3. vSwitch with Public Subnet

Connect VMs directly to a Hetzner vSwitch with a public subnet. VLAN tagging is handled automatically.

Example subnet: `203.0.113.0/24`
Example VLAN ID: `4009`

### Host Configuration

```bash
# /etc/network/interfaces

auto enp0s31f6.4009
iface enp0s31f6.4009 inet manual

auto vmbr4009
iface vmbr4009 inet static
        bridge-ports enp0s31f6.4009
        bridge-stp off
        bridge-fd 0
        mtu 1400
# vSwitch Subnet 203.0.113.0/24
```

### Guest VM Configuration

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto ens18
iface ens18 inet static
        address 203.0.113.2/24
        gateway 203.0.113.1
```

---

## 4. Hetzner Cloud Network

Connect VMs to a Hetzner Cloud Network for private networking.

Example configuration:
- Cloud Network: `192.168.0.0/16`
- Cloud Subnet: `192.168.0.0/24`
- vSwitch: `192.168.1.0/24`
- VLAN ID: `4000`

### Host Configuration

```bash
# /etc/network/interfaces

auto enp0s31f6.4000
iface enp0s31f6.4000 inet manual

auto vmbr4000
iface vmbr4000 inet static
        address 192.168.1.10/24
        bridge-ports enp0s31f6.4000
        bridge-stp off
        bridge-fd 0
        mtu 1400
        post-up ip route add 192.168.0.0/16 via 192.168.1.1 dev vmbr4000
# vSwitch to Cloud Network
```

### Guest VM Configuration

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto ens18
iface ens18 inet static
        address 192.168.1.2/24
        gateway 192.168.1.1
```

---

## 5. Masquerading (NAT)

Use NAT when you don't have additional public IPs. All VMs use the host's public IP for outbound traffic.

### Host Configuration

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

iface lo inet6 loopback

auto enp0s31f6
iface enp0s31f6 inet static
        address 198.51.100.10/24
        gateway 198.51.100.1
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward

auto vmbr4
iface vmbr4 inet static
        address 172.16.16.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   iptables -t nat -A POSTROUTING -s '172.16.16.0/24' -o enp0s31f6 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '172.16.16.0/24' -o enp0s31f6 -j MASQUERADE
```

### Optional: Port Forwarding

This forwards incoming traffic (except ports 22 and 8006) to a specific VM:

```bash
post-up iptables -t nat -A PREROUTING -i enp0s31f6 -p tcp -m multiport ! --dports 22,8006 -j DNAT --to 172.16.16.2
post-down iptables -t nat -D PREROUTING -i enp0s31f6 -p tcp -m multiport ! --dports 22,8006 -j DNAT --to 172.16.16.2
```

### Guest VM Configuration

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto ens18
iface ens18 inet static
        address 172.16.16.2/24
        gateway 172.16.16.1
```

---

## Applying Network Changes

After making changes, apply them:

```bash
systemctl restart networking
```

!!! warning "Running VMs"
    If VMs are running when restarting networking, you must stop and start them again (not restart) to recreate their network connections.

## Verification

- [ ] Network configuration matches your setup type
- [ ] Proxmox host is reachable via main IP
- [ ] VMs can communicate as expected
- [ ] NAT/forwarding works correctly (if applicable)

## Security Recommendations

Consider these additional security measures:

- [Two-Factor Authentication for Proxmox](https://pve.proxmox.com/wiki/Two-Factor_Authentication)
- [Fail2ban against Brute-force attacks](https://pve.proxmox.com/wiki/Fail2ban)
- [Securing SSH Service](https://community.hetzner.com/tutorials/securing_ssh)

## Next Steps

Proceed to [VM Overview](../02-vm-creation/01-overview.md) to prepare for creating your virtual machines.
