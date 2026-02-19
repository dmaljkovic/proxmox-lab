# Rocket.Chat Installation

Brief description: Install Rocket.Chat 6.x on VM-101 using Docker Compose.

## What You'll Learn

- How to install Docker and Docker Compose
- How to configure Rocket.Chat with MongoDB
- How to verify the installation

## Prerequisites

- [ ] VM-101 created and Ubuntu 24.04 LTS installed
- [ ] IP address assigned and documented
- [ ] SSH access to VM-101 working
- [ ] 4GB RAM and 30GB disk available

## Estimated Time

3-4 hours

## Step-by-Step Instructions

### Step 1: SSH to Rocket.Chat VM

```bash
ssh admin@<rocketchat-ip>
```

### Step 2: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3: Install Docker

Install prerequisites:

```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

Add Docker GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Add Docker repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Verify installation:

```bash
docker --version
docker compose version
```

### Step 4: Add User to Docker Group

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Step 5: Create Docker Compose Configuration

Create directory:

```bash
mkdir -p ~/rocketchat
cd ~/rocketchat
```

Create docker-compose.yml:

```bash
cat > docker-compose.yml << 'EOF'
version: '3'

services:
  rocketchat:
    image: rocketchat/rocket.chat:latest
    restart: unless-stopped
    volumes:
      - ./uploads:/app/uploads
    environment:
      - PORT=3000
      - ROOT_URL=http://localhost:3000
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
    ports:
      - 3000:3000
    depends_on:
      - mongo
    networks:
      - rocketchat-network

  mongo:
    image: mongo:6.0
    restart: unless-stopped
    volumes:
      - ./data/db:/data/db
      - ./data/dump:/dump
    command: mongod --oplogSize 128 --replSet rs0
    networks:
      - rocketchat-network

  mongo-init-replica:
    image: mongo:6.0
    command: >
      bash -c
      "for i in \`seq 1 30\`; do
        mongo mongo/rocketchat --eval \"rs.initiate({
          _id: 'rs0',
          members: [ { _id: 0, host: 'mongo:27017' } ]})\" &&
        s=\$\$? && break || s=\$\$?;
        echo \"Tried \$\$i times. Waiting 5 secs...\";
        sleep 5;
      done; (exit \$\$s)"
    depends_on:
      - mongo
    networks:
      - rocketchat-network

networks:
  rocketchat-network:
    driver: bridge
