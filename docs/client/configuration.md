# Client Configuration Guide

This guide explains how to configure the Revio 4 client application after installation.

## Initial Configuration

When you first launch the Revio 4 client, you'll need to configure the connection to your server.

### Server Connection Setup

1. **Launch the Client**
   - Start the Revio 4 client application
   - You'll be presented with the configuration wizard

2. **Enter Server Details**
   ```
   Server Address: server.example.com
   Port: 8443
   Protocol: HTTPS (recommended)
   ```

3. **Test Connection**
   - Click "Test Connection" to verify settings
   - Ensure the server is reachable

4. **Save Configuration**
   - Click "Save" to store the settings
   - Proceed to authentication

## Configuration File Location

Configuration files are stored in different locations depending on your operating system:

### Windows
```
C:\Users\<username>\AppData\Roaming\Revio4\config.json
```

### macOS
```
~/Library/Application Support/Revio4/config.json
```

### Linux
```
~/.config/revio4/config.json
```

## Configuration File Format

The configuration file uses JSON format:

```json
{
  "server": {
    "host": "server.example.com",
    "port": 8443,
    "protocol": "https",
    "timeout": 30000,
    "retryAttempts": 3
  },
  "client": {
    "autoConnect": true,
    "saveCredentials": false,
    "logLevel": "info",
    "theme": "light",
    "language": "en"
  },
  "network": {
    "proxy": {
      "enabled": false,
      "host": "",
      "port": 0,
      "auth": {
        "username": "",
        "password": ""
      }
    },
    "ssl": {
      "verify": true,
      "customCertificate": ""
    }
  },
  "performance": {
    "cacheDuration": 3600,
    "maxCacheSize": 100,
    "compressionEnabled": true
  }
}
```

## Configuration Options

### Server Settings

#### host
- **Type**: String
- **Description**: Server hostname or IP address
- **Example**: `"server.example.com"` or `"192.168.1.100"`

#### port
- **Type**: Number
- **Default**: `8443`
- **Description**: Server port number

#### protocol
- **Type**: String
- **Options**: `"https"` (recommended), `"http"`
- **Default**: `"https"`
- **Description**: Communication protocol

#### timeout
- **Type**: Number (milliseconds)
- **Default**: `30000` (30 seconds)
- **Description**: Connection timeout duration

#### retryAttempts
- **Type**: Number
- **Default**: `3`
- **Description**: Number of connection retry attempts

### Client Settings

#### autoConnect
- **Type**: Boolean
- **Default**: `true`
- **Description**: Automatically connect on startup

#### saveCredentials
- **Type**: Boolean
- **Default**: `false`
- **Description**: Save login credentials (encrypted)
- **Security**: Enable only on trusted devices

#### logLevel
- **Type**: String
- **Options**: `"debug"`, `"info"`, `"warning"`, `"error"`
- **Default**: `"info"`
- **Description**: Logging verbosity level

#### theme
- **Type**: String
- **Options**: `"light"`, `"dark"`, `"system"`
- **Default**: `"light"`
- **Description**: User interface theme

#### language
- **Type**: String
- **Options**: ISO language codes (`"en"`, `"es"`, `"fr"`, `"de"`, etc.)
- **Default**: `"en"`
- **Description**: Interface language

### Network Settings

#### Proxy Configuration

To use a proxy server:

```json
"proxy": {
  "enabled": true,
  "host": "proxy.example.com",
  "port": 8080,
  "auth": {
    "username": "proxyuser",
    "password": "proxypass"
  }
}
```

**Note**: Passwords in config files should use environment variables or secure storage in production.

#### SSL/TLS Configuration

##### verify
- **Type**: Boolean
- **Default**: `true`
- **Description**: Verify SSL certificates
- **Security**: Only disable for testing/development

##### customCertificate
- **Type**: String (path)
- **Description**: Path to custom CA certificate
- **Example**: `"/path/to/custom-ca.pem"`

### Performance Settings

#### cacheDuration
- **Type**: Number (seconds)
- **Default**: `3600` (1 hour)
- **Description**: How long to cache data

#### maxCacheSize
- **Type**: Number (MB)
- **Default**: `100`
- **Description**: Maximum cache size

#### compressionEnabled
- **Type**: Boolean
- **Default**: `true`
- **Description**: Enable data compression

