# Server Configuration Guide

This guide covers configuration options for the Revio 4 server.

## Configuration File Location

The main server configuration file is located at:

- **Linux**: `/etc/revio4/server.conf`
- **Windows**: `C:\Program Files\Revio4\Server\config\server.conf`
- **Docker**: `/etc/revio4/server.conf` (in container)

## Configuration File Format

The configuration file uses INI format:

```ini
[server]
host = 0.0.0.0
port = 8443
protocol = https
max_connections = 1000
timeout = 300

[database]
type = postgresql
host = localhost
port = 5432
name = revio4
user = revio4user
password = secure_password
pool_size = 20
pool_timeout = 30

[ssl]
enabled = true
certificate = /etc/revio4/ssl/server.crt
private_key = /etc/revio4/ssl/server.key
ca_certificate = /etc/revio4/ssl/ca.crt

[security]
jwt_secret = your-secret-key-change-this
jwt_expiration = 3600
password_min_length = 8
session_timeout = 1800
enable_2fa = false

[logging]
level = info
file = /var/log/revio4/server.log
max_size = 100MB
max_files = 10
console = true

[performance]
cache_enabled = true
cache_size = 512MB
compression = true
worker_processes = auto

[email]
enabled = false
smtp_host = smtp.example.com
smtp_port = 587
smtp_user = noreply@example.com
smtp_password = email_password
smtp_tls = true
from_address = Revio 4 <noreply@example.com>
```

## Server Settings

### host
- **Type**: String
- **Default**: `0.0.0.0`
- **Description**: Interface to bind to
- **Options**: 
  - `0.0.0.0` - All interfaces
  - `127.0.0.1` - Localhost only
  - Specific IP address

### port
- **Type**: Integer
- **Default**: `8443`
- **Description**: Port number for server to listen on
- **Range**: 1-65535
- **Note**: Ports below 1024 require root/admin privileges

### protocol
- **Type**: String
- **Default**: `https`
- **Options**: `http`, `https`
- **Production**: Always use `https`

### max_connections
- **Type**: Integer
- **Default**: `1000`
- **Description**: Maximum concurrent client connections
- **Tuning**: Adjust based on expected load and system resources

### timeout
- **Type**: Integer (seconds)
- **Default**: `300` (5 minutes)
- **Description**: Connection timeout duration

## Database Settings

### type
- **Type**: String
- **Options**: `postgresql`, `mysql`
- **Default**: `postgresql`
- **Recommended**: PostgreSQL for production

### host
- **Type**: String
- **Default**: `localhost`
- **Description**: Database server hostname or IP

### port
- **Type**: Integer
- **Default**: `5432` (PostgreSQL), `3306` (MySQL)
- **Description**: Database server port

### name
- **Type**: String
- **Description**: Database name

### user
- **Type**: String
- **Description**: Database username

### password
- **Type**: String
- **Description**: Database password
- **Security**: Use environment variables in production

### pool_size
- **Type**: Integer
- **Default**: `20`
- **Description**: Database connection pool size
- **Tuning**: Increase for high-load scenarios

### pool_timeout
- **Type**: Integer (seconds)
- **Default**: `30`
- **Description**: Connection pool timeout

## SSL/TLS Configuration

### enabled
- **Type**: Boolean
- **Default**: `true`
- **Production**: Must be `true`

### certificate
- **Type**: String (file path)
- **Description**: Path to SSL certificate file
- **Format**: PEM format

### private_key
- **Type**: String (file path)
- **Description**: Path to private key file
- **Format**: PEM format
- **Security**: Restrict file permissions (600)

### ca_certificate
- **Type**: String (file path)
- **Optional**: Yes
- **Description**: Path to CA certificate for client verification

### Generating Self-Signed Certificate (Development Only)

```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate signing request
openssl req -new -key server.key -out server.csr

# Generate self-signed certificate
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# Set permissions
chmod 600 server.key
chmod 644 server.crt
```

### Production Certificate Setup

1. **Obtain Certificate from CA**
   - Let's Encrypt (free, automated)
   - Commercial CA (DigiCert, GlobalSign, etc.)

2. **Let's Encrypt with Certbot**

