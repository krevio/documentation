# Server Deployment Guide

This guide covers deployment strategies, best practices, and operational procedures for Revio 4 server in production environments.

## Deployment Overview

A production deployment of Revio 4 server requires careful planning and execution. This guide covers:

- Pre-deployment checklist
- Deployment strategies
- High availability setup
- Monitoring and maintenance
- Backup and disaster recovery

## Pre-Deployment Checklist

Before deploying to production:

- [ ] Server hardware meets requirements
- [ ] Operating system hardened and updated
- [ ] Database configured and optimized
- [ ] SSL/TLS certificates obtained and installed
- [ ] Firewall rules configured
- [ ] Network infrastructure ready (load balancers, DNS)
- [ ] Backup solution implemented
- [ ] Monitoring tools configured
- [ ] Documentation reviewed
- [ ] Disaster recovery plan in place
- [ ] Testing completed in staging environment
- [ ] Rollback plan prepared

## Deployment Strategies

### Single Server Deployment

Suitable for small deployments and development environments.

**Architecture:**
```
┌─────────────────┐
│  Revio 4 Server │
│   + Database    │
└─────────────────┘
```

**Setup:**

1. Install server and database on same machine
2. Configure according to [Installation Guide](installation.md)
3. Set up local backups
4. Configure monitoring

**Pros:** Simple, low cost
**Cons:** No redundancy, single point of failure

### Separated Database Deployment

Recommended for medium-sized deployments.

**Architecture:**
```
┌─────────────────┐      ┌──────────────┐
│  Revio 4 Server │─────►│   Database   │
└─────────────────┘      └──────────────┘
```

**Setup:**

1. Deploy database server separately
2. Configure server to connect to remote database
3. Secure database connection
4. Set up database replication (recommended)

**Pros:** Better resource allocation, easier scaling
**Cons:** More complex, network dependency

### High Availability Deployment

Required for production environments with uptime requirements.

**Architecture:**
```
                ┌──────────────────┐
                │  Load Balancer   │
                └────────┬─────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼────────┐ ┌─────▼──────────┐ ┌──▼────────────┐
│ Server Node 1  │ │ Server Node 2  │ │ Server Node 3 │
└───────┬────────┘ └─────┬──────────┘ └──┬────────────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
                ┌────────▼─────────┐
                │ Database Cluster │
                │  (Primary + HA)  │
                └──────────────────┘
```

**Components:**

1. **Load Balancer** (HAProxy, Nginx, or cloud LB)
2. **Multiple Server Nodes** (3+ recommended)
3. **Database Cluster** (PostgreSQL with replication)
4. **Shared Cache** (Redis for session management)
5. **Centralized Logging**
6. **Monitoring System**

## High Availability Setup

### Load Balancer Configuration

#### HAProxy Configuration

```haproxy
global
    log /dev/log local0
    maxconn 4096
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend revio4_frontend
    bind *:443 ssl crt /etc/haproxy/certs/revio4.pem
    mode http
    option forwardfor
    reqadd X-Forwarded-Proto:\ https
    default_backend revio4_servers

backend revio4_servers
    mode http
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    server node1 192.168.1.101:8443 check ssl verify none
    server node2 192.168.1.102:8443 check ssl verify none
    server node3 192.168.1.103:8443 check ssl verify none

listen stats
    bind *:9000
    stats enable
    stats uri /stats
    stats refresh 30s
```

#### Nginx Load Balancer

```nginx
upstream revio4_backend {
    least_conn;
    server 192.168.1.101:8443 max_fails=3 fail_timeout=30s;
    server 192.168.1.102:8443 max_fails=3 fail_timeout=30s;
    server 192.168.1.103:8443 max_fails=3 fail_timeout=30s;
}

server {
    listen 443 ssl http2;
    server_name revio4.example.com;

    ssl_certificate /etc/nginx/ssl/revio4.crt;
    ssl_certificate_key /etc/nginx/ssl/revio4.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass https://revio4_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        proxy_pass https://revio4_backend/health;
        access_log off;
    }
}
```

### Database High Availability

#### PostgreSQL Replication

**Primary Server Configuration** (`postgresql.conf`):

```ini
# Replication settings
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
wal_keep_size = 1GB
hot_standby = on
```

**Create Replication User:**

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN ENCRYPTED PASSWORD 'repl_password';
```

**Configure Access** (`pg_hba.conf`):

```
host replication replicator 192.168.1.0/24 md5
```

**Standby Server Setup:**

```bash
# Stop PostgreSQL on standby
sudo systemctl stop postgresql

