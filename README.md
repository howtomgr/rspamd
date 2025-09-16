# Rspamd Installation Guide

Rspamd is a free and open-source Anti-Spam. Fast, free and open-source spam filtering system

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 11333/11334 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 11333/11334 (default rspamd port)
- **Dependencies**:
  - redis, lua
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

# Install rspamd
sudo dnf install -y rspamd redis, lua

# Enable and start service
sudo systemctl enable --now rspamd

# Configure firewall
sudo firewall-cmd --permanent --add-service=rspamd
sudo firewall-cmd --reload

# Verify installation
rspamd --version || systemctl status rspamd
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rspamd
sudo apt install -y rspamd redis, lua

# Enable and start service
sudo systemctl enable --now rspamd

# Configure firewall
sudo ufw allow 11333/11334

# Verify installation
rspamd --version || systemctl status rspamd
```

### Arch Linux

```bash
# Install rspamd
sudo pacman -S rspamd

# Enable and start service
sudo systemctl enable --now rspamd

# Verify installation
rspamd --version || systemctl status rspamd
```

### Alpine Linux

```bash
# Install rspamd
apk add --no-cache rspamd

# Enable and start service
rc-update add rspamd default
rc-service rspamd start

# Verify installation
rspamd --version || rc-service rspamd status
```

### openSUSE/SLES

```bash
# Install rspamd
sudo zypper install -y rspamd redis, lua

# Enable and start service
sudo systemctl enable --now rspamd

# Configure firewall
sudo firewall-cmd --permanent --add-service=rspamd
sudo firewall-cmd --reload

# Verify installation
rspamd --version || systemctl status rspamd
```

### macOS

```bash
# Using Homebrew
brew install rspamd

# Start service
brew services start rspamd

# Verify installation
rspamd --version
```

### FreeBSD

```bash
# Using pkg
pkg install rspamd

# Enable in rc.conf
echo 'rspamd_enable="YES"' >> /etc/rc.conf

# Start service
service rspamd start

# Verify installation
rspamd --version || service rspamd status
```

### Windows

```powershell
# Using Chocolatey
choco install rspamd

# Or using Scoop
scoop install rspamd

# Verify installation
rspamd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/rspamd

# Set up basic configuration
sudo tee /etc/rspamd/rspamd.conf << 'EOF'
# Rspamd Configuration
workers = 4
EOF

# Test configuration
sudo rspamd -t || sudo rspamd configtest

# Reload service
sudo systemctl reload rspamd
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R rspamd:rspamd /etc/rspamd
sudo chmod 750 /etc/rspamd

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable rspamd

# Start service
sudo systemctl start rspamd

# Stop service
sudo systemctl stop rspamd

# Restart service
sudo systemctl restart rspamd

# Reload configuration
sudo systemctl reload rspamd

# Check status
sudo systemctl status rspamd

# View logs
sudo journalctl -u rspamd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add rspamd default

# Start service
rc-service rspamd start

# Stop service
rc-service rspamd stop

# Restart service
rc-service rspamd restart

# Check status
rc-service rspamd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'rspamd_enable="YES"' >> /etc/rc.conf

# Start service
service rspamd start

# Stop service
service rspamd stop

# Restart service
service rspamd restart

# Check status
service rspamd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rspamd
brew services stop rspamd
brew services restart rspamd

# Check status
brew services list | grep rspamd
```

### Windows Service Manager

```powershell
# Start service
net start rspamd

# Stop service
net stop rspamd

# Using PowerShell
Start-Service rspamd
Stop-Service rspamd
Restart-Service rspamd

