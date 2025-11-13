# Client Installation Guide

This guide provides step-by-step instructions for installing the Revio 4 client application on various operating systems.

## Prerequisites

Before installing the Revio 4 client:

1. Review [System Requirements](../system-requirements.md)
2. Ensure you have administrator/root privileges
3. Obtain the client installer package
4. Have your server connection details ready (hostname/IP, port)

## Windows Installation

### Installation Steps

1. **Download the Installer**
   - Download `Revio4-Client-Setup.exe` from the distribution source
   - Verify the installer checksum (optional but recommended)

2. **Run the Installer**
   ```
   - Double-click `Revio4-Client-Setup.exe`
   - Click "Yes" if prompted by User Account Control
   ```

3. **Follow the Installation Wizard**
   - Accept the license agreement
   - Choose installation directory (default: `C:\Program Files\Revio4\Client`)
   - Select components to install
   - Choose Start Menu folder
   - Create desktop shortcut (optional)

4. **Complete Installation**
   - Click "Install" to begin
   - Wait for installation to complete
   - Click "Finish" to exit the wizard

5. **Launch the Application**
   - Use the desktop shortcut or Start Menu item
   - Proceed to [Configuration](configuration.md)

### Silent Installation (Administrators)

For automated deployments:

```cmd
Revio4-Client-Setup.exe /S /D=C:\Program Files\Revio4\Client
```

Parameters:
- `/S` - Silent installation
- `/D=<path>` - Installation directory

## macOS Installation

### Installation Steps

1. **Download the Installer**
   - Download `Revio4-Client.dmg` from the distribution source

2. **Mount the Disk Image**
   - Double-click `Revio4-Client.dmg`
   - The disk image will mount automatically

3. **Install the Application**
   - Drag the Revio4 Client icon to the Applications folder
   - Wait for the copy to complete
   - Eject the disk image

4. **First Launch**
   - Open Applications folder
   - Double-click Revio4 Client
   - If prompted about unidentified developer:
     - Right-click the app and select "Open"
     - Click "Open" in the security dialog
   
5. **Proceed to Configuration**
   - See [Configuration Guide](configuration.md)

### Command Line Installation

Using Terminal:

```bash
# Mount the DMG
hdiutil attach Revio4-Client.dmg

# Copy to Applications
cp -R /Volumes/Revio4\ Client/Revio4\ Client.app /Applications/

# Unmount the DMG
hdiutil detach /Volumes/Revio4\ Client
```

## Linux Installation

### Debian/Ubuntu Installation (.deb)

1. **Download the Package**
   ```bash
   wget https://example.com/downloads/revio4-client_4.0.0_amd64.deb
   ```

2. **Install via APT**
   ```bash
   sudo apt update
   sudo apt install ./revio4-client_4.0.0_amd64.deb
   ```

   Or using dpkg:
   ```bash
   sudo dpkg -i revio4-client_4.0.0_amd64.deb
   sudo apt-get install -f  # Install dependencies if needed
   ```

3. **Launch the Application**
   ```bash
   revio4-client
   ```

### Red Hat/CentOS/Fedora Installation (.rpm)

1. **Download the Package**
   ```bash
   wget https://example.com/downloads/revio4-client-4.0.0-1.x86_64.rpm
   ```

2. **Install via DNF/YUM**
   ```bash
   sudo dnf install revio4-client-4.0.0-1.x86_64.rpm
   ```

   Or using rpm:
   ```bash
   sudo rpm -ivh revio4-client-4.0.0-1.x86_64.rpm
   ```

3. **Launch the Application**
   ```bash
   revio4-client
   ```

### Generic Linux Installation (AppImage)

1. **Download the AppImage**
   ```bash
   wget https://example.com/downloads/Revio4-Client-4.0.0.AppImage
   ```

2. **Make it Executable**
   ```bash
   chmod +x Revio4-Client-4.0.0.AppImage
   ```

3. **Run the Application**
   ```bash
   ./Revio4-Client-4.0.0.AppImage
   ```

4. **Optional: Integrate with Desktop**
   ```bash
   ./Revio4-Client-4.0.0.AppImage --appimage-extract
   sudo mv squashfs-root /opt/revio4-client
   # Create desktop entry manually or use provided script
   ```

## Post-Installation Steps

After installing the client:

1. **Initial Configuration**
   - Launch the Revio 4 client
   - Enter server connection details
   - See [Configuration Guide](configuration.md) for details

2. **Verify Installation**
   - Check that the application launches without errors
   - Verify version number: Help > About

3. **Test Connectivity**
   - Connect to your Revio 4 server
   - Verify authentication works
   - Test basic functionality

4. **Set Up User Preferences**
   - Configure application settings
   - Set up any local preferences
   - Enable auto-start if desired

## Updating the Client

### Windows Update

1. Download the new installer
2. Run the installer (it will detect existing installation)
3. Follow upgrade prompts

### macOS Update

1. Download new DMG
2. Replace existing app in Applications folder
3. Launch updated application

### Linux Update

Download and install new package version:

**Debian/Ubuntu:**
```bash
sudo apt update
sudo apt upgrade revio4-client
```

**Red Hat/Fedora:**
```bash
sudo dnf upgrade revio4-client
```

## Uninstalling the Client

### Windows Uninstall

1. **Via Control Panel:**
   - Open "Add or Remove Programs"
   - Find "Revio 4 Client"
   - Click "Uninstall"

2. **Via Command Line:**
   ```cmd
   "C:\Program Files\Revio4\Client\uninstall.exe" /S
   ```

### macOS Uninstall

```bash
# Remove application
sudo rm -rf /Applications/Revio4\ Client.app

# Remove user data (optional)
rm -rf ~/Library/Application\ Support/Revio4
rm -rf ~/Library/Preferences/com.revio.revio4-client.plist
```

### Linux Uninstall

**Debian/Ubuntu:**
```bash
sudo apt remove revio4-client
sudo apt purge revio4-client  # Also remove configuration
```

**Red Hat/Fedora:**
```bash
sudo dnf remove revio4-client
```

**AppImage:**
```bash
# Simply delete the AppImage file
rm Revio4-Client-4.0.0.AppImage
```

## Troubleshooting Installation Issues

### Installation Fails

- **Insufficient Permissions**: Run installer as administrator/root
- **Disk Space**: Ensure adequate disk space available
- **Antivirus**: Temporarily disable antivirus if blocking installation
- **Corrupted Download**: Verify installer checksum and re-download

### Application Won't Launch

- **Missing Dependencies**: Install required system libraries
- **Permission Issues**: Check file permissions
- **Port Conflicts**: Ensure no port conflicts with other applications

### Connection Issues After Installation

- **Firewall**: Configure firewall to allow Revio 4 client
- **Network**: Verify network connectivity to server
- **Server Details**: Confirm server hostname/IP and port are correct

For more troubleshooting help, see the [Troubleshooting Guide](../troubleshooting.md).

## Next Steps

- [Configure the Client](configuration.md)
- [Review System Requirements](../system-requirements.md)
- [Read Getting Started Guide](../getting-started.md)
