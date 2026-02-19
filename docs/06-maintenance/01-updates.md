# System Updates

Brief description: Procedures for keeping all systems updated and secure.

## Proxmox Host Updates

```bash
# Update package lists
apt update

# Check for upgrades
apt list --upgradable

# Perform upgrade
apt full-upgrade -y

# Check if reboot required
if [ -f /var/run/reboot-required ]; then
    echo "Reboot required"
fi
```

## VM Updates

Script to update all VMs:

```bash
for ip in <rocketchat-ip> <nextcloud-ip> <nginx-ip> <mkdocs-ip> <ldap-ip> <keycloak-ip>; do
    echo "Updating $ip..."
    ssh admin@$ip "sudo apt update && sudo apt upgrade -y"
done
```

## Docker Updates (Rocket.Chat)

```bash
ssh admin@<rocketchat-ip>
cd ~/rocketchat
docker compose pull
docker compose up -d
```

## Snap Updates (Nextcloud)

```bash
ssh admin@<nextcloud-ip>
sudo snap refresh nextcloud
```

## Verification

- [ ] Update procedures documented
- [ ] Scripts created
- [ ] Tested on one VM

## Schedule

- **Security updates**: Weekly
- **Regular updates**: Monthly
- **Major upgrades**: Quarterly

