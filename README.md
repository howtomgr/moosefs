# moosefs Installation Guide

moosefs is a free and open-source distributed file system. MooseFS provides fault-tolerant distributed file system

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 100GB+ for chunks
  - Network: Native protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9420 (default moosefs port)
  - Master on 9419
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

# Install moosefs
sudo dnf install -y moosefs

# Enable and start service
sudo systemctl enable --now moosefs

# Configure firewall
sudo firewall-cmd --permanent --add-port=9420/tcp
sudo firewall-cmd --reload

# Verify installation
moosefs --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install moosefs
sudo apt install -y moosefs

# Enable and start service
sudo systemctl enable --now moosefs

# Configure firewall
sudo ufw allow 9420

# Verify installation
moosefs --version
```

### Arch Linux

```bash
# Install moosefs
sudo pacman -S moosefs

# Enable and start service
sudo systemctl enable --now moosefs

# Verify installation
moosefs --version
```

### Alpine Linux

```bash
# Install moosefs
apk add --no-cache moosefs

# Enable and start service
rc-update add moosefs default
rc-service moosefs start

# Verify installation
moosefs --version
```

### openSUSE/SLES

```bash
# Install moosefs
sudo zypper install -y moosefs

# Enable and start service
sudo systemctl enable --now moosefs

# Configure firewall
sudo firewall-cmd --permanent --add-port=9420/tcp
sudo firewall-cmd --reload

# Verify installation
moosefs --version
```

### macOS

```bash
# Using Homebrew
brew install moosefs

# Start service
brew services start moosefs

# Verify installation
moosefs --version
```

### FreeBSD

```bash
# Using pkg
pkg install moosefs

# Enable in rc.conf
echo 'moosefs_enable="YES"' >> /etc/rc.conf

# Start service
service moosefs start

# Verify installation
moosefs --version
```

### Windows

```bash
# Using Chocolatey
choco install moosefs

# Or using Scoop
scoop install moosefs

# Verify installation
moosefs --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/moosefs

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
moosefs --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable moosefs

# Start service
sudo systemctl start moosefs

# Stop service
sudo systemctl stop moosefs

# Restart service
sudo systemctl restart moosefs

# Check status
sudo systemctl status moosefs

# View logs
sudo journalctl -u moosefs -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add moosefs default

# Start service
rc-service moosefs start

# Stop service
rc-service moosefs stop

# Restart service
rc-service moosefs restart

# Check status
rc-service moosefs status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'moosefs_enable="YES"' >> /etc/rc.conf

# Start service
service moosefs start

# Stop service
service moosefs stop

# Restart service
service moosefs restart

# Check status
service moosefs status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start moosefs
brew services stop moosefs
brew services restart moosefs

# Check status
brew services list | grep moosefs
```

### Windows Service Manager

```powershell
# Start service
net start moosefs

# Stop service
net stop moosefs

# Using PowerShell
Start-Service moosefs
Stop-Service moosefs
Restart-Service moosefs

# Check status
Get-Service moosefs
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream moosefs_backend {
    server 127.0.0.1:9420;
}

server {
    listen 80;
    server_name moosefs.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name moosefs.example.com;

    ssl_certificate /etc/ssl/certs/moosefs.example.com.crt;
    ssl_certificate_key /etc/ssl/private/moosefs.example.com.key;

    location / {
        proxy_pass http://moosefs_backend;
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
    ServerName moosefs.example.com
    Redirect permanent / https://moosefs.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName moosefs.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/moosefs.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/moosefs.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9420/
    ProxyPassReverse / http://127.0.0.1:9420/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend moosefs_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/moosefs.pem
    redirect scheme https if !{ ssl_fc }
    default_backend moosefs_backend

backend moosefs_backend
    balance roundrobin
    server moosefs1 127.0.0.1:9420 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R moosefs:moosefs /etc/moosefs
sudo chmod 750 /etc/moosefs

# Configure firewall
sudo firewall-cmd --permanent --add-port=9420/tcp
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
sudo systemctl status moosefs

# View logs
sudo journalctl -u moosefs -f

# Monitor resource usage
top -p $(pgrep moosefs)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/moosefs"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/moosefs-backup-$DATE.tar.gz" /etc/moosefs /var/lib/moosefs

echo "Backup completed: $BACKUP_DIR/moosefs-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop moosefs

# Restore from backup
tar -xzf /backup/moosefs/moosefs-backup-*.tar.gz -C /

# Start service
sudo systemctl start moosefs
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u moosefs -n 100
sudo tail -f /var/log/moosefs/moosefs.log

# Check configuration
moosefs --version

# Check permissions
ls -la /etc/moosefs
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9420

# Test connectivity
telnet localhost 9420

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep moosefs)

# Check disk I/O
iotop -p $(pgrep moosefs)

# Check connections
ss -an | grep 9420
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  moosefs:
    image: moosefs:latest
    ports:
      - "9420:9420"
    volumes:
      - ./config:/etc/moosefs
      - ./data:/var/lib/moosefs
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update moosefs

# Debian/Ubuntu
sudo apt update && sudo apt upgrade moosefs

# Arch Linux
sudo pacman -Syu moosefs

# Alpine Linux
apk update && apk upgrade moosefs

# openSUSE
sudo zypper update moosefs

# FreeBSD
pkg update && pkg upgrade moosefs

# Always backup before updates
tar -czf /backup/moosefs-pre-update-$(date +%Y%m%d).tar.gz /etc/moosefs

# Restart after updates
sudo systemctl restart moosefs
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/moosefs

# Clean old logs
find /var/log/moosefs -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/moosefs
```

## Additional Resources

- Official Documentation: https://docs.moosefs.org/
- GitHub Repository: https://github.com/moosefs/moosefs
- Community Forum: https://forum.moosefs.org/
- Best Practices Guide: https://docs.moosefs.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
