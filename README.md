# kolibri Installation Guide

kolibri is a free and open-source offline learning. Kolibri provides offline-first learning platform

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 50GB for content
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default kolibri port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install kolibri
sudo dnf install -y kolibri

# Enable and start service
sudo systemctl enable --now kolibri

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
kolibri --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kolibri
sudo apt install -y kolibri

# Enable and start service
sudo systemctl enable --now kolibri

# Configure firewall
sudo ufw allow 8080

# Verify installation
kolibri --version
```

### Arch Linux

```bash
# Install kolibri
sudo pacman -S kolibri

# Enable and start service
sudo systemctl enable --now kolibri

# Verify installation
kolibri --version
```

### Alpine Linux

```bash
# Install kolibri
apk add --no-cache kolibri

# Enable and start service
rc-update add kolibri default
rc-service kolibri start

# Verify installation
kolibri --version
```

### openSUSE/SLES

```bash
# Install kolibri
sudo zypper install -y kolibri

# Enable and start service
sudo systemctl enable --now kolibri

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
kolibri --version
```

### macOS

```bash
# Using Homebrew
brew install kolibri

# Start service
brew services start kolibri

# Verify installation
kolibri --version
```

### FreeBSD

```bash
# Using pkg
pkg install kolibri

# Enable in rc.conf
echo 'kolibri_enable="YES"' >> /etc/rc.conf

# Start service
service kolibri start

# Verify installation
kolibri --version
```

### Windows

```bash
# Using Chocolatey
choco install kolibri

# Or using Scoop
scoop install kolibri

# Verify installation
kolibri --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/kolibri

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
kolibri --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kolibri

# Start service
sudo systemctl start kolibri

# Stop service
sudo systemctl stop kolibri

# Restart service
sudo systemctl restart kolibri

# Check status
sudo systemctl status kolibri

# View logs
sudo journalctl -u kolibri -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kolibri default

# Start service
rc-service kolibri start

# Stop service
rc-service kolibri stop

# Restart service
rc-service kolibri restart

# Check status
rc-service kolibri status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kolibri_enable="YES"' >> /etc/rc.conf

# Start service
service kolibri start

# Stop service
service kolibri stop

# Restart service
service kolibri restart

# Check status
service kolibri status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kolibri
brew services stop kolibri
brew services restart kolibri

# Check status
brew services list | grep kolibri
```

### Windows Service Manager

```powershell
# Start service
net start kolibri

# Stop service
net stop kolibri

# Using PowerShell
Start-Service kolibri
Stop-Service kolibri
Restart-Service kolibri

# Check status
Get-Service kolibri
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kolibri_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name kolibri.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kolibri.example.com;

    ssl_certificate /etc/ssl/certs/kolibri.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kolibri.example.com.key;

    location / {
        proxy_pass http://kolibri_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName kolibri.example.com
    Redirect permanent / https://kolibri.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kolibri.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kolibri.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kolibri.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kolibri_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kolibri.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kolibri_backend

backend kolibri_backend
    balance roundrobin
    server kolibri1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kolibri:kolibri /etc/kolibri
sudo chmod 750 /etc/kolibri

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status kolibri

# View logs
sudo journalctl -u kolibri -f

# Monitor resource usage
top -p $(pgrep kolibri)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/kolibri"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/kolibri-backup-$DATE.tar.gz" /etc/kolibri /var/lib/kolibri

echo "Backup completed: $BACKUP_DIR/kolibri-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kolibri

# Restore from backup
tar -xzf /backup/kolibri/kolibri-backup-*.tar.gz -C /

# Start service
sudo systemctl start kolibri
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kolibri -n 100
sudo tail -f /var/log/kolibri/kolibri.log

# Check configuration
kolibri --version

# Check permissions
ls -la /etc/kolibri
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kolibri)

# Check disk I/O
iotop -p $(pgrep kolibri)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  kolibri:
    image: kolibri:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/kolibri
      - ./data:/var/lib/kolibri
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kolibri

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kolibri

# Arch Linux
sudo pacman -Syu kolibri

# Alpine Linux
apk update && apk upgrade kolibri

# openSUSE
sudo zypper update kolibri

# FreeBSD
pkg update && pkg upgrade kolibri

# Always backup before updates
tar -czf /backup/kolibri-pre-update-$(date +%Y%m%d).tar.gz /etc/kolibri

# Restart after updates
sudo systemctl restart kolibri
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/kolibri

# Clean old logs
find /var/log/kolibri -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/kolibri
```

## Additional Resources

- Official Documentation: https://docs.kolibri.org/
- GitHub Repository: https://github.com/kolibri/kolibri
- Community Forum: https://forum.kolibri.org/
- Best Practices Guide: https://docs.kolibri.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