```bash
# Install Certbot
sudo apt install certbot

# Generate certificate
sudo certbot certonly --standalone -d revio4.example.com

# Certificate location
# /etc/letsencrypt/live/revio4.example.com/fullchain.pem
# /etc/letsencrypt/live/revio4.example.com/privkey.pem

# Update configuration
[ssl]
certificate = /etc/letsencrypt/live/revio4.example.com/fullchain.pem
private_key = /etc/letsencrypt/live/revio4.example.com/privkey.pem

# Set up auto-renewal
sudo certbot renew --dry-run
```

## Security Settings

### jwt_secret
- **Type**: String
- **Description**: Secret key for JWT token signing
- **Important**: Must be changed from default
- **Length**: Minimum 32 characters
- **Generation**:
  ```bash
  openssl rand -base64 32
  ```

### jwt_expiration
- **Type**: Integer (seconds)
- **Default**: `3600` (1 hour)
- **Description**: JWT token expiration time

### password_min_length
- **Type**: Integer
- **Default**: `8`
- **Recommended**: Minimum 8, prefer 12+

### session_timeout
- **Type**: Integer (seconds)
- **Default**: `1800` (30 minutes)
- **Description**: Idle session timeout

### enable_2fa
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable two-factor authentication
- **Recommended**: `true` for production

### Additional Security Options

```ini
[security]
# Rate limiting
rate_limit_enabled = true
rate_limit_requests = 100
rate_limit_window = 60

# CORS settings
cors_enabled = true
cors_allowed_origins = https://client.example.com

# Security headers
hsts_enabled = true
hsts_max_age = 31536000

# IP filtering
ip_whitelist = 192.168.1.0/24,10.0.0.0/8
ip_blacklist = 

# Brute force protection
max_login_attempts = 5
lockout_duration = 900
```

## Logging Configuration

### level
- **Type**: String
- **Options**: `debug`, `info`, `warning`, `error`, `critical`
- **Default**: `info`
- **Production**: `warning` or `error`

### file
- **Type**: String (file path)
- **Description**: Log file path
- **Rotation**: Automatic with max_size and max_files

### max_size
- **Type**: String (size)
- **Default**: `100MB`
- **Description**: Maximum log file size before rotation

### max_files
- **Type**: Integer
- **Default**: `10`
- **Description**: Number of rotated log files to keep

### console
- **Type**: Boolean
- **Default**: `true`
- **Description**: Also log to console/stdout

### Log Format Configuration

```ini
[logging]
format = json
timestamp_format = iso8601
include_request_id = true
include_user_id = true
```

## Performance Settings

### cache_enabled
- **Type**: Boolean
- **Default**: `true`
- **Description**: Enable in-memory caching

### cache_size
- **Type**: String (size)
- **Default**: `512MB`
- **Description**: Maximum cache size

### compression
- **Type**: Boolean
- **Default**: `true`
- **Description**: Enable response compression
- **Formats**: gzip, deflate

### worker_processes
- **Type**: Integer or `auto`
- **Default**: `auto`
- **Description**: Number of worker processes
- **Auto**: Sets to number of CPU cores

### Performance Tuning

```ini
[performance]
# Connection pool
keep_alive_timeout = 65
keep_alive_requests = 100

# Request handling
max_request_size = 10MB
request_timeout = 60

# Database query cache
query_cache_enabled = true
query_cache_ttl = 300

# Background tasks
background_workers = 4
task_queue_size = 1000
```

## Email Configuration

### enabled
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable email functionality

### smtp_host
- **Type**: String
- **Description**: SMTP server hostname

### smtp_port
- **Type**: Integer
- **Default**: `587` (STARTTLS), `465` (SSL)
- **Description**: SMTP server port

### smtp_user
- **Type**: String
- **Description**: SMTP authentication username

### smtp_password
- **Type**: String
- **Description**: SMTP authentication password

### smtp_tls
- **Type**: Boolean
- **Default**: `true`
- **Description**: Use TLS for SMTP

### from_address
- **Type**: String
- **Description**: Default from email address

### Email Templates

```ini
[email]
template_path = /etc/revio4/templates/email
send_welcome = true
send_password_reset = true
send_notifications = true
```

## Environment Variable Configuration

Override configuration with environment variables:

```bash
# Server settings
export REVIO4_SERVER_PORT=8443
export REVIO4_SERVER_HOST=0.0.0.0

# Database settings
export REVIO4_DB_HOST=localhost
export REVIO4_DB_PORT=5432
export REVIO4_DB_NAME=revio4
export REVIO4_DB_USER=revio4user
export REVIO4_DB_PASSWORD=secure_password

# Security
export REVIO4_JWT_SECRET=your-secret-key

# Logging
export REVIO4_LOG_LEVEL=info
```

Priority: Environment Variables > Config File > Defaults

## Advanced Configuration

### High Availability Setup

```ini
[cluster]
enabled = true
node_id = node-1
discovery_method = static
cluster_nodes = node-1:8443,node-2:8443,node-3:8443
heartbeat_interval = 5
failover_timeout = 30

[redis]
enabled = true
host = localhost
port = 6379
password = redis_password
db = 0
```

### Load Balancer Configuration

When behind a load balancer:

```ini
[server]
behind_proxy = true
proxy_protocol = true
trusted_proxies = 192.168.1.10,192.168.1.11

[headers]
real_ip_header = X-Forwarded-For
real_proto_header = X-Forwarded-Proto
```

### Monitoring Integration

```ini
[monitoring]
enabled = true
metrics_port = 9090
metrics_path = /metrics
health_check_path = /health

[prometheus]
enabled = true
scrape_interval = 15

[alerts]
webhook_url = https://alerts.example.com/webhook
alert_on_errors = true
alert_threshold = 100
```

## Configuration Validation

### Validate Configuration

```bash
# Check configuration syntax
sudo revio4-server check-config

# Test configuration with dry-run
sudo revio4-server --dry-run --config /etc/revio4/server.conf

# Validate SSL certificates
sudo revio4-server check-ssl
```

### Configuration Examples

#### Minimal Configuration

```ini
[server]
host = 0.0.0.0
port = 8443

[database]
host = localhost
name = revio4
user = revio4user
password = secure_password

[ssl]
enabled = false
```

#### Production Configuration

```ini
[server]
host = 0.0.0.0
port = 8443
protocol = https
max_connections = 5000

[database]
type = postgresql
host = db.example.com
port = 5432
name = revio4_prod
user = revio4user
password = ${DB_PASSWORD}
pool_size = 50

[ssl]
enabled = true
certificate = /etc/letsencrypt/live/revio4.example.com/fullchain.pem
private_key = /etc/letsencrypt/live/revio4.example.com/privkey.pem

[security]
jwt_secret = ${JWT_SECRET}
enable_2fa = true
rate_limit_enabled = true

[logging]
level = warning
file = /var/log/revio4/server.log

[performance]
cache_enabled = true
cache_size = 2GB
worker_processes = 8
```

## Configuration Management

### Version Control

Store configuration templates in version control:

```bash
# Template with placeholders
/etc/revio4/server.conf.template

# Deployment script replaces placeholders
sed -e "s/{{DB_PASSWORD}}/$DB_PASSWORD/" \
    -e "s/{{JWT_SECRET}}/$JWT_SECRET/" \
    server.conf.template > server.conf
```

### Configuration Backup

```bash
# Backup configuration
sudo cp /etc/revio4/server.conf /etc/revio4/server.conf.backup

# Backup with timestamp
sudo cp /etc/revio4/server.conf \
    /etc/revio4/server.conf.$(date +%Y%m%d_%H%M%S)
```

### Reload Configuration

```bash
# Reload without downtime (if supported)
sudo systemctl reload revio4-server

# Or restart service
sudo systemctl restart revio4-server
```

## Troubleshooting Configuration

### Configuration Errors

```bash
# View configuration errors in logs
sudo journalctl -u revio4-server -n 100 | grep -i error

# Validate database connection
sudo revio4-server test-db --config /etc/revio4/server.conf

# Check port availability
sudo netstat -tulpn | grep 8443
```

### Permission Issues

```bash
# Fix file permissions
sudo chown revio4:revio4 /etc/revio4/server.conf
sudo chmod 640 /etc/revio4/server.conf

# Fix SSL key permissions
sudo chmod 600 /etc/revio4/ssl/server.key
sudo chown revio4:revio4 /etc/revio4/ssl/*
```

## Next Steps

- [Deploy to Production](deployment.md)
- [Monitor Server Performance](deployment.md#monitoring)
- [Configure Backups](deployment.md#backup-configuration)
- [Review Security Best Practices](deployment.md#security-hardening)