## Advanced Configuration

### Environment Variables

You can override configuration using environment variables:

```bash
# Server connection
export REVIO4_SERVER_HOST=server.example.com
export REVIO4_SERVER_PORT=8443
export REVIO4_SERVER_PROTOCOL=https

# Client settings
export REVIO4_LOG_LEVEL=debug
export REVIO4_AUTO_CONNECT=true
```

### Command Line Arguments

Launch the client with specific options:

```bash
# Windows
Revio4Client.exe --server=server.example.com --port=8443 --no-verify-ssl

# macOS/Linux
revio4-client --server=server.example.com --port=8443 --no-verify-ssl
```

Available arguments:
- `--server=<host>` - Server hostname/IP
- `--port=<port>` - Server port
- `--protocol=<http|https>` - Connection protocol
- `--no-verify-ssl` - Disable SSL verification (insecure)
- `--log-level=<level>` - Set log level
- `--config=<path>` - Use custom config file

### Custom Certificate Authority

For self-signed or internal certificates:

1. **Obtain the CA Certificate**
   - Get the `.pem` or `.crt` file from your administrator

2. **Configure Client**
   ```json
   "ssl": {
     "verify": true,
     "customCertificate": "/path/to/ca-cert.pem"
   }
   ```

3. **Restart Client**
   - The custom CA will be used for verification

## Multi-Server Configuration

To manage multiple server profiles:

1. **Create Profile Files**
   ```
   config-production.json
   config-staging.json
   config-development.json
   ```

2. **Switch Profiles**
   ```bash
   revio4-client --config=config-production.json
   ```

3. **Profile Management** (via UI)
   - Settings > Server Profiles
   - Add, edit, or delete profiles
   - Switch between profiles

## Security Best Practices

1. **Use HTTPS**: Always use encrypted connections in production
2. **Verify Certificates**: Keep SSL verification enabled
3. **Secure Credentials**: Don't store passwords in config files
4. **File Permissions**: Restrict config file access
   ```bash
   # Linux/macOS
   chmod 600 ~/.config/revio4/config.json
   ```
5. **Regular Updates**: Keep client software updated
6. **Audit Logs**: Review logs for suspicious activity

## Configuration Backup

### Backup Configuration

**Windows:**
```cmd
copy "%APPDATA%\Revio4\config.json" "backup\config.json"
```

**macOS/Linux:**
```bash
cp ~/.config/revio4/config.json ~/backup/config.json
```

### Restore Configuration

Replace the config file with your backup and restart the client.

## Troubleshooting Configuration Issues

### Client Won't Connect

1. **Verify server details** are correct
2. **Check network connectivity**
3. **Test with ping/telnet**:
   ```bash
   ping server.example.com
   telnet server.example.com 8443
   ```
4. **Review firewall rules**
5. **Check SSL certificate** validity

### Configuration Not Saving

1. **Check file permissions**
2. **Verify disk space**
3. **Run as administrator** (Windows)
4. **Check for file locks**

### Performance Issues

1. **Adjust cache settings**
2. **Reduce log level** to `warning` or `error`
3. **Disable compression** if CPU-limited
4. **Increase timeout** for slow networks

### Invalid Configuration

If configuration becomes corrupted:

1. **Rename or delete** the config file
2. **Restart the client**
3. **Reconfigure** from scratch

The client will create a new default configuration.

## Logging and Diagnostics

### Log File Locations

**Windows:**
```
C:\Users\<username>\AppData\Local\Revio4\logs\
```

**macOS:**
```
~/Library/Logs/Revio4/
```

**Linux:**
```
~/.local/share/revio4/logs/
```

### Enable Debug Logging

```json
{
  "client": {
    "logLevel": "debug"
  }
}
```

Or via command line:
```bash
revio4-client --log-level=debug
```

### View Logs

**Via UI**: Help > View Logs

**Via Command Line**:
```bash
# Linux/macOS
tail -f ~/.local/share/revio4/logs/client.log

# Windows
type "%LOCALAPPDATA%\Revio4\logs\client.log"
```

## Next Steps

- [Review Distribution Methods](distribution.md)
- [Troubleshoot Issues](../troubleshooting.md)
- [Update Client](installation.md#updating-the-client)
