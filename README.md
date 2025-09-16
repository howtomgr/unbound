# Unbound Installation Guide

Unbound is a free and open-source DNS Resolver. A validating, recursive, and caching DNS resolver

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
  - Network: 53 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 53 (default unbound port)
- **Dependencies**:
  - unbound-anchor, unbound-control
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

# Install unbound
sudo dnf install -y unbound unbound-anchor, unbound-control

# Enable and start service
sudo systemctl enable --now unbound

# Configure firewall
sudo firewall-cmd --permanent --add-service=unbound
sudo firewall-cmd --reload

# Verify installation
unbound --version || systemctl status unbound
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install unbound
sudo apt install -y unbound unbound-anchor, unbound-control

# Enable and start service
sudo systemctl enable --now unbound

# Configure firewall
sudo ufw allow 53

# Verify installation
unbound --version || systemctl status unbound
```

### Arch Linux

```bash
# Install unbound
sudo pacman -S unbound

# Enable and start service
sudo systemctl enable --now unbound

# Verify installation
unbound --version || systemctl status unbound
```

### Alpine Linux

```bash
# Install unbound
apk add --no-cache unbound

# Enable and start service
rc-update add unbound default
rc-service unbound start

# Verify installation
unbound --version || rc-service unbound status
```

### openSUSE/SLES

```bash
# Install unbound
sudo zypper install -y unbound unbound-anchor, unbound-control

# Enable and start service
sudo systemctl enable --now unbound

# Configure firewall
sudo firewall-cmd --permanent --add-service=unbound
sudo firewall-cmd --reload

# Verify installation
unbound --version || systemctl status unbound
```

### macOS

```bash
# Using Homebrew
brew install unbound

# Start service
brew services start unbound

# Verify installation
unbound --version
```

### FreeBSD

```bash
# Using pkg
pkg install unbound

# Enable in rc.conf
echo 'unbound_enable="YES"' >> /etc/rc.conf

# Start service
service unbound start

# Verify installation
unbound --version || service unbound status
```

### Windows

```powershell
# Using Chocolatey
choco install unbound

# Or using Scoop
scoop install unbound

# Verify installation
unbound --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/unbound

# Set up basic configuration
sudo tee /etc/unbound/unbound.conf << 'EOF'
# Unbound Configuration
num-threads: 4, msg-cache-size: 50m, rrset-cache-size: 100m
EOF

# Test configuration
sudo unbound -t || sudo unbound configtest

# Reload service
sudo systemctl reload unbound
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R unbound:unbound /etc/unbound
sudo chmod 750 /etc/unbound

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable unbound

# Start service
sudo systemctl start unbound

# Stop service
sudo systemctl stop unbound

# Restart service
sudo systemctl restart unbound

# Reload configuration
sudo systemctl reload unbound

# Check status
sudo systemctl status unbound

# View logs
sudo journalctl -u unbound -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add unbound default

# Start service
rc-service unbound start

# Stop service
rc-service unbound stop

# Restart service
rc-service unbound restart

# Check status
rc-service unbound status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'unbound_enable="YES"' >> /etc/rc.conf

# Start service
service unbound start

# Stop service
service unbound stop

# Restart service
service unbound restart

# Check status
service unbound status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start unbound
brew services stop unbound
brew services restart unbound

# Check status
brew services list | grep unbound
```

### Windows Service Manager

```powershell
# Start service
net start unbound

# Stop service
net stop unbound

# Using PowerShell
Start-Service unbound
Stop-Service unbound
Restart-Service unbound

