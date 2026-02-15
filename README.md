# Vaultwarden Password Manager - Docker Deployment

A complete self-hosted password manager solution using Docker Compose with multiple components:
- **Vaultwarden** (Bitwarden-compatible server) - Password management application
- **PostgreSQL** - Database backend
- **Nginx** - Reverse proxy and web server
- **Automated Backups** - Daily backup service

## Table of Contents
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Detailed Setup Instructions](#detailed-setup-instructions)
- [Configuration](#configuration)
- [Accessing the Application](#accessing-the-application)
- [Admin Panel](#admin-panel)
- [Backup and Restore](#backup-and-restore)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)

## Architecture

```
┌─────────────────────────────────────────┐
│         Client (Browser/App)            │
└──────────────┬──────────────────────────┘
               │ HTTP/HTTPS (Port 80/443)
               ▼
┌─────────────────────────────────────────┐
│     Nginx Reverse Proxy Container       │
│  - SSL/TLS Termination                  │
│  - Request Routing                      │
│  - Static File Serving                  │
└──────────────┬──────────────────────────┘
               │ Internal Network
               ▼
┌─────────────────────────────────────────┐
│    Vaultwarden Application Container    │
│  - Password Vault Logic                 │
│  - API Endpoints                        │
│  - WebSocket Support                    │
└──────────────┬──────────────────────────┘
               │ Database Connection
               ▼
┌─────────────────────────────────────────┐
│     PostgreSQL Database Container       │
│  - User Data Storage                    │
│  - Encrypted Vault Data                 │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│      Backup Service Container           │
│  - Daily Automated Backups              │
│  - Retention Management                 │
└─────────────────────────────────────────┘
```

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/vaultwarden-docker.git
cd vaultwarden-docker
```

### 2. Configure Environment

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your settings
nano .env  # or use your preferred editor
```

**Required changes in `.env`:**
```bash
# Change these to secure random values
DB_PASSWORD=your_secure_database_password_here
ADMIN_TOKEN=your_secure_admin_token_here
```

**Generate secure tokens:**
```bash
# Generate database password
openssl rand -base64 32

# Generate admin token
openssl rand -base64 48
```

### 3. Start the Services

```bash
# Start all containers in detached mode
docker compose up -d

# View logs to ensure everything started correctly
docker compose logs -f
```

### 4. Access the Application

Open your browser and navigate to:
```
http://localhost
```

## Detailed Setup Instructions

### Step 1: Initial Setup

1. **Create project directory:**
   ```bash
   mkdir ~/vaultwarden-docker
   cd ~/vaultwarden-docker
   ```

2. **Clone or download the repository:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/vaultwarden-docker.git .
   ```

### Step 2: Environment Configuration

1. **Copy the environment template:**
   ```bash
   cp .env.example .env
   ```

2. **Edit `.env` file:**
   ```bash
   nano .env
   ```

3. **Configure required variables:**
   ```env
   # Database Configuration
   DB_PASSWORD=<your-secure-password>
   
   # Admin Configuration
   ADMIN_TOKEN=<your-secure-token>
   
   # Application Settings
   SIGNUPS_ALLOWED=true          # Set to false after creating accounts
   INVITATIONS_ALLOWED=true
   DOMAIN=http://localhost       # Change to your domain for production
   ```

### Step 3: Optional SSL Configuration

For HTTPS support (recommended for production):

1. **Generate self-signed certificate (for testing):**
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout nginx/ssl/key.pem \
     -out nginx/ssl/cert.pem \
     -subj "/CN=localhost"
   ```

2. **Update nginx configuration:**
   - Edit `nginx/conf.d/vaultwarden.conf`
   - Uncomment the SSL lines
   - Comment out the HTTP-only configuration

### Step 4: Deploy the Stack

1. **Pull all images:**
   ```bash
   docker compose pull
   ```

2. **Start the services:**
   ```bash
   docker compose up -d
   ```

3. **Verify all containers are running:**
   ```bash
   docker compose ps
   ```

   Expected output:
   ```
   NAME                  STATUS    PORTS
   vaultwarden-app       running   
   vaultwarden-db        running   
   vaultwarden-nginx     running   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
   vaultwarden-backup    running   
   ```

4. **Check container logs:**
   ```bash
   # View all logs
   docker compose logs
   
   # Follow logs in real-time
   docker compose logs -f
   
   # View specific service logs
   docker compose logs vaultwarden
   ```

### Step 5: Initial Application Setup

1. **Access the web interface:**
   - Open browser: `http://localhost`
   - Or if using SSL: `https://localhost`

2. **Create your account:**
   - Click "Create Account"
   - Enter email and master password
   - **IMPORTANT**: Store your master password securely!

3. **Disable public signups (recommended):**
   ```bash
   # Edit .env file
   nano .env
   
   # Change to:
   SIGNUPS_ALLOWED=false
   
   # Restart services
   docker compose restart vaultwarden
   ```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_PASSWORD` | PostgreSQL password | Required |
| `ADMIN_TOKEN` | Admin panel access token | Required |
| `SIGNUPS_ALLOWED` | Allow new user registration | `true` |
| `INVITATIONS_ALLOWED` | Allow user invitations | `true` |
| `DOMAIN` | Public domain URL | `http://localhost` |
| `SMTP_HOST` | Email server hostname | Optional |
| `SMTP_FROM` | Email sender address | Optional |

### Docker Compose Services

| Service | Image | Purpose | Ports |
|---------|-------|---------|-------|
| `postgres` | postgres:15-alpine | Database | 5432 (internal) |
| `vaultwarden` | vaultwarden/server:latest | Password manager | 8080 (internal) |
| `nginx` | nginx:alpine | Reverse proxy | 80, 443 |
| `backup` | postgres:15-alpine | Automated backups | None |

### Customizing Nginx

Edit `nginx/conf.d/vaultwarden.conf` to modify:
- Server name
- SSL certificates
- Proxy timeouts
- Custom headers

After changes:
```bash
docker compose restart nginx
```

## Accessing the Application

### Web Interface
- **URL**: http://localhost (or your configured domain)
- **Create Account**: Click "Create Account" on homepage
- **Login**: Use your email and master password

### Browser Extensions
Install the official Bitwarden extension:
- [Chrome/Edge](https://chrome.google.com/webstore/detail/bitwarden/nngceckbapebfimnlniiiahkandclblb)
- [Firefox](https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager/)

Configure the extension:
1. Click extension icon → Settings
2. Server URL: `http://localhost` (or your domain)
3. Login with your credentials

## Admin Panel

Access the admin panel to manage users and settings.

### Accessing Admin Panel

1. **Navigate to:**
   ```
   http://localhost/admin
   ```

2. **Enter admin token:**
   - Use the `ADMIN_TOKEN` from your `.env` file

### Admin Features
- View all registered users
- Delete user accounts
- Invite new users
- View system diagnostics
- Configure server settings

### Security Tip
Store your admin token securely. To regenerate:
```bash
# Generate new token
openssl rand -base64 48

# Update .env file
nano .env

# Restart Vaultwarden
docker compose restart vaultwarden
```

## Backup and Restore

### Automatic Backups

Backups run daily and are stored in `./backups/` directory.

**Backup contents:**
- PostgreSQL database dump
- Vaultwarden data directory

**Retention:** 7 days (configurable via `BACKUP_KEEP_DAYS`)

### Manual Backup

```bash
# Trigger immediate backup
docker compose exec backup /backup.sh

# View backup files
ls -lh backups/
```

### Restore from Backup

1. **Stop services:**
   ```bash
   docker compose down
   ```

2. **Restore database:**
   ```bash
   # Decompress backup
   gunzip backups/db_backup_YYYYMMDD_HHMMSS.sql.gz
   
   # Start only database
   docker compose up -d postgres
   
   # Restore dump
   docker compose exec -T postgres psql -U vaultwarden vaultwarden < backups/db_backup_YYYYMMDD_HHMMSS.sql
   ```

3. **Restore data directory:**
   ```bash
   # Extract data backup
   tar -xzf backups/vaultwarden_data_YYYYMMDD_HHMMSS.tar.gz -C /path/to/restore
   ```

4. **Restart all services:**
   ```bash
   docker compose up -d
   ```

## 🔧 Troubleshooting

### Check Service Status

```bash
# View running containers
docker compose ps

# Check specific service health
docker compose exec vaultwarden curl http://localhost:8080/alive
```

### View Logs

```bash
# All services
docker compose logs

# Specific service
docker compose logs vaultwarden
docker compose logs postgres
docker compose logs nginx

# Follow logs in real-time
docker compose logs -f
```

### Common Issues

#### Can't Access Web Interface

```bash
# Check if nginx is running
docker compose ps nginx

# Check nginx logs
docker compose logs nginx

# Test nginx config
docker compose exec nginx nginx -t

# Restart nginx
docker compose restart nginx
```

#### Database Connection Errors

```bash
# Check database status
docker compose exec postgres pg_isready -U vaultwarden

# View database logs
docker compose logs postgres

# Restart database
docker compose restart postgres
```

#### Password Reset Issues

Vaultwarden does not support password reset without email configured.

**To enable email:**
1. Configure SMTP settings in `.env`
2. Restart services: `docker compose restart`

**Alternative - Admin Reset:**
1. Access admin panel: `http://localhost/admin`
2. Delete user account
3. User can re-register

### Reset Everything

```bash
# Stop and remove all containers
docker compose down

# Remove all volumes (WARNING: Deletes all data!)
docker compose down -v

# Remove backup files
rm -rf backups/*

# Start fresh
docker compose up -d
```

## Security Considerations

### Essential Security Steps

1. **Change Default Credentials:**
   - Generate strong `DB_PASSWORD` and `ADMIN_TOKEN`
   - Use `openssl rand -base64 48` for secure tokens

2. **Disable Public Signups:**
   ```env
   SIGNUPS_ALLOWED=false
   ```

3. **Enable HTTPS:**
   - Use valid SSL certificates (Let's Encrypt recommended)
   - Redirect HTTP to HTTPS

4. **Use Strong Master Password:**
   - Minimum 12 characters
   - Mix of uppercase, lowercase, numbers, symbols
   - Never share or reuse

5. **Regular Backups:**
   - Verify backups work
   - Store backups securely off-site
   - Test restoration process

6. **Keep Updated:**
   ```bash
   # Pull latest images
   docker compose pull
   
   # Restart with new images
   docker compose up -d
   ```

7. **Network Security:**
   - Use firewall rules
   - Limit exposed ports
   - Consider VPN access

8. **Monitor Logs:**
   ```bash
   # Check for suspicious activity
   docker compose logs nginx | grep -i "error\|fail"
   ```

### Production Deployment

For production use:

1. **Use a proper domain**
2. **Configure real SSL certificates** (Let's Encrypt)
3. **Set up email notifications** (SMTP)
4. **Implement rate limiting**
5. **Use secure passwords for all services**
6. **Regular security audits**
7. **Implement monitoring and alerts**

## Maintenance

### Update Containers

```bash
# Pull latest images
docker compose pull

# Recreate containers with new images
docker compose up -d

# Remove old images
docker image prune
```

### View Resource Usage

```bash
# Container stats
docker stats

# Disk usage
docker system df

# Detailed disk usage
docker system df -v
```

### Clean Up

```bash
# Remove unused images
docker image prune -a

# Remove unused volumes (be careful!)
docker volume prune

# Clean everything (except volumes)
docker system prune
```

