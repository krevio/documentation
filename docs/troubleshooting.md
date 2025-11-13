# Troubleshooting Guide

This guide helps diagnose and resolve common issues with Revio 4 client and server installations.

## General Troubleshooting Approach

1. **Identify the Problem**: Clearly define what's not working
2. **Check Logs**: Review relevant log files for errors
3. **Verify Configuration**: Ensure settings are correct
4. **Test Connectivity**: Verify network connections
5. **Isolate the Issue**: Determine if it's client, server, or network related
6. **Apply Solutions**: Try solutions from most to least likely
7. **Document**: Record the issue and solution for future reference

## Client Issues

### Client Won't Start

**Symptoms:**
- Application crashes on startup
- Error messages during launch
- Application doesn't appear

**Solutions:**

1. **Check System Requirements**
   ```bash
   # Verify minimum requirements are met
   # See System Requirements documentation
   ```

2. **Review Log Files**
   - **Windows**: `%LOCALAPPDATA%\Revio4\logs\client.log`
   - **macOS**: `~/Library/Logs/Revio4/client.log`
   - **Linux**: `~/.local/share/revio4/logs/client.log`

3. **Reset Configuration**
   ```bash
   # Backup and remove config file
   # Windows
   rename "%APPDATA%\Revio4\config.json" config.json.bak
   
   # macOS/Linux
   mv ~/.config/revio4/config.json ~/.config/revio4/config.json.bak
   ```

4. **Reinstall Application**
   - Uninstall completely
   - Delete configuration directories
   - Reinstall from scratch

5. **Check Permissions**
   ```bash
   # Linux/macOS - ensure execute permissions
   chmod +x /path/to/revio4-client
   ```

### Cannot Connect to Server

**Symptoms:**
- "Connection refused" errors
- Timeout errors
- Authentication failures

**Diagnostic Steps:**

1. **Verify Server Details**
   - Confirm hostname/IP is correct
   - Verify port number (default: 8443)
   - Check protocol (http vs https)

2. **Test Network Connectivity**
   ```bash
   # Ping server
   ping server.example.com
   
   # Test port connectivity
   telnet server.example.com 8443
   # Or using nc
   nc -zv server.example.com 8443
   
   # Check DNS resolution
   nslookup server.example.com
   ```

3. **Check Firewall**
   ```bash
   # Windows - Check firewall rules
   netsh advfirewall firewall show rule name=all
   
   # Linux - Check iptables/ufw
   sudo ufw status
   sudo iptables -L -n
   
   # macOS - Check firewall
   /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
   ```

4. **Verify SSL Certificate**
   ```bash
   # Test SSL connection
   openssl s_client -connect server.example.com:8443
   
   # Check certificate expiration
   echo | openssl s_client -connect server.example.com:8443 2>/dev/null | \
       openssl x509 -noout -dates
   ```

5. **Check Proxy Settings**
   - If behind corporate proxy, configure proxy in client settings
   - Verify proxy credentials are correct

**Solutions:**

1. **Update Server Address**
   - Edit configuration with correct server details
   - Save and restart client

2. **Add Firewall Exception**
   ```bash
   # Windows
   netsh advfirewall firewall add rule name="Revio 4 Client" ^
       dir=out action=allow protocol=TCP remoteport=8443
   
   # Linux (ufw)
   sudo ufw allow out 8443/tcp
   ```

