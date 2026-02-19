# Nextcloud Installation

Brief description: Install Nextcloud 28.x on VM-102 using Snap for quick deployment.

## What You'll Learn

- How to install Nextcloud via Snap
- How to configure trusted domains
- How to verify the installation

## Prerequisites

- [ ] VM-102 created and Ubuntu 24.04 LTS installed
- [ ] IP address assigned and documented
- [ ] SSH access to VM-102 working
- [ ] 6GB RAM and 50GB disk available

## Estimated Time

3-4 hours

## Step-by-Step Instructions

### Step 1: SSH to Nextcloud VM

```bash
ssh admin@<nextcloud-ip>
```

### Step 2: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3: Install Snap

```bash
sudo apt install -y snapd
```

Verify snap is installed:

```bash
snap version
```

### Step 4: Install Nextcloud

```bash
sudo snap install nextcloud
```

This will take 10-15 minutes. Monitor progress:

```bash
sudo snap changes
```

Wait until status shows "Done".

### Step 5: Configure Trusted Domains

Add your internal IP:

```bash
sudo nextcloud.occ config:system:set trusted_domains 1 --value=<nextcloud-ip>
```

Add your public domain (for later):

```bash
sudo nextcloud.occ config:system:set trusted_domains 2 --value=cloud.example.com
```

Verify configuration:

```bash
sudo nextcloud.occ config:system:get trusted_domains
```

Should show:
```
localhost
<nextcloud-ip>
cloud.example.com
```

### Step 6: Create Admin Account

```bash
sudo nextcloud.manual-install admin <strong-password>
```

Replace `<strong-password>` with a secure password.

### Step 7: Verify Installation

Check Nextcloud status:

```bash
sudo nextcloud.occ status
```

Expected output:
```
  - installed: true
  - version: 28.0.x.x
  - versionstring: 28.0.x
  - edition: 
```

### Step 8: Configure Background Jobs

Set cron for background tasks:

```bash
sudo nextcloud.occ background:cron
```

Add cron job:

```bash
echo "*/5 * * * * root /snap/bin/nextcloud.occ background:cron" | sudo tee /etc/cron.d/nextcloud
```

### Step 9: Configure Memory Caching

Enable APCu for better performance:

```bash
sudo nextcloud.occ config:system:set memcache.local --value='\OC\Memcache\APCu'
```

### Step 10: Adjust PHP Memory Limit

```bash
sudo snap set nextcloud php.memory-limit=512M
```

### Step 11: Test Web Access

1. Open browser: `http://<nextcloud-ip>`

2. Login with:
   - Username: admin
   - Password: [password you set in Step 6]

3. You should see the Nextcloud dashboard

### Step 12: Initial Configuration

**Configure Email** (optional):
- Settings → Administration → Basic settings
- Email server settings

**Enable Apps**:
- Apps → Your apps
- Enable useful apps:
  - Calendar
  - Contacts
  - Tasks
  - Notes

**Configure Sharing**:
- Settings → Administration → Sharing
- Set appropriate sharing policies

### Step 13: Create Backup Script

```bash
cat > ~/backup-nextcloud.sh << 'EOF'
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$HOME/backups"
mkdir -p $BACKUP_DIR

# Enable maintenance mode
sudo nextcloud.occ maintenance:mode --on

# Backup data directory
tar -czf $BACKUP_DIR/nextcloud_data_$TIMESTAMP.tar.gz /var/snap/nextcloud/common/nextcloud/data

# Export database
sudo nextcloud.export > $BACKUP_DIR/nextcloud_db_$TIMESTAMP.sql

# Disable maintenance mode
sudo nextcloud.occ maintenance:mode --off

# Keep only last 7 backups
ls -t $BACKUP_DIR/*.tar.gz | tail -n +8 | xargs -r rm
ls -t $BACKUP_DIR/*.sql | tail -n +8 | xargs -r rm

echo "Backup completed: $TIMESTAMP"
