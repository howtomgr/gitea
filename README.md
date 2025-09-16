# gitea Installation Guide

gitea is a free and open-source lightweight self-hosted Git service. Written in Go, Gitea provides a lightweight alternative to GitHub, GitLab, or Bitbucket for self-hosted Git repository management

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
  - RAM: 256MB minimum (1GB+ recommended)
  - Storage: 200MB + repository data
  - Network: Git protocol and HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default gitea port)
  - Port 22 for Git SSH
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

# Install gitea
sudo dnf install -y gitea

# Enable and start service
sudo systemctl enable --now gitea

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
gitea --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gitea
sudo apt install -y gitea

# Enable and start service
sudo systemctl enable --now gitea

# Configure firewall
sudo ufw allow 3000

# Verify installation
gitea --version
```

### Arch Linux

```bash
# Install gitea
sudo pacman -S gitea

# Enable and start service
sudo systemctl enable --now gitea

# Verify installation
gitea --version
```

### Alpine Linux

```bash
# Install gitea
apk add --no-cache gitea

# Enable and start service
rc-update add gitea default
rc-service gitea start

# Verify installation
gitea --version
```

### openSUSE/SLES

```bash
# Install gitea
sudo zypper install -y gitea

# Enable and start service
sudo systemctl enable --now gitea

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
gitea --version
```

### macOS

```bash
# Using Homebrew
brew install gitea

# Start service
brew services start gitea

# Verify installation
gitea --version
```

### FreeBSD

```bash
# Using pkg
pkg install gitea

# Enable in rc.conf
echo 'gitea_enable="YES"' >> /etc/rc.conf

# Start service
service gitea start

# Verify installation
gitea --version
```

### Windows

```bash
# Using Chocolatey
choco install gitea

# Or using Scoop
scoop install gitea

# Verify installation
gitea --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gitea

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gitea --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gitea

# Start service
sudo systemctl start gitea

# Stop service
sudo systemctl stop gitea

# Restart service
sudo systemctl restart gitea

# Check status
sudo systemctl status gitea

# View logs
sudo journalctl -u gitea -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gitea default

# Start service
rc-service gitea start

# Stop service
rc-service gitea stop

# Restart service
rc-service gitea restart

# Check status
rc-service gitea status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gitea_enable="YES"' >> /etc/rc.conf

# Start service
service gitea start

# Stop service
service gitea stop

# Restart service
service gitea restart

# Check status
service gitea status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gitea
brew services stop gitea
brew services restart gitea

# Check status
brew services list | grep gitea
```

### Windows Service Manager

```powershell
# Start service
net start gitea

# Stop service
net stop gitea

# Using PowerShell
Start-Service gitea
Stop-Service gitea
Restart-Service gitea

# Check status
Get-Service gitea
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gitea_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name gitea.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gitea.example.com;

    ssl_certificate /etc/ssl/certs/gitea.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gitea.example.com.key;

    location / {
        proxy_pass http://gitea_backend;
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
    ServerName gitea.example.com
    Redirect permanent / https://gitea.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gitea.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gitea.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gitea.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gitea_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gitea.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gitea_backend

backend gitea_backend
    balance roundrobin
    server gitea1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gitea:gitea /etc/gitea
sudo chmod 750 /etc/gitea

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
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
sudo systemctl status gitea

# View logs
sudo journalctl -u gitea -f

# Monitor resource usage
top -p $(pgrep gitea)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gitea"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gitea-backup-$DATE.tar.gz" /etc/gitea /var/lib/gitea

echo "Backup completed: $BACKUP_DIR/gitea-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gitea

# Restore from backup
tar -xzf /backup/gitea/gitea-backup-*.tar.gz -C /

# Start service
sudo systemctl start gitea
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gitea -n 100
sudo tail -f /var/log/gitea/gitea.log

# Check configuration
gitea --version

# Check permissions
ls -la /etc/gitea
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gitea)

# Check disk I/O
iotop -p $(pgrep gitea)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gitea:
    image: gitea:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/gitea
      - ./data:/var/lib/gitea
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gitea

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gitea

# Arch Linux
sudo pacman -Syu gitea

# Alpine Linux
apk update && apk upgrade gitea

# openSUSE
sudo zypper update gitea

# FreeBSD
pkg update && pkg upgrade gitea

# Always backup before updates
tar -czf /backup/gitea-pre-update-$(date +%Y%m%d).tar.gz /etc/gitea

# Restart after updates
sudo systemctl restart gitea
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gitea

# Clean old logs
find /var/log/gitea -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gitea
```

## Additional Resources

- Official Documentation: https://docs.gitea.org/
- GitHub Repository: https://github.com/gitea/gitea
- Community Forum: https://forum.gitea.org/
- Best Practices Guide: https://docs.gitea.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
