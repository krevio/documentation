# Client Distribution Guide

This guide covers methods for distributing the Revio 4 client application to end users in various scenarios.

## Distribution Methods Overview

There are several ways to distribute the Revio 4 client:

1. **Direct Download** - Users download from a website or file share
2. **Package Managers** - Distribution through OS package repositories
3. **Enterprise Deployment** - Automated deployment using enterprise tools
4. **Application Stores** - Distribution through app stores (mobile)
5. **Physical Media** - Distribution via USB drives or optical media

## Direct Download Distribution

### Setting Up a Download Portal

1. **Host Installer Files**
   ```
   /downloads/
   ├── windows/
   │   └── Revio4-Client-Setup-4.0.0.exe
   ├── macos/
   │   └── Revio4-Client-4.0.0.dmg
   └── linux/
       ├── revio4-client_4.0.0_amd64.deb
       ├── revio4-client-4.0.0-1.x86_64.rpm
       └── Revio4-Client-4.0.0.AppImage
   ```

2. **Provide Checksums**
   
   Create checksum files for verification:
   
   ```bash
   # Generate SHA256 checksums
   sha256sum Revio4-Client-Setup-4.0.0.exe > checksums.txt
   sha256sum Revio4-Client-4.0.0.dmg >> checksums.txt
   ```

3. **Create Download Page**
   
   Provide clear instructions with:
   - System requirements
   - Download links for each platform
   - Checksum verification instructions
   - Installation guide links

### Checksum Verification

**Windows (PowerShell):**
```powershell
Get-FileHash Revio4-Client-Setup-4.0.0.exe -Algorithm SHA256
```

**macOS/Linux:**
```bash
sha256sum Revio4-Client-4.0.0.dmg
# Compare output with published checksum
```

## Package Manager Distribution

### APT Repository (Debian/Ubuntu)

1. **Set Up Repository**
   ```bash
   # Create repository structure
   mkdir -p /var/www/apt/{pool,dists/stable/main/binary-amd64}
   ```

2. **Add Package**
   ```bash
   cp revio4-client_4.0.0_amd64.deb /var/www/apt/pool/
   ```

3. **Generate Packages Index**
   ```bash
   cd /var/www/apt
   dpkg-scanpackages pool /dev/null | gzip -9c > dists/stable/main/binary-amd64/Packages.gz
   ```

4. **Sign Repository** (optional but recommended)
   ```bash
   gpg --gen-key
   apt-ftparchive release dists/stable > dists/stable/Release
   gpg --default-key <KEY_ID> -abs -o dists/stable/Release.gpg dists/stable/Release
   ```

5. **Client Configuration**
   
   Users add repository:
   ```bash
   echo "deb [trusted=yes] http://repo.example.com/apt stable main" | sudo tee /etc/apt/sources.list.d/revio4.list
   sudo apt update
   sudo apt install revio4-client
   ```

### YUM/DNF Repository (Red Hat/CentOS/Fedora)

1. **Create Repository**
   ```bash
   mkdir -p /var/www/yum/revio4
   cp revio4-client-4.0.0-1.x86_64.rpm /var/www/yum/revio4/
   ```

2. **Generate Metadata**
   ```bash
   createrepo /var/www/yum/revio4
   ```

3. **Sign Packages** (optional)
   ```bash
   rpm --addsign /var/www/yum/revio4/*.rpm
   ```

4. **Client Configuration**
   
   Create `/etc/yum.repos.d/revio4.repo`:
   ```ini
   [revio4]
   name=Revio 4 Repository
   baseurl=http://repo.example.com/yum/revio4
   enabled=1
   gpgcheck=0
   ```

   Install:
   ```bash
   sudo dnf install revio4-client
   ```

## Enterprise Deployment Methods

### Windows Group Policy Deployment

1. **Prepare MSI Package**
   - Convert EXE to MSI if necessary
   - Or use built-in MSI package

2. **Create GPO**
   - Open Group Policy Management
   - Create new GPO: "Deploy Revio 4 Client"

3. **Configure Software Installation**
   ```
   Computer Configuration > Policies > Software Settings > Software Installation
   - Right-click > New > Package
   - Browse to MSI on network share
   - Select "Assigned" deployment
   ```

4. **Apply GPO**
   - Link GPO to target OU
   - Wait for group policy refresh or force update:
   ```cmd
   gpupdate /force
   ```

### SCCM/ConfigMgr Deployment

1. **Import Application**
   - Create new application in SCCM
   - Specify installer details

2. **Configure Detection Method**
   ```
   Registry: HKLM\SOFTWARE\Revio4\Client
   Value: Version
   ```

3. **Create Deployment**
   - Distribute content to distribution points
   - Deploy to device collection
   - Set deployment schedule

4. **Monitor Deployment**
   - Check deployment status
   - Review reports

### Intune Deployment (Windows 10/11)

1. **Package Application**
   - Use Microsoft Win32 Content Prep Tool
   ```cmd
   IntuneWinAppUtil.exe -c source -s Revio4-Client-Setup.exe -o output
   ```

2. **Add to Intune**
   - Apps > Windows > Add > Windows app (Win32)
   - Upload .intunewin file

3. **Configure App**
   - Install command: `Revio4-Client-Setup.exe /S`
   - Uninstall command: `"C:\Program Files\Revio4\Client\uninstall.exe" /S`

4. **Set Requirements**
   - Operating system: Windows 10/11
   - Architecture: x64

5. **Assign to Groups**
   - Assign to user or device groups
   - Set deployment as required or available

### macOS MDM Deployment (Jamf, Intune)

1. **Create Package**
   - Use `pkgbuild` to create .pkg from .app:
   ```bash
   pkgbuild --component /Applications/Revio4\ Client.app \
            --install-location /Applications \
            Revio4-Client-4.0.0.pkg
   ```

