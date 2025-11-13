# Server Installation Guide

This guide provides comprehensive instructions for installing the Revio 4 server on various platforms.

## Prerequisites

Before installing the Revio 4 server:

1. Review [System Requirements](../system-requirements.md)
2. Ensure you have administrator/root privileges
3. Prepare database server (PostgreSQL or MySQL)
4. Obtain SSL/TLS certificate (for production)
5. Configure firewall and network settings

## Pre-Installation Checklist

- [ ] Server meets minimum hardware requirements
- [ ] Operating system is up to date
- [ ] Database server is installed and running
- [ ] Required ports are available (8443, 5432, etc.)
- [ ] DNS records configured (if using domain name)
- [ ] SSL certificate obtained (production)
- [ ] Backup solution in place

## Linux Installation (Ubuntu/Debian)

### Step 1: System Preparation

```bash
# Update system packages
sudo apt update
sudo apt upgrade -y

# Install required dependencies
sudo apt install -y curl wget gnupg2 software-properties-common
```

### Step 2: Install Database (PostgreSQL)

```bash
# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Start and enable PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE revio4;
CREATE USER revio4user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE revio4 TO revio4user;
\q
EOF
```

### Step 3: Add Revio 4 Repository

```bash
# Add repository GPG key
wget -qO - https://repo.example.com/gpg.key | sudo apt-key add -

# Add repository
echo "deb [trusted=yes] https://repo.example.com/apt stable main" | \
    sudo tee /etc/apt/sources.list.d/revio4.list

# Update package list
sudo apt update
```

### Step 4: Install Revio 4 Server

```bash
# Install server package
sudo apt install -y revio4-server

# Verify installation
revio4-server --version
```

### Step 5: Initial Configuration

```bash
# Edit configuration file
sudo nano /etc/revio4/server.conf

# Initialize database
sudo revio4-server init-db

# Start the service
sudo systemctl start revio4-server
sudo systemctl enable revio4-server

# Check status
sudo systemctl status revio4-server
```

## Linux Installation (Red Hat/CentOS/Fedora)

### Step 1: System Preparation

```bash
# Update system packages
sudo dnf update -y

# Install required dependencies
sudo dnf install -y curl wget gnupg2
```

### Step 2: Install Database (PostgreSQL)

```bash
# Install PostgreSQL
sudo dnf install -y postgresql-server postgresql-contrib

# Initialize database
sudo postgresql-setup --initdb

# Start and enable PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE revio4;
CREATE USER revio4user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE revio4 TO revio4user;
\q
EOF

# Configure PostgreSQL authentication
sudo nano /var/lib/pgsql/data/pg_hba.conf
# Change 'peer' to 'md5' for local connections

sudo systemctl restart postgresql
```

### Step 3: Add Revio 4 Repository

```bash
# Create repository configuration
sudo tee /etc/yum.repos.d/revio4.repo << EOF
[revio4]
name=Revio 4 Repository
baseurl=https://repo.example.com/yum/revio4
enabled=1
gpgcheck=1
gpgkey=https://repo.example.com/gpg.key
EOF

# Update package list
sudo dnf makecache
```

### Step 4: Install Revio 4 Server

```bash
# Install server package
sudo dnf install -y revio4-server

# Verify installation
revio4-server --version
```

### Step 5: Initial Configuration

```bash
# Edit configuration file
sudo nano /etc/revio4/server.conf

# Initialize database
sudo revio4-server init-db

# Configure firewall
sudo firewall-cmd --permanent --add-port=8443/tcp
sudo firewall-cmd --reload

# Start the service
sudo systemctl start revio4-server
sudo systemctl enable revio4-server

# Check status
sudo systemctl status revio4-server
```

## Windows Server Installation

### Step 1: Install Prerequisites

1. **Install .NET Framework**
   - Download .NET Framework 4.7.2 or later
   - Run installer and follow prompts

2. **Install Database**
   - Download PostgreSQL for Windows
   - Run installer, set password
   - Create database:
   ```sql
   CREATE DATABASE revio4;
   CREATE USER revio4user WITH PASSWORD 'secure_password';
   GRANT ALL PRIVILEGES ON DATABASE revio4 TO revio4user;
   ```

### Step 2: Install Revio 4 Server

1. **Download Installer**
   - Download `Revio4-Server-Setup.exe`

2. **Run Installer**
   - Right-click and "Run as administrator"
   - Accept license agreement
   - Choose installation directory (default: `C:\Program Files\Revio4\Server`)
   - Configure service account

3. **Complete Installation**
   - Click "Install"
   - Wait for completion

### Step 3: Configure Server

1. **Edit Configuration**
   - Open `C:\Program Files\Revio4\Server\config\server.conf`
   - Configure database connection
   - Set server parameters

2. **Initialize Database**
   ```cmd
   cd "C:\Program Files\Revio4\Server\bin"
   revio4-server.exe init-db
   ```

3. **Configure Windows Firewall**
   ```powershell
   New-NetFirewallRule -DisplayName "Revio 4 Server" -Direction Inbound `
       -Protocol TCP -LocalPort 8443 -Action Allow
   ```

### Step 4: Start Service

1. **Via Services Management**
   - Open Services (services.msc)
   - Find "Revio 4 Server"
   - Set startup type to "Automatic"
   - Start the service

2. **Via Command Line**
   ```cmd
   net start Revio4Server
   ```

## Docker Installation

### Using Docker Compose

1. **Create docker-compose.yml**

```yaml
version: '3.8'