3. **Install Custom CA Certificate**
   - For self-signed certificates
   - Add CA cert to client configuration
   - See [Client Configuration](client/configuration.md#custom-certificate-authority)

4. **Configure Proxy**
   ```json
   {
     "proxy": {
       "enabled": true,
       "host": "proxy.company.com",
       "port": 8080,
       "auth": {
         "username": "user",
         "password": "pass"
       }
     }
   }
   ```

### Slow Performance

**Symptoms:**
- Lag or delays in UI
- Slow data loading
- Timeouts

**Solutions:**

1. **Check Network Speed**
   ```bash
   # Test bandwidth to server
   speedtest-cli
   
   # Check latency
   ping -c 10 server.example.com
   ```

2. **Adjust Client Settings**
   ```json
   {
     "performance": {
       "cacheDuration": 7200,
       "maxCacheSize": 200,
       "compressionEnabled": true
     }
   }
   ```

3. **Clear Cache**
   - Settings > Clear Cache
   - Or delete cache directory and restart

4. **Update Client**
   - Install latest version for performance improvements

5. **Check System Resources**
   ```bash
   # Windows
   taskmgr
   
   # macOS
   Activity Monitor
   
   # Linux
   top
   htop
   ```

### Authentication Issues

**Symptoms:**
- Login failures
- "Invalid credentials" errors
- Session expires immediately

**Solutions:**

1. **Verify Credentials**
   - Confirm username and password are correct
   - Check for typos, caps lock

2. **Reset Password**
   - Use password reset functionality
   - Contact administrator if needed

3. **Check Time Synchronization**
   ```bash
   # JWT tokens are time-sensitive
   # Linux
   timedatectl status
   
   # macOS
   sudo sntp -sS time.apple.com
   
   # Windows
   w32tm /query /status
   ```

4. **Clear Saved Credentials**
   - Remove saved credentials
   - Re-enter manually

5. **Check Server Status**
   - Verify server is running
   - Check if authentication service is up

## Server Issues

### Server Won't Start

**Symptoms:**
- Service fails to start
- Immediate crash after starting
- Port binding errors

**Diagnostic Steps:**

1. **Check Service Status**
   ```bash
   # Linux
   sudo systemctl status revio4-server
   
   # Windows
   sc query Revio4Server
   ```

2. **Review Logs**
   ```bash
   # Linux systemd
   sudo journalctl -u revio4-server -n 100 --no-pager
   
   # Linux log file
   sudo tail -f /var/log/revio4/server.log
   
   # Windows
   # Check Event Viewer or C:\Program Files\Revio4\Server\logs\
   ```

3. **Validate Configuration**
   ```bash
   sudo revio4-server check-config
   ```

**Common Issues and Solutions:**

1. **Port Already in Use**
   ```bash
   # Find process using port
   sudo lsof -i :8443
   sudo netstat -tulpn | grep 8443
   
   # Kill conflicting process or change port in configuration
   ```

2. **Database Connection Failed**
   ```bash
   # Test database connectivity
   psql -h localhost -U revio4user -d revio4
   
   # Check database is running
   sudo systemctl status postgresql
   
   # Verify credentials in server configuration
   ```

3. **Permission Errors**
   ```bash
   # Fix file permissions
   sudo chown -R revio4:revio4 /var/lib/revio4
   sudo chown -R revio4:revio4 /etc/revio4
   sudo chmod 640 /etc/revio4/server.conf
   sudo chmod 600 /etc/revio4/ssl/*.key
   ```

4. **SSL Certificate Issues**
   ```bash
   # Verify certificate files exist
   ls -la /etc/revio4/ssl/
   
   # Check certificate validity
   openssl x509 -in /etc/revio4/ssl/server.crt -text -noout
   
   # Verify private key matches certificate
   openssl x509 -noout -modulus -in server.crt | openssl md5
   openssl rsa -noout -modulus -in server.key | openssl md5
   ```

5. **Missing Dependencies**
   ```bash
   # Check for missing libraries
   ldd /usr/bin/revio4-server
   
   # Install missing dependencies
   sudo apt install <missing-package>
   ```

### High CPU Usage

**Symptoms:**
- Server consuming excessive CPU
- Slow response times
- System becomes unresponsive

**Diagnostic Steps:**

1. **Identify Resource Usage**
   ```bash
   # Overall system
   top
   htop
   
   # Specific to Revio 4
   ps aux | grep revio4-server
   
   # CPU usage over time
   sar -u 5 10
   ```

2. **Check Active Connections**
   ```bash
   sudo revio4-server stats
   
   # Check database connections
   sudo -u postgres psql -c "SELECT count(*) FROM pg_stat_activity;"
   ```

3. **Review Slow Queries**
   ```sql
   -- PostgreSQL slow query log
   SELECT pid, now() - query_start AS duration, query
   FROM pg_stat_activity
   WHERE state = 'active' AND now() - query_start > interval '5 seconds';
   ```

**Solutions:**

1. **Adjust Worker Processes**
   ```ini
   [performance]
   worker_processes = 4  # Reduce if too high
   ```

2. **Enable Query Caching**
   ```ini
   [performance]
   query_cache_enabled = true
   query_cache_ttl = 300
   ```

3. **Optimize Database**
   ```bash
   # Analyze and vacuum
   sudo -u postgres vacuumdb -z revio4
   
   # Reindex
   sudo -u postgres reindexdb revio4
   ```

4. **Add Database Indexes**
   ```sql
   -- Identify missing indexes
   -- Add appropriate indexes based on query patterns
   ```

5. **Scale Horizontally**
   - Add more server nodes
   - Implement load balancing

### High Memory Usage

**Symptoms:**
- Server consuming excessive memory
- Out of memory errors
- System swapping

**Diagnostic Steps:**

1. **Check Memory Usage**
   ```bash
   free -h
   vmstat 1
   ps aux --sort=-%mem | head
   ```

2. **Review Cache Settings**
   ```bash
   # Check current cache size
   sudo revio4-server cache-stats
   ```

**Solutions:**

1. **Adjust Cache Size**
   ```ini
   [performance]
   cache_size = 256MB  # Reduce from default
   ```

2. **Optimize Database Connection Pool**
   ```ini
   [database]
   pool_size = 10  # Reduce if too high
   ```

3. **Add More RAM**
   - Upgrade server hardware
   - Or reduce load by scaling horizontally

4. **Enable Memory Limits** (Docker)
   ```bash
   docker run --memory="2g" revio4/server:4.0.0
   ```

### Database Issues

**Symptoms:**
- Database connection errors
- Query timeouts
- Data inconsistencies

**Diagnostic Steps:**

1. **Check Database Status**
   ```bash
   sudo systemctl status postgresql
   
   # Connect to database
   psql -h localhost -U revio4user -d revio4
   ```

2. **Check Database Size**
   ```sql
   SELECT pg_size_pretty(pg_database_size('revio4'));
   ```

3. **Review Active Queries**
   ```sql
   SELECT * FROM pg_stat_activity WHERE state = 'active';
   ```

**Solutions:**

1. **Restart Database**
   ```bash
   sudo systemctl restart postgresql
   ```

2. **Increase Connection Limits**
   ```ini
   # postgresql.conf
   max_connections = 200
   ```

3. **Optimize Queries**
   - Add indexes
   - Rewrite inefficient queries
   - Use query analysis tools

4. **Clean Up Database**
   ```sql
   -- Remove old data
   DELETE FROM logs WHERE created_at < NOW() - INTERVAL '90 days';
   
   -- Vacuum
   VACUUM FULL;
   ```

5. **Check Disk Space**
   ```bash
   df -h
   
   # Clean up if needed
   sudo apt clean
   sudo journalctl --vacuum-time=7d
   ```

## Network Issues

### Firewall Blocking Connections

**Symptoms:**
- Connection timeouts
- Clients can't reach server
- Intermittent connectivity

**Solutions:**

1. **Configure Firewall Rules**
   ```bash
   # Linux (ufw)
   sudo ufw allow 8443/tcp
   sudo ufw reload
   
   # Linux (firewalld)
   sudo firewall-cmd --permanent --add-port=8443/tcp
   sudo firewall-cmd --reload
   
   # Linux (iptables)
   sudo iptables -A INPUT -p tcp --dport 8443 -j ACCEPT
   sudo iptables-save
   ```

2. **Check Security Groups** (Cloud)
   - AWS: Update Security Group rules
   - Azure: Update Network Security Group
   - GCP: Update Firewall rules

3. **Verify Port Forwarding** (NAT)
   - Configure router to forward port 8443 to server

### SSL/TLS Certificate Problems

**Symptoms:**
- "Certificate not trusted" errors
- "Certificate expired" errors
- Connection refused with SSL

**Solutions:**

1. **Check Certificate Validity**
   ```bash
   openssl x509 -in /etc/revio4/ssl/server.crt -text -noout | grep -A2 Validity
   ```

2. **Renew Expired Certificate**
   ```bash
   # Let's Encrypt
   sudo certbot renew
   sudo systemctl restart revio4-server
   ```

3. **Install Intermediate Certificates**
   ```bash
   # Combine certificate with intermediate chain
   cat server.crt intermediate.crt > fullchain.crt
   ```

4. **Use Valid Certificate Authority**
   - Obtain certificate from trusted CA
   - Or distribute custom CA to clients

## Data Issues

### Data Corruption

**Symptoms:**
- Database errors
- Inconsistent data
- Application crashes when accessing data

**Solutions:**

1. **Check Database Integrity**
   ```sql
   -- PostgreSQL
   SELECT * FROM pg_catalog.pg_database WHERE datname = 'revio4';
   ```

2. **Restore from Backup**
   ```bash
   # Stop server
   sudo systemctl stop revio4-server
   
   # Restore database
   gunzip -c /var/backups/revio4/db_backup.sql.gz | psql -U revio4user revio4
   
   # Start server
   sudo systemctl start revio4-server
   ```

3. **Run Database Repair**
   ```bash
   sudo -u postgres reindexdb revio4
   ```

### Missing or Lost Data

**Solutions:**

1. **Check Backups**
   ```bash
   ls -lh /var/backups/revio4/
   ```

2. **Restore from Backup**
   - See backup restoration procedures in [Deployment Guide](server/deployment.md#restore-from-backup)

3. **Check Transaction Logs**
   - Review audit logs for data modifications
   - Reconstruct data if possible

## Upgrade Issues

### Upgrade Fails

**Solutions:**

1. **Check Prerequisites**
   - Verify system requirements for new version
   - Ensure backups are current

2. **Review Upgrade Logs**
   ```bash
   sudo journalctl -u revio4-server -n 200
   ```

3. **Rollback to Previous Version**
   ```bash
   # Linux (apt)
   sudo apt install revio4-server=<previous-version>
   
   # Linux (dnf)
   sudo dnf downgrade revio4-server-<previous-version>
   
   # Restore database backup if needed
   ```

4. **Manual Upgrade Steps**
   - Follow upgrade guide carefully
   - Apply database migrations manually if needed

## Getting Additional Help

If you can't resolve the issue:

1. **Gather Information**
   - Exact error messages
   - Log files
   - Configuration files (sanitized)
   - System information
   - Steps to reproduce

2. **Check Documentation**
   - Review relevant documentation sections
   - Search for known issues

3. **Contact Support**
   - Provide gathered information
   - Include version numbers
   - Describe troubleshooting steps already taken

4. **Community Resources**
   - Check community forums
   - Search for similar issues
   - Ask for help with detailed information

## Diagnostic Commands Reference

### System Information

```bash
# OS version
cat /etc/os-release

# Kernel version
uname -a

# CPU info
lscpu

# Memory info
free -h

# Disk info
df -h

# Network interfaces
ip addr show
```

### Revio 4 Specific

```bash
# Version
revio4-server --version

# Status
sudo systemctl status revio4-server

# Configuration check
sudo revio4-server check-config

# Database connection test
sudo revio4-server test-db

# View stats
sudo revio4-server stats

# Health check
curl -k https://localhost:8443/health
```

### Log Files

```bash
# Server logs
sudo tail -f /var/log/revio4/server.log

# System logs
sudo journalctl -u revio4-server -f

# Database logs (PostgreSQL)
sudo tail -f /var/log/postgresql/postgresql-14-main.log

# Web server logs (if applicable)
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## Prevention Best Practices

1. **Regular Backups**: Implement automated backup procedures
2. **Monitoring**: Set up proactive monitoring and alerting
3. **Updates**: Keep software and OS up to date
4. **Documentation**: Document your configuration and procedures
5. **Testing**: Test changes in staging before production
6. **Security**: Follow security best practices
7. **Capacity Planning**: Monitor growth and plan for scaling
8. **Training**: Ensure team members are trained on the system

## Emergency Procedures

### System Down - Critical

1. **Assess Impact**: Determine scope of outage
2. **Notify Stakeholders**: Alert relevant parties
3. **Check Basics**: Power, network, services running
4. **Review Recent Changes**: What changed recently?
5. **Check Logs**: Look for error messages
6. **Attempt Restart**: Restart affected services
7. **Failover**: Switch to backup systems if available
8. **Escalate**: Contact vendor support if needed

### Data Loss - Critical

1. **Stop All Services**: Prevent further data loss
2. **Assess Damage**: Determine what was lost
3. **Check Backups**: Identify most recent good backup
4. **Restore**: Follow backup restoration procedures
5. **Verify**: Confirm data integrity after restore
6. **Document**: Record incident for post-mortem
7. **Improve**: Update procedures to prevent recurrence