2. **Upload to MDM**
   - Upload .pkg to Jamf Pro or Intune
   - Configure deployment settings

3. **Deploy**
   - Assign to computers or users
   - Set installation schedule

### Linux Configuration Management

#### Ansible Playbook

```yaml
---
- name: Install Revio 4 Client
  hosts: workstations
  become: yes
  tasks:
    - name: Add Revio 4 repository (Debian/Ubuntu)
      apt_repository:
        repo: "deb [trusted=yes] http://repo.example.com/apt stable main"
        state: present
      when: ansible_os_family == "Debian"

    - name: Install Revio 4 Client (Debian/Ubuntu)
      apt:
        name: revio4-client
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Add Revio 4 repository (RedHat/CentOS)
      yum_repository:
        name: revio4
        description: Revio 4 Repository
        baseurl: http://repo.example.com/yum/revio4
        gpgcheck: no
      when: ansible_os_family == "RedHat"

    - name: Install Revio 4 Client (RedHat/CentOS)
      dnf:
        name: revio4-client
        state: present
      when: ansible_os_family == "RedHat"
```

#### Puppet Manifest

```puppet
class revio4::client {
  case $::osfamily {
    'Debian': {
      apt::source { 'revio4':
        location => 'http://repo.example.com/apt',
        release  => 'stable',
        repos    => 'main',
      }

      package { 'revio4-client':
        ensure  => installed,
        require => Apt::Source['revio4'],
      }
    }
    'RedHat': {
      yumrepo { 'revio4':
        baseurl  => 'http://repo.example.com/yum/revio4',
        enabled  => 1,
        gpgcheck => 0,
      }

      package { 'revio4-client':
        ensure  => installed,
        require => Yumrepo['revio4'],
      }
    }
  }
}
```

## Pre-configured Client Distribution

### Customizing Installation Packages

1. **Create Configuration File**
   ```json
   {
     "server": {
       "host": "revio4.company.com",
       "port": 8443,
       "protocol": "https"
     },
     "client": {
       "autoConnect": true
     }
   }
   ```

2. **Include in Installation**
   - Windows: Include config.json in installer
   - macOS: Bundle config with .app
   - Linux: Include in package postinstall script

3. **Deployment Script Example**
   
   **Windows (PowerShell):**
   ```powershell
   # Install client
   Start-Process "Revio4-Client-Setup.exe" -ArgumentList "/S" -Wait
   
   # Deploy configuration
   $configPath = "$env:APPDATA\Revio4"
   New-Item -ItemType Directory -Force -Path $configPath
   Copy-Item "config.json" "$configPath\config.json"
   ```

   **Linux (Bash):**
   ```bash
   #!/bin/bash
   # Install client
   sudo dpkg -i revio4-client_4.0.0_amd64.deb
   
   # Deploy configuration
   mkdir -p ~/.config/revio4
   cp config.json ~/.config/revio4/config.json
   chmod 600 ~/.config/revio4/config.json
   ```

## Update Distribution

### Automatic Updates

Configure clients for automatic updates:

1. **Enable Auto-Update**
   ```json
   {
     "client": {
       "autoUpdate": true,
       "updateChannel": "stable"
     }
   }
   ```

2. **Update Server**
   - Host update manifests and packages
   - Client checks for updates periodically

### Manual Update Notification

Send notifications to users:

1. **Email Notification**
2. **In-App Notification**
3. **Administrator Portal**

### Controlled Rollout

For large deployments:

1. **Pilot Group** (10% of users)
2. **Early Adopters** (25% of users)
3. **General Availability** (All users)

Monitor each stage before proceeding.

## License Distribution

If using license-based deployment:

1. **License File Distribution**
   ```
   license.lic file deployment options:
   - Include in installation package
   - Deploy via configuration management
   - User imports manually
   ```

2. **License Server**
   - Central license server
   - Clients check out licenses
   - Floating license model

3. **Volume Licensing**
   - Single license key for organization
   - Configured during deployment

## Mobile Distribution

### iOS (App Store or Enterprise)

1. **App Store Distribution**
   - Submit to Apple App Store
   - Users install via App Store

2. **Enterprise Distribution**
   - Use Apple Developer Enterprise Program
   - Distribute via MDM or web page

### Android (Play Store or Enterprise)

1. **Google Play Store**
   - Publish to Play Store
   - Users install from store

2. **Enterprise Distribution**
   - Use managed Google Play
   - Or distribute APK directly (with caution)

## Network Installation

For environments with limited internet:

1. **Local Mirror**
   - Set up local repository/file share
   - Clients access from internal network

2. **Offline Installer**
   - Create package with all dependencies
   - Distribute via physical media or network share

## Distribution Best Practices

1. **Version Control**
   - Maintain version history
   - Track deployed versions
   - Plan rollback strategy

2. **Testing**
   - Test installations in lab environment
   - Verify on all target platforms
   - Test upgrade paths

3. **Documentation**
   - Provide clear installation instructions
   - Include troubleshooting steps
   - Document configuration options

4. **Support**
   - Establish support channels
   - Monitor installation success rates
   - Gather user feedback

5. **Security**
   - Sign all packages
   - Use secure distribution channels (HTTPS)
   - Verify integrity with checksums
   - Scan for malware before distribution

6. **Communication**
   - Notify users before deployment
   - Provide training if needed
   - Set expectations for timeline

## Monitoring Distribution

Track deployment metrics:

- Number of successful installations
- Failed installations and reasons
- Versions in use
- Active users
- Support tickets related to installation

## Next Steps

- [Configure the Client](configuration.md)
- [Review Installation Guide](installation.md)
- [Troubleshoot Issues](../troubleshooting.md)
