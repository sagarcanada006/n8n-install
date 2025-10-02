# 🚀 n8n Setup on Amazon Linux 2023 with DuckDNS + HTTPS

This guide explains how to install and run **n8n** on an **Amazon Linux 2023 EC2 instance** using Docker, Traefik (reverse proxy), and DuckDNS (free dynamic DNS).  
It enables **secure HTTPS access** to your n8n workflows (e.g., for Telegram bots).

---

## ✅ 1. Launch EC2 Instance
1. Go to **AWS Console → EC2 → Launch Instance**.  
2. Choose **Amazon Linux 2023** as AMI.  
3. Select **t2.micro** (free-tier eligible).  
4. Create a **Security Group** with these inbound rules:
   - **22 (SSH)** → Your IP only
   - **80 (HTTP)** → 0.0.0.0/0
   - **443 (HTTPS)** → 0.0.0.0/0  
5. Launch and connect to your instance via SSH.

---

## ✅ 2. Install Docker & Docker Compose
Run the following commands on your EC2:

```bash
# Update system
sudo dnf update -y

# Install Docker
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
newgrp docker

# Install Docker Compose (standalone binary)
DOCKER_COMPOSE_VERSION=v2.27.0
sudo curl -SL https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-linux-x86_64 \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker --version
docker-compose version
```
---

## ✅ 3. Setup DuckDNS
DuckDNS provides a free hostname that always points to your server’s public IP. This is needed for Let’s Encrypt SSL certificates.

1. Go to DuckDNS.org
.
2. Sign in with GitHub, Google, or Twitter.

3. Under Domains, add a new subdomain (e.g., mydomain).

4. Your hostname will be: mydomain.duckdns.org

Copy your DuckDNS Token from the dashboard. You will use this in your .env file.

🔑 Example:

Domain: mydomain.duckdns.org

Token: abcd1234efgh5678ijkl

---

## ✅ 4. Clone and Configure .env
Clone your GitHub repo and create an .env file:

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
vi .env
```

Example .env

# Timezone
GENERIC_TIMEZONE=America/New_York

# Your email for Let's Encrypt SSL
N8N_EMAIL=your-email@example.com

# DuckDNS config
DUCKDNS_SUBDOMAIN=mydomain
DUCKDNS_TOKEN=your-duckdns-token

---

## ✅ 5. Clone docker-compose.yml
Clone docker-compose.yml

---

## ✅ 6. Start Services

```bash
docker-compose up -d
```

What happens:

DuckDNS container updates your current EC2 IP to sagary.duckdns.org.

Traefik listens on ports 80 and 443, requests a Let’s Encrypt SSL certificate, and forwards traffic to n8n.

n8n starts and is reachable on HTTPS.

✅ After a few seconds, check the logs to confirm:

```bash
docker-compose logs -f traefik
```
You should see messages about a successful ACME certificate request.

If you see certificate errors, double-check:

- Ports 80 and 443 are open in the EC2 Security Group.

- Your DuckDNS domain points to your instance’s public IP.

- The email in .env is valid (Let’s Encrypt rejects example.com).

---

## ✅ 6. Access n8n

- Open: https://mydomain.duckdns.org

- Accept the Let’s Encrypt certificate (automatic)

- Sign up with your email (N8N_EMAIL)