# Check status
Get-Service rspamd
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/rspamd/rspamd.conf << 'EOF'
workers = 4
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart rspamd
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream rspamd_backend {
    server 127.0.0.1:11333/11334;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name rspamd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rspamd.example.com;

    ssl_certificate /etc/ssl/certs/rspamd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rspamd.example.com.key;

    location / {
        proxy_pass http://rspamd_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName rspamd.example.com
    Redirect permanent / https://rspamd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rspamd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rspamd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rspamd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:11333/11334/
    ProxyPassReverse / http://127.0.0.1:11333/11334/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:11333/11334/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rspamd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rspamd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rspamd_backend

backend rspamd_backend
    balance roundrobin
    option httpchk GET /health
    server rspamd1 127.0.0.1:11333/11334 check
    server rspamd2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rspamd:rspamd /etc/rspamd
sudo chmod 750 /etc/rspamd

# Configure firewall
sudo firewall-cmd --permanent --add-service=rspamd
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/rspamd.conf << 'EOF'
[rspamd]
enabled = true
port = 11333/11334
filter = rspamd
logpath = /var/log/rspamd/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/rspamd.key \
    -out /etc/ssl/certs/rspamd.crt

# Configure SSL in rspamd
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE rspamd_db;
CREATE USER rspamd_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE rspamd_db TO rspamd_user;
EOF

# Configure rspamd to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE rspamd_db;
CREATE USER 'rspamd_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON rspamd_db.* TO 'rspamd_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Rspamd specific tuning
workers = 4
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
rspamd soft nofile 65535
rspamd hard nofile 65535
rspamd soft nproc 32768
rspamd hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'rspamd'
    static_configs:
      - targets: ['localhost:11333/11334']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet rspamd; then
    echo "Rspamd is running"
    exit 0
else
    echo "Rspamd is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/rspamd << 'EOF'
/var/log/rspamd/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 rspamd rspamd
    postrotate
        systemctl reload rspamd > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Rspamd backup script
BACKUP_DIR="/backup/rspamd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop rspamd

# Backup configuration
tar -czf "$BACKUP_DIR/rspamd-config-$DATE.tar.gz" /etc/rspamd

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/rspamd-data-$DATE.tar.gz" /var/lib/rspamd

# Start service
systemctl start rspamd

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop rspamd

# Restore configuration
sudo tar -xzf /backup/rspamd/rspamd-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/rspamd/rspamd-data-*.tar.gz -C /

# Set permissions
sudo chown -R rspamd:rspamd /etc/rspamd
sudo chown -R rspamd:rspamd /var/lib/rspamd

# Start service
sudo systemctl start rspamd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u rspamd -n 100
sudo tail -f /var/log/rspamd/*.log

# Check configuration
sudo rspamd -t || sudo rspamd configtest

# Check permissions
ls -la /etc/rspamd
ls -la /var/lib/rspamd
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 11333/11334
sudo netstat -tlnp | grep 11333/11334

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 11333/11334
nc -zv localhost 11333/11334
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep rspamd)
htop -p $(pgrep rspamd)

# Check connections
ss -ant | grep :11333/11334 | wc -l

# Monitor I/O
iotop -p $(pgrep rspamd)
```

### Debug Mode

```bash
# Run in debug mode
sudo rspamd -d
# or
sudo rspamd debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  rspamd:
    image: rspamd:latest
    container_name: rspamd
    ports:
      - "11333/11334:11333/11334"
    volumes:
      - ./config:/etc/rspamd
      - ./data:/var/lib/rspamd
    environment:
      - rspamd_CONFIG=/etc/rspamd/rspamd.conf
    restart: unless-stopped
    networks:
      - rspamd_net

networks:
  rspamd_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rspamd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rspamd
  template:
    metadata:
      labels:
        app: rspamd
    spec:
      containers:
      - name: rspamd
        image: rspamd:latest
        ports:
        - containerPort: 11333/11334
        volumeMounts:
        - name: config
          mountPath: /etc/rspamd
      volumes:
      - name: config
        configMap:
          name: rspamd-config
---
apiVersion: v1
kind: Service
metadata:
  name: rspamd
spec:
  selector:
    app: rspamd
  ports:
  - port: 11333/11334
    targetPort: 11333/11334
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Rspamd
  hosts: all
  become: yes
  tasks:
    - name: Install rspamd
      package:
        name: rspamd
        state: present
    
    - name: Configure rspamd
      template:
        src: rspamd.conf.j2
        dest: /etc/rspamd/rspamd.conf
        owner: rspamd
        group: rspamd
        mode: '0640'
      notify: restart rspamd
    
    - name: Start and enable rspamd
      systemd:
        name: rspamd
        state: started
        enabled: yes
  
  handlers:
    - name: restart rspamd
      systemd:
        name: rspamd
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rspamd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rspamd

# Arch Linux
sudo pacman -Syu rspamd

# Alpine Linux
apk update && apk upgrade rspamd

# openSUSE
sudo zypper update rspamd

# FreeBSD
pkg update && pkg upgrade rspamd

# Always backup before updates
tar -czf /backup/rspamd-pre-update-$(date +%Y%m%d).tar.gz /etc/rspamd

# Restart after updates
sudo systemctl restart rspamd
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/rspamd -name "*.log" -mtime +30 -delete

# Verify integrity
sudo rspamd --verify || sudo rspamd check

# Update databases (if applicable)
sudo rspamd-update-db

# Optimize performance
sudo rspamd-optimize

# Check for security updates
sudo rspamd --security-check
```

## Additional Resources

- Official Documentation: https://docs.rspamd.org/
- GitHub Repository: https://github.com/rspamd/rspamd
- Community Forum: https://forum.rspamd.org/
- Wiki: https://wiki.rspamd.org/
- Comparison vs SpamAssassin, ASSP, MailScanner, SpamTitan: https://docs.rspamd.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