# Remove existing data
sudo rm -rf /var/lib/postgresql/14/main

# Create base backup
sudo -u postgres pg_basebackup -h 192.168.1.101 -D /var/lib/postgresql/14/main \
    -U replicator -P -v -R

# Start PostgreSQL
sudo systemctl start postgresql
```

**Verify Replication:**

```sql
-- On primary
SELECT * FROM pg_stat_replication;

-- On standby
SELECT * FROM pg_stat_wal_receiver;
```

### Session Management with Redis

**Install Redis:**

```bash
sudo apt install redis-server
```

**Configure Redis** (`/etc/redis/redis.conf`):

```ini
bind 0.0.0.0
protected-mode yes
requirepass redis_password
maxmemory 2gb
maxmemory-policy allkeys-lru
```

**Server Configuration:**

```ini
[session]
backend = redis
redis_host = redis.example.com
redis_port = 6379
redis_password = redis_password
redis_db = 0
```

## Container Orchestration

### Kubernetes Deployment

**Deployment YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: revio4-server
  namespace: revio4
spec:
  replicas: 3
  selector:
    matchLabels:
      app: revio4-server
  template:
    metadata:
      labels:
        app: revio4-server
    spec:
      containers:
      - name: revio4-server
        image: revio4/server:4.0.0
        ports:
        - containerPort: 8443
        env:
        - name: DB_HOST
          value: postgres-service
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: revio4-secrets
              key: db-password
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8443
            scheme: HTTPS
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8443
            scheme: HTTPS
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: revio4-service
  namespace: revio4
spec:
  type: LoadBalancer
  selector:
    app: revio4-server
  ports:
  - protocol: TCP
    port: 443
    targetPort: 8443
```

**Deploy:**

```bash
kubectl apply -f revio4-deployment.yaml
kubectl get pods -n revio4
kubectl get services -n revio4
```

## Monitoring

### Prometheus + Grafana Setup

**Install Prometheus:**

```bash
# Download and extract
wget https://github.com/prometheus/prometheus/releases/download/v2.40.0/prometheus-2.40.0.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
cd prometheus-*

# Configure
cat > prometheus.yml <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'revio4'
    static_configs:
      - targets: ['localhost:9090']
      - targets: ['192.168.1.101:9090']
      - targets: ['192.168.1.102:9090']
      - targets: ['192.168.1.103:9090']
EOF

# Run
./prometheus --config.file=prometheus.yml
```

**Install Grafana:**

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

**Key Metrics to Monitor:**

- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- Request rate
- Response time
- Error rate
- Database connections
- Cache hit rate
- Queue depth

### Log Aggregation

**Using ELK Stack (Elasticsearch, Logstash, Kibana):**