# Check status
Get-Service unbound
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/unbound/unbound.conf << 'EOF'
num-threads: 4, msg-cache-size: 50m, rrset-cache-size: 100m
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart unbound
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
upstream unbound_backend {
    server 127.0.0.1:53;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name unbound.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name unbound.example.com;

    ssl_certificate /etc/ssl/certs/unbound.example.com.crt;
    ssl_certificate_key /etc/ssl/private/unbound.example.com.key;

    location / {
        proxy_pass http://unbound_backend;
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
    ServerName unbound.example.com
    Redirect permanent / https://unbound.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName unbound.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/unbound.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/unbound.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:53/
    ProxyPassReverse / http://127.0.0.1:53/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:53/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend unbound_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/unbound.pem
    redirect scheme https if !{ ssl_fc }
    default_backend unbound_backend

backend unbound_backend
    balance roundrobin
    option httpchk GET /health
    server unbound1 127.0.0.1:53 check
    server unbound2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R unbound:unbound /etc/unbound
sudo chmod 750 /etc/unbound

# Configure firewall
sudo firewall-cmd --permanent --add-service=unbound
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/unbound.conf << 'EOF'
[unbound]
enabled = true
port = 53
filter = unbound
logpath = /var/log/unbound/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/unbound.key \
    -out /etc/ssl/certs/unbound.crt

# Configure SSL in unbound
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE unbound_db;
CREATE USER unbound_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE unbound_db TO unbound_user;
EOF

# Configure unbound to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE unbound_db;
CREATE USER 'unbound_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON unbound_db.* TO 'unbound_user'@'localhost';
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

# Unbound specific tuning
num-threads: 4, msg-cache-size: 50m, rrset-cache-size: 100m
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
unbound soft nofile 65535
unbound hard nofile 65535
unbound soft nproc 32768
unbound hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'unbound'
    static_configs:
      - targets: ['localhost:53']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet unbound; then
    echo "Unbound is running"
    exit 0
else
    echo "Unbound is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/unbound << 'EOF'
/var/log/unbound/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 unbound unbound
    postrotate
        systemctl reload unbound > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Unbound backup script
BACKUP_DIR="/backup/unbound"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop unbound

# Backup configuration
tar -czf "$BACKUP_DIR/unbound-config-$DATE.tar.gz" /etc/unbound

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/unbound-data-$DATE.tar.gz" /var/lib/unbound

# Start service
systemctl start unbound

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop unbound

# Restore configuration
sudo tar -xzf /backup/unbound/unbound-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/unbound/unbound-data-*.tar.gz -C /

# Set permissions
sudo chown -R unbound:unbound /etc/unbound
sudo chown -R unbound:unbound /var/lib/unbound

# Start service
sudo systemctl start unbound
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u unbound -n 100
sudo tail -f /var/log/unbound/*.log

# Check configuration
sudo unbound -t || sudo unbound configtest

# Check permissions
ls -la /etc/unbound
ls -la /var/lib/unbound
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 53
sudo netstat -tlnp | grep 53

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 53
nc -zv localhost 53
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep unbound)
htop -p $(pgrep unbound)

# Check connections
ss -ant | grep :53 | wc -l

# Monitor I/O
iotop -p $(pgrep unbound)
```

### Debug Mode

```bash
# Run in debug mode
sudo unbound -d
# or
sudo unbound debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  unbound:
    image: unbound:latest
    container_name: unbound
    ports:
      - "53:53"
    volumes:
      - ./config:/etc/unbound
      - ./data:/var/lib/unbound
    environment:
      - unbound_CONFIG=/etc/unbound/unbound.conf
    restart: unless-stopped
    networks:
      - unbound_net

networks:
  unbound_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: unbound
spec:
  replicas: 3
  selector:
    matchLabels:
      app: unbound
  template:
    metadata:
      labels:
        app: unbound
    spec:
      containers:
      - name: unbound
        image: unbound:latest
        ports:
        - containerPort: 53
        volumeMounts:
        - name: config
          mountPath: /etc/unbound
      volumes:
      - name: config
        configMap:
          name: unbound-config
---
apiVersion: v1
kind: Service
metadata:
  name: unbound
spec:
  selector:
    app: unbound
  ports:
  - port: 53
    targetPort: 53
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Unbound
  hosts: all
  become: yes
  tasks:
    - name: Install unbound
      package:
        name: unbound
        state: present
    
    - name: Configure unbound
      template:
        src: unbound.conf.j2
        dest: /etc/unbound/unbound.conf
        owner: unbound
        group: unbound
        mode: '0640'
      notify: restart unbound
    
    - name: Start and enable unbound
      systemd:
        name: unbound
        state: started
        enabled: yes
  
  handlers:
    - name: restart unbound
      systemd:
        name: unbound
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update unbound

# Debian/Ubuntu
sudo apt update && sudo apt upgrade unbound

# Arch Linux
sudo pacman -Syu unbound

# Alpine Linux
apk update && apk upgrade unbound

# openSUSE
sudo zypper update unbound

# FreeBSD
pkg update && pkg upgrade unbound

# Always backup before updates
tar -czf /backup/unbound-pre-update-$(date +%Y%m%d).tar.gz /etc/unbound

# Restart after updates
sudo systemctl restart unbound
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/unbound -name "*.log" -mtime +30 -delete

# Verify integrity
sudo unbound --verify || sudo unbound check

# Update databases (if applicable)
sudo unbound-update-db

# Optimize performance
sudo unbound-optimize

# Check for security updates
sudo unbound --security-check
```

## Additional Resources

- Official Documentation: https://docs.unbound.org/
- GitHub Repository: https://github.com/unbound/unbound
- Community Forum: https://forum.unbound.org/
- Wiki: https://wiki.unbound.org/
- Comparison vs BIND, PowerDNS Recursor, Knot Resolver: https://docs.unbound.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
