# Prometheus Monitoring Stack

A complete Docker-based monitoring solution using Prometheus, Grafana, Node Exporter, and a sample Python application.

## Components

- **Prometheus**: Time-series database for metrics collection (Port 9090)
- **Grafana**: Visualization and dashboarding platform (Port 3000)
- **Node Exporter**: System and hardware metrics exporter (Port 9100)
- **Sample App**: Python Flask application with Prometheus metrics (Port 8080)

## Architecture

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   Grafana   │◄─────│  Prometheus  │◄─────│ Sample App  │
│   :3000     │      │    :9090     │      │   :8080     │
└─────────────┘      └──────────────┘      └─────────────┘
                            ▲
                            │
                     ┌──────┴──────┐
                     │Node Exporter│
                     │    :9100    │
                     └─────────────┘
```

## Prerequisites

- Ubuntu 20.04+ or similar Linux distribution
- Docker Engine 20.10+
- Docker Compose 2.0+
- At least 2GB of free RAM
- Ports 3000, 8080, 9090, and 9100 available

## Installation on Ubuntu

### Step 1: Install Docker

```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group
sudo usermod -aG docker $USER

# Log out and back in for group changes to take effect
```

### Step 2: Verify Docker Installation

```bash
docker --version
docker compose version
```

### Step 3: Deploy the Stack

```bash
# Clone the repository
git clone <your-repo-url>
cd prometheus-monitoring-stack

# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Check status
docker compose ps
```

## Quick Start

```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down

# Stop and remove all data
docker compose down -v
```

## Access the Services

- **Grafana**: http://localhost:3000
  - Username: `admin`
  - Password: `admin` (change on first login)
  
- **Prometheus**: http://localhost:9090
  - View targets: http://localhost:9090/targets
  - Run queries: http://localhost:9090/graph
  
- **Sample App**: http://localhost:8080
  - Metrics: http://localhost:8080/metrics

- **Node Exporter**: http://localhost:9100/metrics

## Using the Stack

### 1. Access Grafana Dashboard

1. Navigate to http://localhost:3000
2. Login with admin/admin
3. Go to Dashboards → Browse
4. Open "Node Exporter Dashboard"
5. You should see CPU, Memory, and system metrics

### 2. Query Prometheus

Navigate to http://localhost:9090 and try these queries:

```promql
# Check all targets are up
up

# CPU usage percentage
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Application request rate
rate(app_requests_total[5m])

# Application active users
app_active_users
```

### 3. Generate Sample Application Traffic

```bash
# Make requests to generate metrics
for i in {1..100}; do curl http://localhost:8080/; done
for i in {1..50}; do curl http://localhost:8080/api/data; done

# View metrics
curl http://localhost:8080/metrics
```

## Project Structure

```
prometheus-monitoring-stack/
├── docker-compose.yml              # Container orchestration
├── README.md                       # This file
├── DEPLOYMENT_GUIDE.md            # Detailed deployment guide
├── prometheus/
│   ├── prometheus.yml             # Prometheus configuration
│   └── alert_rules.yml            # Alert definitions
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── datasource.yml     # Auto-configure Prometheus datasource
│   │   └── dashboards/
│   │       └── dashboard.yml      # Dashboard provider config
│   └── dashboards/
│       └── node-exporter-dashboard.json  # Pre-built dashboard
└── sample-app/
    ├── Dockerfile                 # Flask app container build
    ├── app.py                     # Flask application
    └── requirements.txt           # Python dependencies
