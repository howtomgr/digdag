# digdag Installation Guide

digdag is a free and open-source workflow engine. Digdag provides simple workflow engine for multi-cloud

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
  - RAM: 2GB minimum
  - Storage: 5GB for logs
  - Network: HTTP/REST
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 65432 (default digdag port)
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

# Install digdag
sudo dnf install -y digdag

# Enable and start service
sudo systemctl enable --now digdag

# Configure firewall
sudo firewall-cmd --permanent --add-port=65432/tcp
sudo firewall-cmd --reload

# Verify installation
digdag --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install digdag
sudo apt install -y digdag

# Enable and start service
sudo systemctl enable --now digdag

# Configure firewall
sudo ufw allow 65432

# Verify installation
digdag --version
```

### Arch Linux

```bash
# Install digdag
sudo pacman -S digdag

# Enable and start service
sudo systemctl enable --now digdag

# Verify installation
digdag --version
```

### Alpine Linux

```bash
# Install digdag
apk add --no-cache digdag

# Enable and start service
rc-update add digdag default
rc-service digdag start

# Verify installation
digdag --version
```

### openSUSE/SLES

```bash
# Install digdag
sudo zypper install -y digdag

# Enable and start service
sudo systemctl enable --now digdag

# Configure firewall
sudo firewall-cmd --permanent --add-port=65432/tcp
sudo firewall-cmd --reload

# Verify installation
digdag --version
```

### macOS

```bash
# Using Homebrew
brew install digdag

# Start service
brew services start digdag

# Verify installation
digdag --version
```

### FreeBSD

```bash
# Using pkg
pkg install digdag

# Enable in rc.conf
echo 'digdag_enable="YES"' >> /etc/rc.conf

# Start service
service digdag start

# Verify installation
digdag --version
```

### Windows

```bash
# Using Chocolatey
choco install digdag

# Or using Scoop
scoop install digdag

# Verify installation
digdag --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/digdag

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
digdag --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable digdag

# Start service
sudo systemctl start digdag

# Stop service
sudo systemctl stop digdag

# Restart service
sudo systemctl restart digdag

# Check status
sudo systemctl status digdag

# View logs
sudo journalctl -u digdag -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add digdag default

# Start service
rc-service digdag start

# Stop service
rc-service digdag stop

# Restart service
rc-service digdag restart

# Check status
rc-service digdag status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'digdag_enable="YES"' >> /etc/rc.conf

# Start service
service digdag start

# Stop service
service digdag stop

# Restart service
service digdag restart

# Check status
service digdag status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start digdag
brew services stop digdag
brew services restart digdag

# Check status
brew services list | grep digdag
```

### Windows Service Manager

```powershell
# Start service
net start digdag

# Stop service
net stop digdag

# Using PowerShell
Start-Service digdag
Stop-Service digdag
Restart-Service digdag

# Check status
Get-Service digdag
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream digdag_backend {
    server 127.0.0.1:65432;
}

server {
    listen 80;
    server_name digdag.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name digdag.example.com;

    ssl_certificate /etc/ssl/certs/digdag.example.com.crt;
    ssl_certificate_key /etc/ssl/private/digdag.example.com.key;

    location / {
        proxy_pass http://digdag_backend;
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
    ServerName digdag.example.com
    Redirect permanent / https://digdag.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName digdag.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/digdag.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/digdag.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:65432/
    ProxyPassReverse / http://127.0.0.1:65432/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend digdag_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/digdag.pem
    redirect scheme https if !{ ssl_fc }
    default_backend digdag_backend

backend digdag_backend
    balance roundrobin
    server digdag1 127.0.0.1:65432 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R digdag:digdag /etc/digdag
sudo chmod 750 /etc/digdag

# Configure firewall
sudo firewall-cmd --permanent --add-port=65432/tcp
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
sudo systemctl status digdag

# View logs
sudo journalctl -u digdag -f

# Monitor resource usage
top -p $(pgrep digdag)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/digdag"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/digdag-backup-$DATE.tar.gz" /etc/digdag /var/lib/digdag

echo "Backup completed: $BACKUP_DIR/digdag-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop digdag

# Restore from backup
tar -xzf /backup/digdag/digdag-backup-*.tar.gz -C /

# Start service
sudo systemctl start digdag
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u digdag -n 100
sudo tail -f /var/log/digdag/digdag.log

# Check configuration
digdag --version

# Check permissions
ls -la /etc/digdag
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 65432

# Test connectivity
telnet localhost 65432

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep digdag)

# Check disk I/O
iotop -p $(pgrep digdag)

# Check connections
ss -an | grep 65432
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  digdag:
    image: digdag:latest
    ports:
      - "65432:65432"
    volumes:
      - ./config:/etc/digdag
      - ./data:/var/lib/digdag
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update digdag

# Debian/Ubuntu
sudo apt update && sudo apt upgrade digdag

# Arch Linux
sudo pacman -Syu digdag

# Alpine Linux
apk update && apk upgrade digdag

# openSUSE
sudo zypper update digdag

# FreeBSD
pkg update && pkg upgrade digdag

# Always backup before updates
tar -czf /backup/digdag-pre-update-$(date +%Y%m%d).tar.gz /etc/digdag

# Restart after updates
sudo systemctl restart digdag
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/digdag

# Clean old logs
find /var/log/digdag -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/digdag
```

## Additional Resources

- Official Documentation: https://docs.digdag.org/
- GitHub Repository: https://github.com/digdag/digdag
- Community Forum: https://forum.digdag.org/
- Best Practices Guide: https://docs.digdag.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
