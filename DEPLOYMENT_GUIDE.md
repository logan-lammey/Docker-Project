# Deployment Guide - Prometheus Monitoring Stack

Complete step-by-step guide to deploy the Prometheus monitoring stack on Ubuntu.

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Initial Setup](#initial-setup)
3. [Docker Installation](#docker-installation)
4. [Deploy the Stack](#deploy-the-stack)
5. [Verification](#verification)
6. [Usage Guide](#usage-guide)
7. [Maintenance](#maintenance)
8. [Troubleshooting](#troubleshooting)

## System Requirements

### Minimum Requirements
- **OS**: Ubuntu 20.04 LTS or later
- **CPU**: 2 cores
- **RAM**: 2GB minimum (4GB recommended)
- **Disk**: 10GB free space
- **Network**: Internet access for Docker images

### Software Prerequisites
- Ubuntu Server/Desktop
- sudo access
- Internet connection

## Initial Setup

### Step 1: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Prerequisites

```bash
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    software-properties-common
```

## Docker Installation

### Install Docker Engine

```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
docker compose version
```

### Post-Installation Steps

```bash
# Add your user to docker group (avoid using sudo)
sudo usermod -aG docker $USER

# Apply group changes (or log out and back in)
newgrp docker

# Test Docker
docker run hello-world
```

## Deploy the Stack

### Step 1: Download the Project

```bash
# Option 1: Clone from GitHub
git clone <your-github-repo-url>
cd prometheus-monitoring-stack

# Option 2: Download and extract zip
# Upload the zip file to your server, then:
unzip prometheus-monitoring-stack.zip
cd prometheus-monitoring-stack
```

### Step 2: Verify File Structure

```bash
# Check all files are present
ls -R

# Expected structure:
# .
# ├── docker-compose.yml
# ├── prometheus/
# │   ├── prometheus.yml
# │   └── alert_rules.yml
# ├── grafana/
# │   ├── provisioning/
# │   └── dashboards/
# └── sample-app/
#     ├── Dockerfile
#     ├── app.py
#     └── requirements.txt
```

### Step 3: Validate Configuration

```bash
# Validate docker-compose.yml syntax
docker compose config

# Should display the configuration without errors
```

### Step 4: Pull Docker Images (Optional)

```bash
# Pre-download images to speed up first start
docker compose pull
```

### Step 5: Start the Stack

```bash
# Start all services in detached mode
docker compose up -d

# Expected output:
# [+] Running 5/5
#  ✔ Network prometheus-monitoring-stack_monitoring    Created
#  ✔ Volume "prometheus-monitoring-stack_prometheus_data" Created
#  ✔ Volume "prometheus-monitoring-stack_grafana_data"   Created
#  ✔ Container prometheus      Started
#  ✔ Container node-exporter   Started
#  ✔ Container sample-app      Started
#  ✔ Container grafana         Started
```

### Step 6: Monitor Startup

```bash
# Watch logs as services start
docker compose logs -f

# Press Ctrl+C to exit (services keep running)

# Check all containers are running
docker compose ps
```

## Verification

### 1. Check All Services Are Running

```bash
docker compose ps

# Expected output (all containers should be "Up"):
# NAME            SERVICE         STATUS          PORTS
# grafana         grafana         Up              0.0.0.0:3000->3000/tcp
# node-exporter   node-exporter   Up              0.0.0.0:9100->9100/tcp
# prometheus      prometheus      Up              0.0.0.0:9090->9090/tcp
# sample-app      sample-app      Up              0.0.0.0:8080->8080/tcp
```

### 2. Test Each Service

#### Prometheus
```bash
# Check Prometheus health
curl http://localhost:9090/-/healthy

# Expected: Prometheus is Healthy.

# Check targets
curl http://localhost:9090/api/v1/targets | jq
```

#### Grafana
```bash
# Check Grafana health
curl http://localhost:3000/api/health

# Expected: {"database":"ok","version":"..."}
```

#### Node Exporter
```bash
# Check Node Exporter metrics
curl http://localhost:9100/metrics | head -20
```

#### Sample Application
```bash
# Check sample app
curl http://localhost:8080

# Check app metrics
curl http://localhost:8080/metrics
```

### 3. Access Web Interfaces

Open in your browser:
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)
- **Sample App**: http://localhost:8080

### 4. Verify Prometheus Targets

1. Go to http://localhost:9090/targets
2. All targets should show "UP" status:
   - prometheus (localhost:9090)
   - node-exporter (node-exporter:9100)
   - sample-app (sample-app:8080)

## Usage Guide

### Accessing Grafana

1. Navigate to http://localhost:3000
2. Login:
   - Username: `admin`
   - Password: `admin`
3. Change password when prompted
4. Go to Dashboards → Browse
5. Open "Node Exporter Dashboard"

### Creating Prometheus Queries

Navigate to http://localhost:9090 and try:

```promql
# Check all services are up
up

# CPU usage percentage
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Request rate
rate(app_requests_total[5m])

# Active users
app_active_users
```

### Generating Traffic

```bash
# Generate requests to sample app
for i in {1..100}; do 
    curl -s http://localhost:8080/ > /dev/null
    echo "Request $i sent"
done

# Check metrics updated
curl http://localhost:8080/metrics | grep app_requests_total
```

## Maintenance

### View Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f prometheus
docker compose logs -f grafana

# Last 100 lines
docker compose logs --tail=100 grafana
```

### Restart Services

```bash
# Restart all
docker compose restart

# Restart specific service
docker compose restart prometheus

# Rebuild and restart (after code changes)
docker compose up -d --build sample-app
```

### Update Configuration

```bash
# After modifying prometheus.yml or alert_rules.yml
docker compose exec prometheus kill -HUP 1

# Or restart the service
docker compose restart prometheus
```

### Backup Data

```bash
# Create backup directory
mkdir -p ~/backups

# Backup Prometheus data
docker run --rm \
  -v prometheus-monitoring-stack_prometheus_data:/data \
  -v ~/backups:/backup \
  ubuntu tar czf /backup/prometheus-$(date +%Y%m%d).tar.gz /data

# Backup Grafana data
docker run --rm \
  -v prometheus-monitoring-stack_grafana_data:/data \
  -v ~/backups:/backup \
  ubuntu tar czf /backup/grafana-$(date +%Y%m%d).tar.gz /data
```

### Clean Up

```bash
# Stop services
docker compose down

# Stop and remove volumes (⚠️ deletes all data)
docker compose down -v

# Remove unused images
docker image prune -a
```

## Troubleshooting

### Port Already in Use

**Problem**: Error: "bind: address already in use"

**Solution**:
```bash
# Find what's using the port
sudo lsof -i :3000
sudo netstat -tulpn | grep 3000

# Change port in docker-compose.yml
# Edit: ports: - "3001:3000"
```

### Permission Denied

**Problem**: Docker permission errors

**Solution**:
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or:
newgrp docker

# Check membership
groups
```

### Service Not Starting

**Problem**: Container exits immediately

**Solution**:
```bash
# Check specific logs
docker compose logs prometheus

# Verify config files
docker compose config

# Check disk space
df -h

# Restart Docker
sudo systemctl restart docker
```

### Grafana Shows No Data

**Solution**:
1. Verify Prometheus is running: `docker compose ps`
2. Check targets: http://localhost:9090/targets
3. Test query in Prometheus first
4. Check Grafana datasource: Configuration → Data Sources
5. Verify network:
   ```bash
   docker compose exec grafana ping prometheus
   ```

### High Resource Usage

```bash
# Check resource usage
docker stats

# Limit resources in docker-compose.yml:
grafana:
  deploy:
    resources:
      limits:
        cpus: '0.5'
        memory: 512M
```

## Advanced Configuration

### Enable External Access

```bash
# Open firewall ports
sudo ufw allow 3000/tcp
sudo ufw allow 9090/tcp

# Or for firewalld
sudo firewall-cmd --add-port=3000/tcp --permanent
sudo firewall-cmd --add-port=9090/tcp --permanent
sudo firewall-cmd --reload
```

### Auto-Start on Boot

Docker Compose services will auto-start if Docker is enabled:

```bash
# Ensure Docker starts on boot
sudo systemctl enable docker

# Services with restart: unless-stopped will auto-start
```

## Deployment Checklist

- [ ] Docker installed and running
- [ ] Repository cloned/extracted
- [ ] All services started (`docker compose up -d`)
- [ ] Can access Grafana (http://localhost:3000)
- [ ] Can access Prometheus (http://localhost:9090)
- [ ] Sample app responding (http://localhost:8080)
- [ ] All Prometheus targets "UP"
- [ ] Grafana datasource configured
- [ ] Dashboard showing data

**Congratulations! Your Prometheus monitoring stack is deployed!**
