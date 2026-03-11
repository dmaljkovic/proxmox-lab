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

### Step 7: Initial Access & Admin Console (Temporary – via Internal)

From another internal VM (or Proxmox console forwarding):

`http://<keycloak-internal-ip>:8080/admin`

Login: admin / <password from service file>

After first login → remove bootstrap env vars from service file, reload & restart.

## Part 2: OIDC Client Configuration

### Add LDAP Federation (Keycloak → OpenLDAP)

Open the Keycloak Admin Console.

`https://auth.unseen-uni.online`

Login with the admin account.

Navigate:

```
Realm: example
→ User Federation
→ Add provider
→ ldap
```

**Basic settings**

| Field | Value |
| Vendor | Other |
| Connection URL | `ldap://LDAP_VM_IP:389` |
| Bind DN | `cn=admin,dc=unseen-uni,dc=online` |
| Bind Credential | LDAP admin password |
| Users DN | `ou=people,dc=unseen-uni,dc=online` |
| Username LDAP attribute | `uid` |
| RDN LDAP attribute | `uid` |
| UUID LDAP attribute | `entryUUID` |
| User Object Classes | `inetOrgPerson` |
| Import users | ON |

**Set:**

- Import Users = ON
- Edit Mode = READ_ONLY

Then click:

- Save
- Test connection

Click:

- Test connection
- Test authentication

Then click:

- Synchronize all users

Your testuser should now appear in:

- Users → View all users

### Create OIDC Clients

Clients represent applications that use Keycloak login.

Go to:

`Clients → Create client`

#### Client 1 — Rocket.Chat

**General settings**

| Field | Value |
| Client type | OpenID Connect |
| Client ID | `rocketchat` |
| Name | Rocket.Chat |

Click Next

**Capability config**

Enable:

- Client authentication = ON
- Authorization = OFF
- Standard flow = ON

Click Next

**Login settings**

| Field | Value |
| Root URL | `https://chat.unseen-uni.online` |
| Home URL | `https://chat.unseen-uni.online` |
| Valid redirect URIs | `https://chat.unseen-uni.online/*` |
| Web origins | `https://chat.unseen-uni.online` |

Click Save

#### Client 2 — Nextcloud

Create another client.

**General**

| Field | Value |
| Client ID | `nextcloud` |
| Client type | OpenID Connect |

**Login settings**

| Field | Value |
| Root URL | `https://cloud.unseen-uni.online` |
| Valid redirect URIs | `https://cloud.unseen-uni.online/*` |
| Web origins | `https://cloud.unseen-uni.online` |

Save.

### Get Client Secrets

For each client:

1. Clients → select client → Credentials tab

You will see:

**Client Secret**

Copy it.

Example:

- `rocketchat secret: 4d4c8c1a-xxxx-xxxx`
- `nextcloud secret: 39a8b71f-xxxx-xxxx`

Save them somewhere safe (password manager).

### Final Architecture

Your authentication flow becomes:

```
User
 ↓
Rocket.Chat / Nextcloud
 ↓
Keycloak (OIDC)
 ↓
OpenLDAP
```

So:

- **LDAP** → identity storage
- **Keycloak** → authentication provider
- **Apps** → trust Keycloak
