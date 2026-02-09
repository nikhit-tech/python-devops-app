# Python DevOps API with Observability Stack

A comprehensive Python-based API application with full observability using Prometheus and Grafana. This project demonstrates best practices for monitoring, logging, and visualization in modern DevOps environments.

## Project Overview

This repository contains a FastAPI-based Python application with integrated observability stack. The project showcases:

- **Python API**: RESTful API built with FastAPI framework
- **Prometheus**: Metrics collection and monitoring
- **Grafana**: Data visualization and dashboarding
- **Redis**: In-memory caching and message broker
- **PostgreSQL**: Primary database for persistent storage

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps   │───▶│   Python API    │───▶│   PostgreSQL    │
└─────────────────┘    │  (Port 8000)    │    │   (Port 5432)   │
                       └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Redis Cache   │    │   /metrics      │
                       │   (Port 6379)   │    │   Endpoint      │
                       └─────────────────┘    └─────────────────┘
                                                       │
                       ┌─────────────────┐            ▼
                       │   Grafana       │◀───┌─────────────────┐
                       │   (Port 3000)   │    │   Prometheus    │
                       └─────────────────┘    │   (Port 9090)   │
                                              └─────────────────┘
```

### Why Prometheus and Grafana?

- **Prometheus**: Provides powerful time-series data collection, flexible querying, and excellent integration with Python applications
- **Grafana**: Offers beautiful, customizable dashboards for visualizing metrics collected by Prometheus
- **Combined**: Create a comprehensive monitoring solution for production applications

## Repository Structure

```
python-devops-app/
├── app/
│   └── main.py                    # FastAPI application entry point
├── monitoring/
│   ├── prometheus.yml            # Prometheus configuration
│   ├── prometheus-configmap.yaml # Kubernetes Prometheus config
│   └── prometheus-deployment.yaml # Kubernetes Prometheus deployment
├── k8s/
│   ├── deployment.yaml           # Kubernetes API deployment
│   ├── service.yaml              # Kubernetes service definition
│   └── ingress.yaml              # Kubernetes ingress configuration
├── terraform/                    # Infrastructure as Code (optional)
├── .github/
│   └── workflows/
│       └── ci-cd.yml            # GitHub Actions CI/CD pipeline
├── docker-compose.yml            # Docker Compose configuration
├── Dockerfile                    # Docker image definition
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

## Prerequisites

### For Docker-Based Setup
- Docker Engine 20.10+
- Docker Compose v2.0+
- Git

### For Manual (Non-Docker) Setup
- Python 3.11+
- pip (Python package manager)
- PostgreSQL 15+ (or access to remote PostgreSQL)
- Redis 7+ (or access to remote Redis)
- Prometheus (latest version)
- Grafana (latest version)
- System administrator access for service installation

## Docker-Based Setup

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd python-devops-app
   ```

2. **Start the entire stack:**
   ```bash
   docker-compose up -d
   ```

3. **Verify all services are running:**
   ```bash
   docker-compose ps
   ```

## Stopping the Application

### How to Identify if Application is Running

1. **Check running containers:**
   ```bash
   docker-compose ps
   # Look for containers with names starting with "python-devops-app-"
   ```

2. **Check listening ports:**
   ```bash
   netstat -tlnp 2>/dev/null | grep -E ":(8000|19090|3000|5432|6379)"
   # Should show ports 8000, 19090, 3000, 5432, 6379 if running
   ```

3. **Check Docker containers:**
   ```bash
   docker ps | grep python-devops-app
   ```

### How to Stop the Application Gracefully

1. **Stop all services:**
   ```bash
   docker-compose down
   ```

2. **Verify all containers are stopped:**
   ```bash
   docker-compose ps
   # Should show no running containers
   ```

3. **Verify ports are no longer listening:**
   ```bash
   netstat -tlnp 2>/dev/null | grep -E ":(8000|19090|3000|5432|6379)"
   # Should return empty (no output)
   ```

### Cleanup Steps

The `docker-compose down` command handles most cleanup automatically:

- **Stopped containers:** All application containers are stopped and removed
- **Networks:** Docker networks are removed
- **Ports:** All application ports (8000, 19090, 3000, 5432, 6379) are released

**Optional additional cleanup:**

1. **Remove data volumes (WARNING: This deletes all data):**
   ```bash
   docker-compose down -v
   # This will delete PostgreSQL and Grafana data
   ```

2. **Remove Docker images (if you want to reclaim space):**
   ```bash
   docker-compose down --rmi all
   # This removes all images built for this project
   ```

3. **Force cleanup of unused Docker resources:**
   ```bash
   docker system prune -f
   # Removes stopped containers, unused networks, and dangling images
   ```

### Troubleshooting Stopping Issues

**If containers won't stop:**
```bash
# Force stop all containers
docker-compose kill

