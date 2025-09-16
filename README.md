# osquery Installation Guide

osquery is a free and open-source SQL-based OS instrumentation. osquery exposes OS as relational database for monitoring

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
  - RAM: 512MB minimum
  - Storage: 5GB for logs
  - Network: Thrift API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9090 (default osquery port)
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

# Install osquery
sudo dnf install -y osquery

# Enable and start service
sudo systemctl enable --now osquery

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload

# Verify installation
osquery --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install osquery
sudo apt install -y osquery

# Enable and start service
sudo systemctl enable --now osquery

# Configure firewall
sudo ufw allow 9090

# Verify installation
osquery --version
```

### Arch Linux

```bash
# Install osquery
sudo pacman -S osquery

# Enable and start service
sudo systemctl enable --now osquery

# Verify installation
osquery --version
```

### Alpine Linux

```bash
# Install osquery
apk add --no-cache osquery

# Enable and start service
rc-update add osquery default
rc-service osquery start

# Verify installation
osquery --version
```

### openSUSE/SLES

```bash
# Install osquery
sudo zypper install -y osquery

# Enable and start service
sudo systemctl enable --now osquery

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload

# Verify installation
osquery --version
```

### macOS

```bash
# Using Homebrew
brew install osquery

# Start service
brew services start osquery

# Verify installation
osquery --version
```

### FreeBSD

```bash
# Using pkg
pkg install osquery

# Enable in rc.conf
echo 'osquery_enable="YES"' >> /etc/rc.conf

# Start service
service osquery start

# Verify installation
osquery --version
```

### Windows

```bash
# Using Chocolatey
choco install osquery

# Or using Scoop
scoop install osquery

# Verify installation
osquery --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/osquery

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
osquery --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable osquery

# Start service
sudo systemctl start osquery

# Stop service
sudo systemctl stop osquery

# Restart service
sudo systemctl restart osquery

# Check status
sudo systemctl status osquery

# View logs
sudo journalctl -u osquery -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add osquery default

# Start service
rc-service osquery start

# Stop service
rc-service osquery stop

# Restart service
rc-service osquery restart

# Check status
rc-service osquery status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'osquery_enable="YES"' >> /etc/rc.conf

# Start service
service osquery start

# Stop service
service osquery stop

# Restart service
service osquery restart

# Check status
service osquery status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start osquery
brew services stop osquery
brew services restart osquery

# Check status
brew services list | grep osquery
```

### Windows Service Manager

```powershell
# Start service
net start osquery

# Stop service
net stop osquery

# Using PowerShell
Start-Service osquery
Stop-Service osquery
Restart-Service osquery

# Check status
Get-Service osquery
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream osquery_backend {
    server 127.0.0.1:9090;
}

server {
    listen 80;
    server_name osquery.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name osquery.example.com;

    ssl_certificate /etc/ssl/certs/osquery.example.com.crt;
    ssl_certificate_key /etc/ssl/private/osquery.example.com.key;

    location / {
        proxy_pass http://osquery_backend;
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
    ServerName osquery.example.com
    Redirect permanent / https://osquery.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName osquery.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/osquery.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/osquery.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9090/
    ProxyPassReverse / http://127.0.0.1:9090/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend osquery_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/osquery.pem
    redirect scheme https if !{ ssl_fc }
    default_backend osquery_backend

backend osquery_backend
    balance roundrobin
    server osquery1 127.0.0.1:9090 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R osquery:osquery /etc/osquery
sudo chmod 750 /etc/osquery

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
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
sudo systemctl status osquery

# View logs
sudo journalctl -u osquery -f

# Monitor resource usage
top -p $(pgrep osquery)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/osquery"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/osquery-backup-$DATE.tar.gz" /etc/osquery /var/lib/osquery

echo "Backup completed: $BACKUP_DIR/osquery-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop osquery

# Restore from backup
tar -xzf /backup/osquery/osquery-backup-*.tar.gz -C /

# Start service
sudo systemctl start osquery
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u osquery -n 100
sudo tail -f /var/log/osquery/osquery.log

# Check configuration
osquery --version

# Check permissions
ls -la /etc/osquery
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9090

# Test connectivity
telnet localhost 9090

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep osquery)

# Check disk I/O
iotop -p $(pgrep osquery)

# Check connections
ss -an | grep 9090
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  osquery:
    image: osquery:latest
    ports:
      - "9090:9090"
    volumes:
      - ./config:/etc/osquery
      - ./data:/var/lib/osquery
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update osquery

# Debian/Ubuntu
sudo apt update && sudo apt upgrade osquery

# Arch Linux
sudo pacman -Syu osquery

# Alpine Linux
apk update && apk upgrade osquery

# openSUSE
sudo zypper update osquery

# FreeBSD
pkg update && pkg upgrade osquery

# Always backup before updates
tar -czf /backup/osquery-pre-update-$(date +%Y%m%d).tar.gz /etc/osquery

# Restart after updates
sudo systemctl restart osquery
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/osquery

# Clean old logs
find /var/log/osquery -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/osquery
```

## Additional Resources

- Official Documentation: https://docs.osquery.org/
- GitHub Repository: https://github.com/osquery/osquery
- Community Forum: https://forum.osquery.org/
- Best Practices Guide: https://docs.osquery.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