**Filebeat Configuration** (`/etc/filebeat/filebeat.yml`):

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/revio4/*.log
  fields:
    service: revio4-server
    environment: production

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "revio4-logs-%{+yyyy.MM.dd}"

processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
```

### Health Checks

**Configure Health Check Endpoint:**

```bash
# Test health endpoint
curl -k https://localhost:8443/health

# Expected response
{
  "status": "healthy",
  "version": "4.0.0",
  "database": "connected",
  "cache": "active"
}
```

## Backup Configuration

### Automated Database Backup

**Backup Script** (`/usr/local/bin/revio4-backup.sh`):

```bash
#!/bin/bash

# Configuration
BACKUP_DIR="/var/backups/revio4"
DB_NAME="revio4"
DB_USER="revio4user"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup
pg_dump -U $DB_USER $DB_NAME | gzip > "$BACKUP_DIR/db_backup_$DATE.sql.gz"

# Configuration backup
tar -czf "$BACKUP_DIR/config_backup_$DATE.tar.gz" /etc/revio4

# Data backup
tar -czf "$BACKUP_DIR/data_backup_$DATE.tar.gz" /var/lib/revio4

# Remove old backups
find $BACKUP_DIR -type f -mtime +$RETENTION_DAYS -delete

# Verify backup
if [ -f "$BACKUP_DIR/db_backup_$DATE.sql.gz" ]; then
    echo "Backup successful: $DATE"
else
    echo "Backup failed: $DATE" >&2
    exit 1
fi
```

**Make Executable and Schedule:**

```bash
sudo chmod +x /usr/local/bin/revio4-backup.sh

# Add to crontab (daily at 2 AM)
sudo crontab -e
0 2 * * * /usr/local/bin/revio4-backup.sh >> /var/log/revio4/backup.log 2>&1
```

### Restore from Backup

```bash
# Restore database
gunzip -c /var/backups/revio4/db_backup_20240101_020000.sql.gz | \
    psql -U revio4user revio4

# Restore configuration
tar -xzf /var/backups/revio4/config_backup_20240101_020000.tar.gz -C /

# Restore data
tar -xzf /var/backups/revio4/data_backup_20240101_020000.tar.gz -C /

# Restart service
sudo systemctl restart revio4-server
```

## Security Hardening

### Firewall Configuration

**UFW (Ubuntu):**

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTPS
sudo ufw allow 8443/tcp

# Allow database (from application servers only)
sudo ufw allow from 192.168.1.0/24 to any port 5432

# Enable firewall
sudo ufw enable
```

**firewalld (Red Hat/CentOS):**

```bash
# Add services
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-port=8443/tcp

# Reload
sudo firewall-cmd --reload
```

### SELinux Configuration

```bash
# Check SELinux status
sestatus

# Create custom policy if needed
audit2allow -a -M revio4
semodule -i revio4.pp

# Or set permissive for testing
sudo setenforce 0
```

### Regular Security Updates

```bash
# Automated security updates (Ubuntu)
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Or manual updates
sudo apt update
sudo apt upgrade
```

## Deployment Procedures

### Zero-Downtime Deployment

1. **Deploy new version to staging**
2. **Test thoroughly**
3. **Deploy to one production node**
4. **Monitor for issues**
5. **If successful, deploy to remaining nodes**
6. **If issues, rollback affected node**

**Deployment Script:**

```bash
#!/bin/bash

NODES=("node1" "node2" "node3")
NEW_VERSION="4.1.0"

for node in "${NODES[@]}"; do
    echo "Deploying to $node..."
    
    # Remove from load balancer
    ssh $node "sudo systemctl stop revio4-server"
    
    # Update software
    ssh $node "sudo apt update && sudo apt install revio4-server=$NEW_VERSION"
    
    # Run migrations if needed
    ssh $node "sudo revio4-server migrate"
    
    # Start service
    ssh $node "sudo systemctl start revio4-server"
    
    # Health check
    sleep 10
    if curl -k https://$node:8443/health | grep -q "healthy"; then
        echo "$node deployed successfully"
    else
        echo "$node deployment failed, rolling back"
        ssh $node "sudo apt install revio4-server=4.0.0"
        ssh $node "sudo systemctl restart revio4-server"
        exit 1
    fi
    
    # Delay before next node
    sleep 30
done

echo "Deployment complete"
```

### Rollback Procedure

```bash
# Quick rollback
sudo apt install revio4-server=<previous-version>
sudo systemctl restart revio4-server

# Database rollback (if migrations were run)
sudo revio4-server migrate-down
```

## Troubleshooting

### Service Not Starting

```bash
# Check service status
sudo systemctl status revio4-server

# View logs
sudo journalctl -u revio4-server -n 100 --no-pager

# Check configuration
sudo revio4-server check-config

# Test database connection
sudo revio4-server test-db
```

### Performance Issues

```bash
# Check system resources
top
htop
iostat -x 1
vmstat 1

# Check database performance
# PostgreSQL
SELECT * FROM pg_stat_activity;
SELECT * FROM pg_stat_database;

# Check connection pool
sudo revio4-server pool-status
```

### High Memory Usage

```bash
# Check memory usage
free -h
ps aux --sort=-%mem | head

# Adjust cache size in configuration
[performance]
cache_size = 256MB  # Reduce if needed

# Restart service
sudo systemctl restart revio4-server
```

## Maintenance

### Regular Maintenance Tasks

**Daily:**
- Monitor system health
- Check error logs
- Verify backups completed

**Weekly:**
- Review performance metrics
- Check disk space
- Update documentation

**Monthly:**
- Apply security updates
- Review and optimize database
- Test disaster recovery procedures
- Audit user accounts and permissions

### Database Maintenance

```bash
# Analyze database
sudo -u postgres vacuumdb -z revio4

# Reindex database
sudo -u postgres reindexdb revio4

# Check database size
sudo -u postgres psql -c "SELECT pg_database_size('revio4');"
```

## Next Steps

- [Configure Server](configuration.md)
- [Review Troubleshooting Guide](../troubleshooting.md)
- [Set Up Monitoring Dashboard](configuration.md#monitoring-integration)
- [Plan Disaster Recovery](../troubleshooting.md)