# Remove containers by force
docker-compose rm -f
```

**If ports are still in use:**
```bash
# Find processes using the ports
sudo lsof -i :8000
sudo lsof -i :19090
sudo lsof -i :3000
sudo lsof -i :5432
sudo lsof -i :6379

# Kill the processes (replace <PID> with actual process ID)
sudo kill -9 <PID>
```

### Service Access Points

| Service | Port | URL | Description |
|---------|------|-----|-------------|
| Python API | 8000 | http://localhost:8000 | Main application API |
| Prometheus | 19090 | http://localhost:19090 | Metrics collection |
| Grafana | 3000 | http://localhost:3000 | Visualization dashboard |
| PostgreSQL | 5432 | localhost:5432 | Database |
| Redis | 6379 | localhost:6379 | Cache |

### Verification Steps

1. **API Health Check:**
   ```bash
   curl http://localhost:8000/health
   # Expected: {"status": "healthy", "version": "1.0.0"}
   ```

2. **API Root Endpoint:**
   ```bash
   curl http://localhost:8000/
   # Expected: {"message": "Python DevOps API is running!"}
   ```

3. **Prometheus Targets:**
   - Visit http://localhost:19090/targets
   - Verify both 'prometheus' and 'python-devops-api' targets are UP

4. **Grafana Login:**
   - Visit http://localhost:3000
   - Login with username: `admin`, password: `admin`

### Common Issues & Troubleshooting

**Port Conflicts:**
```bash
# Check what's using ports
netstat -tulpn | grep -E ':(8000|19090|3000|5432|6379)'

# Kill conflicting processes or modify docker-compose.yml ports
```

**Container Startup Issues:**
```bash
# Check logs for specific service
docker-compose logs api
docker-compose logs prometheus
docker-compose logs grafana
```

**Permission Issues:**
```bash
# Fix volume permissions
sudo chown -R $USER:$USER .
```

## Manual Setup (Without Docker)

### 1. Python API Installation

1. **Install Python dependencies:**
   ```bash
   python3.11 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Start the API:**
   ```bash
   cd app
   uvicorn main:app --host 0.0.0.0 --port 8000
   ```

3. **Verify API is running:**
   ```bash
   curl http://localhost:8000/health
   ```

### 2. PostgreSQL Setup

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**CentOS/RHEL:**
```bash
sudo yum install postgresql-server postgresql
sudo postgresql-setup initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**Create Database and User:**
```bash
sudo -u postgres psql
CREATE DATABASE devopsdb;
CREATE USER admin WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE devopsdb TO admin;
\q
```

### 3. Redis Setup

**Ubuntu/Debian:**
```bash
sudo apt install redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

**CentOS/RHEL:**
```bash
sudo yum install redis
sudo systemctl start redis
sudo systemctl enable redis
```

### 4. Prometheus Manual Installation

1. **Download and install Prometheus:**
   ```bash
   wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-*.linux-amd64.tar.gz
   tar xvfz prometheus-*.linux-amd64.tar.gz
   cd prometheus-*.linux-amd64
   ```

2. **Create Prometheus configuration:**
   ```bash
   sudo mkdir -p /etc/prometheus
   sudo cp monitoring/prometheus.yml /etc/prometheus/
   ```

3. **Update prometheus.yml for manual setup:**
   ```yaml
   global:
     scrape_interval: 15s
     evaluation_interval: 15s

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']

     - job_name: 'python-devops-api'
       static_configs:
         - targets: ['localhost:8000']  # Changed from 'api:8000'
       metrics_path: '/metrics'
       scrape_interval: 30s
   ```

4. **Create Prometheus user and service:**
   ```bash
   sudo useradd --no-create-home --shell /bin/false prometheus
   sudo mkdir -p /var/lib/prometheus
   sudo chown prometheus:prometheus /var/lib/prometheus
   ```

5. **Create systemd service:**
   ```bash
   sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
   [Unit]
   Description=Prometheus
   Wants=network-online.target
   After=network-online.target

   [Service]
   User=prometheus
   Group=prometheus
   Type=simple
   ExecStart=/usr/local/bin/prometheus \
     --config.file /etc/prometheus/prometheus.yml \
     --storage.tsdb.path /var/lib/prometheus/ \
     --web.console.templates=/etc/prometheus/consoles \
     --web.console.libraries=/etc/prometheus/console_libraries

   [Install]
   WantedBy=multi-user.target
   EOF

   sudo systemctl daemon-reload
   sudo systemctl start prometheus
   sudo systemctl enable prometheus
   ```

6. **Verify Prometheus:**
   - Visit http://localhost:9090
   - Check targets at http://localhost:9090/targets

### 5. Grafana Manual Installation

**Ubuntu/Debian:**
```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
```

**CentOS/RHEL:**
```bash
sudo yum install -y https://dl.grafana.com/oss/release/grafana-latest-1.x86_64.rpm
```

**Start and enable Grafana:**
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

