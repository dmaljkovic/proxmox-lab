# Network Configuration

Brief description: Configure bridged network setup for Proxmox VE on Hetzner with public and private networks.

## What You'll Learn

- How Hetzner's IP/MAC binding works
- How to configure bridged network setup with public IP
- How to set up private internal network with NAT

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

## Network Configuration

Edit `/etc/network/interfaces` on the Proxmox host:

```bash
nano /etc/network/interfaces
```

Add the following configuration:

```bash
# Loopback
auto lo
iface lo inet loopback
iface lo inet6 loopback

# Physical Interface (replace enp5s0 with your actual interface)
iface enp5s0 inet manual

# Include other network configurations
source /etc/network/interfaces.d/*

# Proxmox Bridge - Public Network
auto vmbr0
iface vmbr0 inet static
      address      YOUR_IP/26
      gateway      YOUR_GATEWAY
      bridge-ports YOUR_INTERFACE
      bridge-stp   off
      bridge-fd    0
      up           sysctl -p

# Proxmox Bridge - Private Internal Network (NAT)
auto vmbr2
iface vmbr2 inet static
      address      192.168.192.5/18
      bridge-ports none
      bridge-stp   off
      bridge-fd    0
      post-up      iptables -t nat -A POSTROUTING -s '192.168.192.0/18' -o vmbr0 -j MASQUERADE
      post-down    iptables -t nat -D POSTROUTING -s '192.168.192.0/18' -o vmbr0 -j MASQUERADE
      post-up      iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
      post-down    iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
```

### Configuration Values

Replace the following placeholders with your actual values:

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `YOUR_IP/26` | Your server's public IP with subnet mask | `192.168.1.100/26` |
| `YOUR_GATEWAY` | Your server's gateway IP | `192.168.1.1` |
| `YOUR_INTERFACE` | Your physical network interface | `enp5s0` |

!!! warning "Verify Interface Name"
    Use `ip addr` or `predict-check` to identify your actual network interface name before configuring.

---

## Network Explanation

### vmbr0 - Public Bridge

- Connects to the physical network interface
- Handles all public traffic
- VMs attached to this bridge get direct network access
- Uses the main IP and MAC from Hetzner

### vmbr2 - Private Internal Network

- Internal-only bridge with no physical port
- Provides NAT (Network Address Translation) to vmbr0
- IP range: `192.168.192.0/18` (192.168.192.1 - 192.168.255.254)
- Use this for internal-only VMs that don't need public IPs

---

## Applying Network Changes

After making changes, apply them:

```bash
systemctl restart networking
```

!!! warning "Running VMs"
    If VMs are running when restarting networking, you must stop and start them again (not restart) to recreate their network connections.

---

## Port Forwarding for Nginx Proxy

Since all VMs are on vmbr2 (private network), we need to forward ports 80 and 443 from vmbr0 (public) to the nginx-proxy VM.

### Create Port Forwarding Script

Create a script to set up port forwarding:

```bash
sudo nano /etc/network/if-pre-up.d/iptables-nat-forward
```

Add the following content:

```bash
#!/bin/sh
/sbin/iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to-destination 192.168.192.20:80
/sbin/iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 443 -j DNAT --to-destination 192.168.192.20:443
/sbin/iptables -A FORWARD -p tcp -d 192.168.192.20 --dport 80 -j ACCEPT
/sbin/iptables -A FORWARD -p tcp -d 192.168.192.20 --dport 443 -j ACCEPT
```

Make it executable:

```bash
sudo chmod +x /etc/network/if-pre-up.d/iptables-nat-forward
```

Apply the rules immediately:

```bash
/etc/network/if-pre-up.d/iptables-nat-forward
```

!!! note "Nginx Proxy IP"
    The nginx-proxy VM should have IP 192.168.192.20. If you use a different IP, update the rules accordingly.

---

## VM Network Configuration

### For VMs with Public IPs (attach to vmbr0)

Configure via Proxmox GUI:
1. Create/edit VM → Hardware → Network Device
2. Select **Bridge: vmbr0**
3. Add MAC address from Hetzner Robot (for additional IPs)

### For VMs with Internal IPs only (attach to vmbr2)

```bash
# Inside the VM, configure static IP
auto ens18
iface ens18 inet static
        address 192.168.192.100/18
        gateway 192.168.192.5
```

---

## Verification

- [ ] Network configuration matches your setup
- [ ] Proxmox host is reachable via main IP
- [ ] vmbr0 bridge is active
- [ ] vmbr2 bridge is active with NAT
- [ ] VMs can communicate as expected
- [ ] Internal VMs can reach the internet via NAT

## Next Steps

Proceed to [VM Overview](../02-vm-creation/01-overview.md) to prepare for creating your virtual machines.