```

## Configuration Files

### [docker-compose.yml](docker-compose.yml)

Defines four services:
- `prometheus`: Metrics database
- `grafana`: Visualization platform
- `node-exporter`: System metrics collector
- `sample-app`: Demo application to monitor

### [prometheus/prometheus.yml](prometheus/prometheus.yml)

Prometheus configuration with:
- 15-second scrape interval
- Three scrape targets (prometheus, node-exporter, sample-app)
- Alert rules loading

### [prometheus/alert_rules.yml](prometheus/alert_rules.yml)

Alert definitions for:
- Instance down
- High CPU usage (>80%)
- High memory usage (>85%)

### [grafana/provisioning/datasources/datasource.yml](grafana/provisioning/datasources/datasource.yml)

Auto-configures Prometheus as the default Grafana datasource.

### [sample-app/Dockerfile](sample-app/Dockerfile)

Builds the Flask application container with Prometheus client library.

## Customization

### Add More Scrape Targets

Edit `prometheus/prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'my-service'
    static_configs:
      - targets: ['my-service:port']
```

Then reload Prometheus:
```bash
docker compose exec prometheus kill -HUP 1
```

### Create Custom Alerts

Edit `prometheus/alert_rules.yml` and add your rules:

```yaml
- alert: MyCustomAlert
  expr: my_metric > 100
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Custom alert triggered"
```

### Change Grafana Password

Edit `docker-compose.yml`:

```yaml
environment:
  - GF_SECURITY_ADMIN_PASSWORD=your-secure-password
```

## Management Commands

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs (all services)
docker compose logs -f

# View logs (specific service)
docker compose logs -f prometheus

# Restart a service
docker compose restart prometheus

# Rebuild after code changes
docker compose up -d --build sample-app

# Remove everything including volumes
docker compose down -v

# Check service status
docker compose ps

# Access Prometheus container
docker compose exec prometheus sh

# Access database directly
docker compose exec prometheus promtool check config /etc/prometheus/prometheus.yml
```

## Monitoring Best Practices

1. **Set meaningful alert thresholds** based on your baseline
2. **Create custom dashboards** for your specific metrics
3. **Regularly review and update** alert rules
4. **Monitor Prometheus itself** to ensure scraping works
5. **Back up Grafana dashboards** and Prometheus data

## Troubleshooting

### Port Already in Use

Edit `docker-compose.yml` to use different ports:

```yaml
grafana:
  ports:
    - "3001:3000"  # Change 3000 to 3001
```

### Service Not Starting

```bash
# Check logs
docker compose logs <service-name>

# Restart specific service
docker compose restart <service-name>

# Rebuild container
docker compose up -d --build <service-name>
```

### Prometheus Targets Down

1. Check Prometheus targets: http://localhost:9090/targets
2. Verify network connectivity:
   ```bash
   docker compose exec prometheus wget -qO- http://node-exporter:9100/metrics
   ```
3. Check service logs:
   ```bash
   docker compose logs node-exporter
   ```

### Grafana Shows No Data

1. Check Prometheus is accessible: http://localhost:9090
2. Verify datasource in Grafana: Configuration → Data Sources
3. Test query in Prometheus first, then use in Grafana
4. Check Grafana logs:
   ```bash
   docker compose logs grafana
   ```

## Data Persistence

Data is persisted in Docker volumes:
- `prometheus_data`: Prometheus metrics data
- `grafana_data`: Grafana dashboards and settings

To backup:
```bash
docker run --rm -v prometheus-monitoring-stack_prometheus_data:/data -v $(pwd):/backup ubuntu tar czf /backup/prometheus-backup.tar.gz /data
```

## Security Considerations

For production use:
1. Change default Grafana password
2. Enable HTTPS/TLS
3. Implement authentication for Prometheus
4. Use secrets management for passwords
5. Restrict network access with firewall rules

## Next Steps

- Explore Grafana's dashboard gallery
- Add alerting integrations (Slack, email, PagerDuty)
- Monitor your own applications
- Set up long-term storage with Thanos or Cortex
- Implement service discovery for dynamic environments

## Contributing

Feel free to submit issues and enhancement requests!

## License

MIT License - Free to use for any purpose.

## Author

Created for educational purposes - Docker and Prometheus monitoring demonstration.
