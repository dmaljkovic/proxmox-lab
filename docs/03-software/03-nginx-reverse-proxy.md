# Nginx Reverse Proxy and SSL

Brief description: Install Nginx 1.24, configure reverse proxy for all services, and obtain SSL certificates.

## What You'll Learn

- How to install Nginx and Certbot
- How to configure reverse proxy for backend services
- How to obtain and configure SSL certificates

## Prerequisites

- [ ] VM-103 created and Ubuntu 24.04 LTS installed
- [ ] IP address assigned and documented
- [ ] Backend services installed (Rocket.Chat, Nextcloud)
- [ ] Domain names planned (chat.example.com, cloud.example.com, etc.)
- [ ] Ports 80 and 443 available on public IP

## Estimated Time

4-6 hours

## Step-by-Step Instructions

### Step 1: SSH to Nginx Proxy VM

```bash
ssh admin@<nginx-ip>
```

### Step 2: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3: Install Nginx

```bash
sudo apt install -y nginx
```

Verify installation:

```bash
nginx -v
```

Expected: nginx version: nginx/1.24.x

Start Nginx:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

Test:

```bash
curl http://localhost
```

Should show "Welcome to nginx!"

### Step 4: Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Verify:

```bash
certbot --version
```

### Step 5: Configure Firewall

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
```

Check status:

```bash
sudo ufw status
```

### Step 6: Create Upstream Definitions

```bash
sudo nano /etc/nginx/conf.d/upstreams.conf
```

Add:

```nginx
# Upstream definitions for backend services

upstream rocketchat_backend {
    server <rocketchat-ip>:3000;
}

upstream nextcloud_backend {
    server <nextcloud-ip>:80;
}

upstream keycloak_backend {
    server <keycloak-ip>:8080;
}

upstream mkdocs_backend {
    server <mkdocs-ip>:8000;
}
```

Save (Ctrl+O, Enter, Ctrl+X)

### Step 7: Create HTTP to HTTPS Redirect

```bash
sudo nano /etc/nginx/sites-available/redirect
```

Add:

```nginx
# HTTP to HTTPS redirect for all domains
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
    
    location / {
        return 301 https://$host$request_uri;
    }
}
```

### Step 8: Create Rocket.Chat Configuration

```bash
sudo nano /etc/nginx/sites-available/rocketchat
```

Add:

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name chat.example.com;

    ssl_certificate /etc/letsencrypt/live/chat.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/chat.example.com/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # WebSocket support for Rocket.Chat
    location / {
        proxy_pass http://rocketchat_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Nginx-Proxy true;
        proxy_redirect off;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

### Step 9: Create Nextcloud Configuration

```bash
sudo nano /etc/nginx/sites-available/nextcloud
```

Add:

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name cloud.example.com;

    ssl_certificate /etc/letsencrypt/live/cloud.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cloud.example.com/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    client_max_body_size 10G;
    client_body_timeout 300s;
    fastcgi_buffers 64 4K;
    
    location / {
        proxy_pass http://nextcloud_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        
        proxy_buffering off;
        proxy_request_buffering off;
    }
}
```

### Step 10: Enable Sites

Remove default site:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Enable redirect:

```bash
sudo ln -s /etc/nginx/sites-available/redirect /etc/nginx/sites-enabled/
```

Test configuration:

```bash
sudo nginx -t
```

Should show "syntax is ok" and "test is successful"

### Step 11: Obtain SSL Certificates

Create webroot for Certbot:

```bash
sudo mkdir -p /var/www/certbot
```

Obtain certificates:

```bash
sudo certbot certonly --webroot -w /var/www/certbot \
  -d chat.example.com \
  -d cloud.example.com \
  -d auth.example.com \
  -d docs.example.com
```

Follow prompts:
- Enter email address
- Accept terms of service
- Choose whether to share email with EFF

### Step 12: Enable Service Sites

Now that certificates exist, enable service configurations:

```bash
sudo ln -s /etc/nginx/sites-available/rocketchat /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
```

Test and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Step 13: Setup Auto-Renewal

Test renewal:

```bash
sudo certbot renew --dry-run
```

Enable timer:

```bash
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

Verify timer:

```bash
sudo systemctl status certbot.timer
```

### Step 14: Test Access

Test Rocket.Chat:

```bash
curl -I https://chat.example.com
```

Test Nextcloud:

```bash
curl -I https://cloud.example.com
```

Both should return HTTP 200 or 302.

## Verification

- [ ] Nginx installed and running
- [ ] Certbot installed
- [ ] Firewall configured (UFW)
- [ ] Upstream definitions created
- [ ] HTTP to HTTPS redirect configured
- [ ] SSL certificates obtained for all domains
- [ ] Rocket.Chat reverse proxy configured
- [ ] Nextcloud reverse proxy configured
- [ ] Configuration syntax valid
- [ ] Sites enabled and Nginx reloaded
- [ ] Auto-renewal configured
- [ ] HTTPS access working for all services

## Common Issues

### Issue: certbot: command not found
**Solution**: 
```bash
sudo apt install -y certbot python3-certbot-nginx
```

### Issue: nginx: [emerg] bind() to 0.0.0.0:80 failed
**Solution**: Something else is using port 80:
```bash
sudo lsof -i :80
sudo systemctl stop apache2  # if Apache is running
sudo systemctl disable apache2
```

### Issue: Certificate validation failed
**Solution**: Ensure DNS points to your server IP:
```bash
nslookup chat.example.com
# Should return your server IP
```

### Issue: WebSocket errors in Rocket.Chat
**Solution**: Ensure WebSocket headers are configured:
```bash
sudo nano /etc/nginx/sites-available/rocketchat
# Verify these lines exist:
# proxy_set_header Upgrade $http_upgrade;
# proxy_set_header Connection "upgrade";
```

### Issue: Large file uploads fail in Nextcloud
**Solution**: Already configured with `client_max_body_size 10G`. If still failing, check Nextcloud settings.

## Next Steps

- [OpenLDAP Server](../04-security/01-firewall.md) (after all services configured)
- [Keycloak & OIDC](04-keycloak-oidc.md)
