# argo-cd Installation Guide

argo-cd is a free and open-source declarative GitOps CD for Kubernetes. Argo CD automates deployment of applications to Kubernetes clusters using Git repositories as the source of truth

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: Git and Kubernetes access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default argo-cd port)
  - Port 8083 for metrics
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

# Install argo-cd
sudo dnf install -y argocd

# Enable and start service
sudo systemctl enable --now argocd-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
argocd version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install argo-cd
sudo apt install -y argocd

# Enable and start service
sudo systemctl enable --now argocd-server

# Configure firewall
sudo ufw allow 8080

# Verify installation
argocd version
```

### Arch Linux

```bash
# Install argo-cd
sudo pacman -S argocd

# Enable and start service
sudo systemctl enable --now argocd-server

# Verify installation
argocd version
```

### Alpine Linux

```bash
# Install argo-cd
apk add --no-cache argocd

# Enable and start service
rc-update add argocd-server default
rc-service argocd-server start

# Verify installation
argocd version
```

### openSUSE/SLES

```bash
# Install argo-cd
sudo zypper install -y argocd

# Enable and start service
sudo systemctl enable --now argocd-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
argocd version
```

### macOS

```bash
# Using Homebrew
brew install argocd

# Start service
brew services start argocd

# Verify installation
argocd version
```

### FreeBSD

```bash
# Using pkg
pkg install argocd

# Enable in rc.conf
echo 'argocd-server_enable="YES"' >> /etc/rc.conf

# Start service
service argocd-server start

# Verify installation
argocd version
```

### Windows

```bash
# Using Chocolatey
choco install argocd

# Or using Scoop
scoop install argocd

# Verify installation
argocd version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/argocd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
argocd version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable argocd-server

# Start service
sudo systemctl start argocd-server

# Stop service
sudo systemctl stop argocd-server

# Restart service
sudo systemctl restart argocd-server

# Check status
sudo systemctl status argocd-server

# View logs
sudo journalctl -u argocd-server -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add argocd-server default

# Start service
rc-service argocd-server start

# Stop service
rc-service argocd-server stop

# Restart service
rc-service argocd-server restart

# Check status
rc-service argocd-server status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'argocd-server_enable="YES"' >> /etc/rc.conf

# Start service
service argocd-server start

# Stop service
service argocd-server stop

# Restart service
service argocd-server restart

# Check status
service argocd-server status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start argocd
brew services stop argocd
brew services restart argocd

# Check status
brew services list | grep argocd
```

### Windows Service Manager

```powershell
# Start service
net start argocd-server

# Stop service
net stop argocd-server

# Using PowerShell
Start-Service argocd-server
Stop-Service argocd-server
Restart-Service argocd-server

# Check status
Get-Service argocd-server
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream argocd_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name argocd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name argocd.example.com;

    ssl_certificate /etc/ssl/certs/argocd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/argocd.example.com.key;

    location / {
        proxy_pass http://argocd_backend;
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
    ServerName argocd.example.com
    Redirect permanent / https://argocd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName argocd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/argocd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/argocd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend argocd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/argocd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend argocd_backend

backend argocd_backend
    balance roundrobin
    server argocd1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R argocd:argocd /etc/argocd
sudo chmod 750 /etc/argocd

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
sudo systemctl status argocd-server

# View logs
sudo journalctl -u argocd-server -f

# Monitor resource usage
top -p $(pgrep argocd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/argocd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/argocd-backup-$DATE.tar.gz" /etc/argocd /var/lib/argocd

echo "Backup completed: $BACKUP_DIR/argocd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop argocd-server

# Restore from backup
tar -xzf /backup/argocd/argocd-backup-*.tar.gz -C /

# Start service
sudo systemctl start argocd-server
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u argocd-server -n 100
sudo tail -f /var/log/argocd/argocd.log

# Check configuration
argocd version

# Check permissions
ls -la /etc/argocd
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
top -p $(pgrep argocd)

# Check disk I/O
iotop -p $(pgrep argocd)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  argocd:
    image: argocd:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/argocd
      - ./data:/var/lib/argocd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update argocd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade argocd

# Arch Linux
sudo pacman -Syu argocd

# Alpine Linux
apk update && apk upgrade argocd

# openSUSE
sudo zypper update argocd

# FreeBSD
pkg update && pkg upgrade argocd

# Always backup before updates
tar -czf /backup/argocd-pre-update-$(date +%Y%m%d).tar.gz /etc/argocd

# Restart after updates
sudo systemctl restart argocd-server
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/argocd

# Clean old logs
find /var/log/argocd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/argocd
```

## Additional Resources

- Official Documentation: https://docs.argocd.org/
- GitHub Repository: https://github.com/argocd/argocd
- Community Forum: https://forum.argocd.org/
- Best Practices Guide: https://docs.argocd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