services:
  database:
    image: postgres:14
    environment:
      POSTGRES_DB: revio4
      POSTGRES_USER: revio4user
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - revio4-network

  revio4-server:
    image: revio4/server:4.0.0
    ports:
      - "8443:8443"
    environment:
      DB_HOST: database
      DB_PORT: 5432
      DB_NAME: revio4
      DB_USER: revio4user
      DB_PASSWORD: secure_password
    volumes:
      - server_config:/etc/revio4
      - server_data:/var/lib/revio4
    depends_on:
      - database
    networks:
      - revio4-network

volumes:
  postgres_data:
  server_config:
  server_data:

networks:
  revio4-network:
    driver: bridge
```

2. **Deploy with Docker Compose**

```bash
# Start services
docker-compose up -d

# Initialize database
docker-compose exec revio4-server revio4-server init-db

# View logs
docker-compose logs -f revio4-server

# Check status
docker-compose ps
```

### Using Docker CLI

```bash
# Create network
docker network create revio4-network

# Start database
docker run -d \
  --name revio4-postgres \
  --network revio4-network \
  -e POSTGRES_DB=revio4 \
  -e POSTGRES_USER=revio4user \
  -e POSTGRES_PASSWORD=secure_password \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:14

# Start Revio 4 server
docker run -d \
  --name revio4-server \
  --network revio4-network \
  -p 8443:8443 \
  -e DB_HOST=revio4-postgres \
  -e DB_PORT=5432 \
  -e DB_NAME=revio4 \
  -e DB_USER=revio4user \
  -e DB_PASSWORD=secure_password \
  -v server_config:/etc/revio4 \
  -v server_data:/var/lib/revio4 \
  revio4/server:4.0.0

# Initialize database
docker exec revio4-server revio4-server init-db
```

## Cloud Platform Installation

### AWS Installation

1. **Launch EC2 Instance**
   - Choose Ubuntu Server 22.04 LTS AMI
   - Instance type: t3.medium or larger
   - Configure security group (port 8443)

2. **Install via User Data Script**

```bash
#!/bin/bash
apt update
apt install -y postgresql postgresql-contrib
systemctl start postgresql
systemctl enable postgresql

# Create database
sudo -u postgres psql -c "CREATE DATABASE revio4;"
sudo -u postgres psql -c "CREATE USER revio4user WITH PASSWORD 'secure_password';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE revio4 TO revio4user;"

# Install Revio 4
wget -qO - https://repo.example.com/gpg.key | apt-key add -
echo "deb https://repo.example.com/apt stable main" > /etc/apt/sources.list.d/revio4.list
apt update
apt install -y revio4-server

# Configure and start
revio4-server init-db
systemctl start revio4-server
systemctl enable revio4-server
```

### Azure Installation

1. **Create Virtual Machine**
   - Choose Ubuntu Server 22.04 LTS
   - Size: Standard_B2s or larger
   - Configure NSG (allow port 8443)

2. **Connect and Install**
   - SSH to VM
   - Follow Linux installation steps above

### Google Cloud Platform

1. **Create Compute Engine Instance**
   - Machine type: e2-medium or larger
   - Boot disk: Ubuntu 22.04 LTS
   - Firewall: Allow HTTPS traffic

2. **Install Server**
   - SSH to instance
   - Follow Linux installation steps

## Post-Installation Steps

### 1. Verify Installation

```bash
# Check service status
sudo systemctl status revio4-server

# Check logs
sudo journalctl -u revio4-server -n 50

# Test connectivity
curl -k https://localhost:8443/health
```

### 2. Configure SSL/TLS

```bash
# Copy certificate files
sudo cp server.crt /etc/revio4/ssl/
sudo cp server.key /etc/revio4/ssl/

# Update configuration
sudo nano /etc/revio4/server.conf
# ssl.certificate=/etc/revio4/ssl/server.crt
# ssl.key=/etc/revio4/ssl/server.key

# Restart service
sudo systemctl restart revio4-server
```

### 3. Create Admin User

```bash
sudo revio4-server create-admin --username admin --email admin@example.com
```

### 4. Configure Backup

```bash
# Create backup script
sudo nano /usr/local/bin/revio4-backup.sh

# Add to crontab
sudo crontab -e
# 0 2 * * * /usr/local/bin/revio4-backup.sh
```

## Troubleshooting Installation

### Service Won't Start

```bash
# Check logs
sudo journalctl -u revio4-server -xe

# Verify configuration
sudo revio4-server check-config

# Test database connection
sudo revio4-server test-db
```

### Database Connection Fails

1. Verify database is running
2. Check credentials in configuration
3. Test database connectivity:
```bash
psql -h localhost -U revio4user -d revio4
```

### Port Already in Use

```bash
# Check what's using the port
sudo lsof -i :8443
sudo netstat -tulpn | grep 8443

# Change port in configuration or stop conflicting service
```

### Permission Issues

```bash
# Fix file permissions
sudo chown -R revio4:revio4 /var/lib/revio4
sudo chmod 750 /etc/revio4
```

## Next Steps

- [Configure the Server](configuration.md)
- [Deploy to Production](deployment.md)
- [Review Security Settings](configuration.md#security-settings)
- [Set Up Monitoring](deployment.md#monitoring)

## Additional Resources

- [System Requirements](../system-requirements.md)
- [Troubleshooting Guide](../troubleshooting.md)
- [Getting Started](../getting-started.md)