**Configure Grafana:**
1. Visit http://localhost:3000
2. Login with admin/admin (change password on first login)
3. Add Prometheus data source:
   - Go to Configuration > Data Sources > Add data source
   - Select Prometheus
   - URL: `http://localhost:9090`
   - Click Save & Test

## Metrics and Observability

### What Metrics Are Exposed

The Python application exposes metrics at `/metrics` endpoint including:

- **HTTP Request Metrics**: Request count, duration, status codes
- **Application Metrics**: Custom business metrics
- **System Metrics**: CPU, memory, and process information
- **Database Metrics**: Connection pool status, query performance
- **Cache Metrics**: Redis hit/miss ratios

### Metrics Endpoint

```bash
# View all available metrics
curl http://localhost:8000/metrics
```

### Example PromQL Queries

**HTTP Request Rate:**
```promql
rate(http_requests_total[5m])
```

**Request Duration:**
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**Error Rate:**
```promql
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

**Database Connections:**
```promql
pg_stat_activity_count
```

**Redis Memory Usage:**
```promql
redis_memory_used_bytes
```

## Environment Variables & Configuration

### Required Environment Variables

```bash
# Application
ENV=development  # or production
API_HOST=0.0.0.0
API_PORT=8000

# Database
DATABASE_URL=postgresql://admin:password@localhost:5432/devopsdb
POSTGRES_DB=devopsdb
POSTGRES_USER=admin
POSTGRES_PASSWORD=password

# Redis
REDIS_URL=redis://localhost:6379
REDIS_HOST=localhost
REDIS_PORT=6379

# Monitoring
PROMETHEUS_ENDPOINT=http://localhost:9090
GRAFANA_ENDPOINT=http://localhost:3000
```

### Configuration Files

- `monitoring/prometheus.yml`: Prometheus scraping configuration
- `docker-compose.yml`: Docker services definition
- `requirements.txt`: Python dependencies
- `Dockerfile`: Container image specification

## Troubleshooting

### API Not Starting

**Check Python Environment:**
```bash
# Verify Python version
python --version

# Check dependencies
pip list | grep -E "(fastapi|uvicorn|prometheus)"

# Run in debug mode
uvicorn main:app --host 0.0.0.0 --port 8000 --reload --log-level debug
```

**Common Issues:**
- Port 8000 already in use
- Missing Python dependencies
- Database connection failures

### Prometheus Not Scraping Metrics

**Check Configuration:**
```bash
# Validate Prometheus config
promtool check config /etc/prometheus/prometheus.yml

# Check Prometheus logs
sudo journalctl -u prometheus -f

# Test metrics endpoint
curl http://localhost:8000/metrics
```

**Common Issues:**
- Wrong target URL in prometheus.yml
- Firewall blocking connections
- API metrics endpoint not available

### Grafana Dashboards Showing No Data

**Verify Data Source:**
1. Go to Configuration > Data Sources
2. Test Prometheus connection
3. Check for "Data source is working" message

**Common Issues:**
- Incorrect Prometheus URL
- Network connectivity issues
- Prometheus not collecting metrics

### Port Conflicts

**WSL/Linux:**
```bash
# Find processes using ports
sudo ss -tulpn | grep -E ':(8000|9090|3000|5432|6379)'

# Kill processes
sudo kill -9 <PID>

# Alternative: Change ports in docker-compose.yml
```

**Windows:**
```cmd
# Find processes using ports
netstat -ano | findstr ":8000 :9090 :3000 :5432 :6379"

# Kill processes
taskkill /PID <PID> /F
```

## Future Improvements

### Scaling Considerations

1. **Horizontal Scaling:**
   - Load balancer configuration
   - Multiple API instances
   - Database read replicas

2. **Container Orchestration:**
   - Kubernetes deployment (already configured)
   - Service mesh integration
   - Auto-scaling policies

### Monitoring Enhancements

1. **Alerting with Alertmanager:**
   ```yaml
   # Example alert rule
   groups:
   - name: api_alerts
     rules:
     - alert: HighErrorRate
       expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
       for: 2m
       labels:
         severity: critical
       annotations:
         summary: "High error rate detected"
   ```

2. **Custom Metrics:**
   - Business metrics tracking
   - Performance baselines
   - SLA monitoring

3. **Log Aggregation:**
   - ELK stack integration
   - Loki + Grafana
   - Structured logging

### Kubernetes Migration

The repository already includes Kubernetes configurations:

```bash
# Deploy to Kubernetes
kubectl apply -f k8s/
kubectl apply -f monitoring/

# Verify deployment
kubectl get pods -l app=python-devops-api
kubectl get services
```

### Security Hardening

1. **Network Policies:**
   - Restrict inter-service communication
   - Implement ingress controllers
   - TLS encryption everywhere

2. **Secrets Management:**
   - Kubernetes secrets
   - HashiCorp Vault
   - Environment variable encryption

3. **Compliance:**
   - Audit logging
   - Data encryption at rest
   - Access control policies

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
- Check the troubleshooting section above
- Create an issue in the repository
- Review existing documentation and discussions