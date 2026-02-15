# Vaultwarden Deployment Guide

Complete step-by-step guide for deploying the Vaultwarden password manager stack.

---

## Prerequisites Checklist

Before starting, ensure you have:

- [ ] Ubuntu 22.04/24.04 LTS (or similar Linux)
- [ ] Docker 20.10+ installed
- [ ] Docker Compose v2.0+ installed
- [ ] 2GB RAM available
- [ ] 10GB disk space
- [ ] Ports 80 and 443 free
- [ ] Root or sudo access

---

## Installation Steps

### 1. Install Docker

```bash
# Download and run Docker installation script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to docker group
sudo usermod -aG docker $USER

# Log out and back in, then verify
docker --version
docker compose version
```

### 2. Clone Repository

```bash
cd ~
git clone https://github.com/YOUR_USERNAME/vaultwarden-docker.git
cd vaultwarden-docker
ls -la
```

### 3. Configure Environment

```bash
# Copy environment template
cp .env.example .env

# Generate secure passwords
echo "Database Password: $(openssl rand -base64 32)"
echo "Admin Token: $(openssl rand -base64 48)"

# Edit environment file
nano .env
```

Update these variables in `.env`:
```env
DB_PASSWORD=paste_generated_db_password_here
ADMIN_TOKEN=paste_generated_admin_token_here
SIGNUPS_ALLOWED=true
DOMAIN=http://localhost
```

Save with `Ctrl+X`, `Y`, `Enter`.

### 4. Deploy the Stack

```bash
# Pull all Docker images
docker compose pull

# Start all services
docker compose up -d

# Watch startup logs
docker compose logs -f
```

Wait for:
- PostgreSQL: `database system is ready to accept connections`
- Vaultwarden: `Rocket has launched`
- Nginx: `start worker process`

Press `Ctrl+C` to exit logs.

### 5. Verify Deployment

```bash
# Check all containers are running
docker compose ps

# Test web access
curl -I http://localhost
```

---

## Initial Setup

### Create First User Account

1. Open browser to `http://localhost`
2. Click "Create Account"
3. Enter email address
4. Create strong master password
5. Click "Submit"

**⚠️ CRITICAL:** Your master password cannot be recovered!

### Access Admin Panel

1. Navigate to `http://localhost/admin`
2. Enter your `ADMIN_TOKEN` from `.env` file

### Disable Public Signups

After creating all needed accounts:

```bash
nano .env
# Change: SIGNUPS_ALLOWED=false
docker compose restart vaultwarden
```

---

## Using Your Password Manager

### Web Vault
- URL: `http://localhost`
- Store passwords, notes, cards
- Generate strong passwords

### Browser Extensions
1. Install Bitwarden extension
2. Settings → Server URL: `http://localhost`
3. Login with credentials

### Mobile Apps
1. Download Bitwarden app
2. Settings → Self-hosted → Server URL
3. Login with credentials

---

## Backup and Restore

### Manual Backup
```bash
docker compose exec backup /scripts/backup.sh
ls -lh backups/
```

### Restore from Backup
```bash
docker compose down
# Restore database and data
docker compose up -d
```

---

## Common Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f

# Update
docker compose pull
docker compose up -d
```

---

## Troubleshooting

### Can't Access Web Interface
```bash
docker compose logs nginx
docker compose restart nginx
```

### Database Errors
```bash
docker compose logs postgres
docker compose restart postgres vaultwarden
```

---

**For detailed information, see README.md**
