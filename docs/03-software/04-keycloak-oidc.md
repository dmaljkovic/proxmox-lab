# Keycloak Installation and OIDC Configuration

Brief description: Install Keycloak 24.0.0, configure OpenLDAP federation, and set up OIDC for SSO.

## What You'll Learn

- How to install Keycloak 24.0.0
- How to configure OpenLDAP user federation
- How to create OIDC clients for SSO
- How to integrate with Rocket.Chat and Nextcloud

## Prerequisites

- [ ] VM-106 created and Ubuntu 24.04 LTS installed
- [ ] VM-105 (OpenLDAP) created and accessible
- [ ] OpenJDK 17 installed on Keycloak VM
- [ ] IP addresses documented
- [ ] Nginx reverse proxy configured

## Estimated Time

6-8 hours (Days 7-8 combined)

## Part 1: Keycloak Installation

### Step 1: SSH to Keycloak VM

```bash
ssh admin@<keycloak-ip>
```

### Step 2: Verify Java Installation

```bash
java -version
```

Should show OpenJDK 17.

If not installed:

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk
```

### Step 3: Download and Install Keycloak

```bash
cd /tmp
wget https://github.com/keycloak/keycloak/releases/download/24.0.0/keycloak-24.0.0.tar.gz
```

Extract:

```bash
sudo tar -xzf keycloak-24.0.0.tar.gz -C /opt/
sudo mv /opt/keycloak-24.0.0 /opt/keycloak
```

### Step 4: Create Keycloak User

```bash
sudo groupadd keycloak
sudo useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
```

Set ownership:

```bash
sudo chown -R keycloak:keycloak /opt/keycloak
sudo mkdir -p /opt/keycloak/data
sudo mkdir -p /var/log/keycloak
sudo chown keycloak:keycloak /opt/keycloak/data /var/log/keycloak
```

### Step 5: Create Systemd Service

```bash
sudo nano /etc/systemd/system/keycloak.service
```

Add:

```ini
[Unit]
Description=Keycloak Identity Provider
After=network.target

[Service]
Type=idle
User=keycloak
Group=keycloak
ExecStart=/opt/keycloak/bin/kc.sh start --hostname=auth.example.com --http-port=8080 --proxy-headers=xforwarded --log=console
WorkingDirectory=/opt/keycloak
Restart=always
RestartSec=10
Environment="JAVA_OPTS=-Xms1024m -Xmx2048m -Djava.security.egd=file:/dev/./urandom"

[Install]
WantedBy=multi-user.target
```

### Step 6: Create Admin User

```bash
cd /opt/keycloak
sudo -u keycloak bin/kc.sh bootstrap-admin user --username admin --password <admin-password>
```

Replace `<admin-password>` with a strong password.

### Step 7: Start Keycloak

```bash
sudo systemctl daemon-reload
sudo systemctl enable keycloak
sudo systemctl start keycloak
```

Check status:

```bash
sudo systemctl status keycloak
```

View logs:

```bash
sudo journalctl -u keycloak -f
```

Wait for "Listening on: http://0.0.0.0:8080"

Press Ctrl+C to exit logs.

### Step 8: Access Admin Console

Open browser: `https://auth.example.com/admin`

Login:
- Username: admin
- Password: [password you set]

## Part 2: OpenLDAP Configuration

### Step 9: Configure OpenLDAP (VM-105)

SSH to OpenLDAP VM:

```bash
ssh admin@<openldap-ip>
```

Install OpenLDAP:

```bash
sudo apt update
sudo apt install -y slapd ldap-utils
```

Reconfigure:

```bash
sudo dpkg-reconfigure slapd
```

Answer wizard:
- Omit OpenLDAP server configuration? **No**
- DNS domain name: **example.com**
- Organization name: **Your Organization**
- Administrator password: [strong password]
- Database backend: **MDB**
- Remove database when slapd is purged? **No**
- Move old database? **Yes**

### Step 10: Configure LDAP TLS

Create certificate directory:

```bash
sudo mkdir -p /etc/ldap/certs
```

Generate self-signed certificate:

```bash
cd /etc/ldap/certs
sudo openssl req -new -x509 -days 365 -nodes \
  -out ldap.crt \
  -keyout ldap.key \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=ldap.example.com"
```

Set permissions:

```bash
sudo chown openldap:openldap /etc/ldap/certs/ldap.key
sudo chmod 600 /etc/ldap/certs/ldap.key
```

### Step 11: Create Base Structure

Create LDIF file:

```bash
cat > ~/base-structure.ldif << 'EOF'
dn: ou=users,dc=example,dc=com
objectClass: organizationalUnit
ou: users

dn: ou=groups,dc=example,dc=com
objectClass: organizationalUnit
ou: groups
