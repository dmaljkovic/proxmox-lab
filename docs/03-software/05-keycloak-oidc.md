# Keycloak Installation and OIDC Configuration

Brief description: Install Keycloak 26.5.5 and set up OIDC for SSO.

## What You'll Learn

- How to install Keycloak 26.5.5
- How to create OIDC clients for SSO
- How to integrate with Rocket.Chat and Nextcloud

## Prerequisites

- [ ] VM-106 created and Ubuntu 24.04 LTS installed
- [ ] OpenJDK 17 installed on Keycloak VM
- [ ] IP addresses documented
- [ ] Nginx reverse proxy configured

## Estimated Time

2-3 hours

## Part 1: Keycloak Installation

### Step 1: Update System and Install Java

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk
```

Verify:

```bash
java -version   # Should show openjdk 17.x
```

### Step 2: Download and Install Keycloak (Standalone Tarball Method)

No official APT package exists from Keycloak project → use the recommended tar.gz distribution.

```bash
cd /tmp
wget https://github.com/keycloak/keycloak/releases/download/26.5.5/keycloak-26.5.5.tar.gz
sudo tar -xzf keycloak-26.5.5.tar.gz -C /opt/
sudo mv /opt/keycloak-26.5.5 /opt/keycloak
```

### Step 3: Create Dedicated Keycloak User/Group and Set Permissions

```bash
sudo groupadd keycloak
sudo useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak

sudo chown -R keycloak:keycloak /opt/keycloak
sudo chmod -R u+rwX /opt/keycloak

sudo mkdir -p /opt/keycloak/data /var/log/keycloak
sudo chown -R keycloak:keycloak /opt/keycloak/data /var/log/keycloak
```

### Step 4: Build Optimized Image (Production Recommendation)

```bash
cd /opt/keycloak
sudo -u keycloak bin/kc.sh build
```

### Step 5: Create Systemd Service File

```bash
sudo nano /etc/systemd/system/keycloak.service
```

Paste (customize hostname, password, and internal listening):

```ini
[Unit]
Description=Keycloak Identity and Access Management
After=network.target

[Service]
Type=simple
User=keycloak
Group=keycloak
WorkingDirectory=/opt/keycloak
ExecStart=/opt/keycloak/bin/kc.sh start --optimized \
  --hostname=auth.example.com \
  --http-port=8080 \
  --http-host=0.0.0.0 \
  --proxy-headers=xforwarded \
  --log=console,file --log-file=/var/log/keycloak/keycloak.log

Restart=always
RestartSec=10

Environment="JAVA_OPTS=-Xms1024m -Xmx2048m -Djava.security.egd=file:/dev/./urandom"

# ONLY for FIRST start – remove these lines after initial setup!
Environment="KC_BOOTSTRAP_ADMIN_USERNAME=admin"
Environment="KC_BOOTSTRAP_ADMIN_PASSWORD=<very-strong-password-here>"

[Install]
WantedBy=multi-user.target
```

- `--hostname=...`: Your public domain (Keycloak enforces redirects/URIs).
- `--proxy-headers=xforwarded`: Trusts headers from Nginx proxy.

Remove the two `KC_BOOTSTRAP_ADMIN_*` lines after first successful start (edit file → sudo systemctl daemon-reload → restart).

### Step 6: Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable keycloak
sudo systemctl start keycloak
```

Check:

```bash
sudo systemctl status keycloak
sudo journalctl -u keycloak -f   # Wait for "Listening on: http://0.0.0.0:8080"
```

### Step 7: Firewall Hardening (UFW – Part of CIS Benchmarking)

Allow SSH + internal traffic only from Nginx proxy VM (adjust subnet or specific IP):

```bash
sudo ufw allow from <nginx-vm-internal-ip> to any port 8080 proto tcp
sudo ufw allow OpenSSH
sudo ufw --force enable
sudo ufw status
```

→ No public access to 8080.

### Step 8: Initial Access & Admin Console (Temporary – via Internal)

From another internal VM (or Proxmox console forwarding):

`http://<keycloak-internal-ip>:8080/admin`

Login: admin / <password from service file>

After first login → remove bootstrap env vars from service file, reload & restart.

## Part 2: OIDC Client Configuration

(TBD: Steps to create OIDC clients for Rocket.Chat and Nextcloud integration)
