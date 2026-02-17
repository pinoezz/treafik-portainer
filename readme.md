# 🚀 Production Setup - Traefik + Portainer

Setup lengkap production-ready untuk Traefik reverse proxy dengan Let's Encrypt SSL (Cloudflare DNS) dan Portainer untuk container management.

## 📋 Overview

Setup ini terdiri dari 2 komponen utama:

1. **Traefik** - Reverse proxy dengan auto SSL dari Let's Encrypt
2. **Portainer** - Web-based container management

Kedua service berjalan di folder terpisah untuk better organization dan maintenance.

## 📁 Struktur Project

```
treafik/
├── traefik-cloudflare/          # Traefik dengan Let's Encrypt
│   ├── docker-compose.yml
│   ├── .env
│   ├── .env.example
│   ├── data/
│   │   ├── traefik.yml
│   │   ├── config.yml
│   │   └── acme.json
│   ├── logs/
│   └── README.md
│
├── portainer/                    # Portainer container management
│   ├── docker-compose.yml
│   ├── .env
│   ├── .env.example
│   ├── data/
│   └── README.md
│
└── README-PRODUCTION.md          # File ini
```

## 🎯 Arsitektur

```
Internet
    ↓
Cloudflare DNS
    ↓
Your Server (IP Public)
    ↓
Traefik (Port 80/443)
    ├── traefik.yourdomain.com → Traefik Dashboard
    └── portainer.yourdomain.com → Portainer
```

## 🔧 Prerequisites

### 1. Server Requirements

- **OS**: Linux (Ubuntu 20.04+ recommended) atau macOS
- **RAM**: Minimal 1GB (recommended 2GB+)
- **Storage**: Minimal 10GB free space
- **Network**: Public IP address

### 2. Software Requirements

```bash
# Docker
docker --version
# Docker version 20.10+

# Docker Compose
docker-compose --version
# Docker Compose version 1.29+ atau 2.0+
```

Install Docker jika belum ada:

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# macOS
brew install docker docker-compose
```

### 3. Domain & DNS

- Domain yang sudah terdaftar
- Domain menggunakan Cloudflare nameservers
- Akses ke Cloudflare dashboard

### 4. Cloudflare API Token

Dapatkan dari: https://dash.cloudflare.com/profile/api-tokens

## 🚀 Quick Start

### Step 1: Clone atau Download Project

```bash
cd /path/to/your/workspace
# Project sudah ada di: /Users/850165/pino/treafik
```

### Step 2: Setup Traefik

```bash
cd traefik-cloudflare

# Copy environment template
cp .env.example .env

# Edit dengan credentials Anda
nano .env
```

Isi `.env`:

```env
DOMAIN=yourdomain.com
CF_API_EMAIL=your-email@example.com
CF_DNS_API_TOKEN=your-cloudflare-dns-api-token
TRAEFIK_DASHBOARD_USERS=admin:$$apr1$$xxx$$xxx
```

Setup permissions:

```bash
# Create acme.json
touch data/acme.json
chmod 600 data/acme.json

# Create logs directory
mkdir -p logs
```

Setup DNS di Cloudflare:

```
A Record:
- Name: @, Content: YOUR_SERVER_IP, Proxy: DNS only
- Name: *, Content: YOUR_SERVER_IP, Proxy: DNS only
- Name: traefik, Content: YOUR_SERVER_IP, Proxy: DNS only
```

Start Traefik:

```bash
# Create network
docker network create proxy

# Start Traefik
docker-compose up -d

# Check logs
docker-compose logs -f
```

### Step 3: Setup Portainer

```bash
cd ../portainer

# Copy environment template
cp .env.example .env

# Edit dengan domain yang SAMA
nano .env
```

Isi `.env`:

```env
DOMAIN=yourdomain.com
```

Setup DNS di Cloudflare:

```
A Record:
- Name: portainer, Content: YOUR_SERVER_IP, Proxy: DNS only
```

Start Portainer:

```bash
# Start Portainer
docker-compose up -d

# Check logs
docker-compose logs -f
```

### Step 4: Initial Configuration

**Traefik Dashboard:**
1. Akses: `https://traefik.yourdomain.com`
2. Login dengan username/password dari `.env`

**Portainer:**
1. Akses: `https://portainer.yourdomain.com`
2. Buat admin user (5 menit timeout!)
3. Pilih Docker environment
4. Connect

## 🔒 Security Checklist

### ✅ Pre-deployment

- [ ] Ganti password default Traefik dashboard
- [ ] Generate strong password untuk Portainer admin
- [ ] Cloudflare API token dengan minimal permissions (DNS Edit only)
- [ ] Set `acme.json` permission ke 600
- [ ] Review security headers di `config.yml`
- [ ] Enable Cloudflare DNS only (grey cloud, bukan orange)

### ✅ Post-deployment

- [ ] Verify SSL certificates ter-generate (check `acme.json`)
- [ ] Test HTTPS redirect (HTTP → HTTPS)
- [ ] Test security headers (use securityheaders.com)
- [ ] Setup firewall (UFW/iptables)
- [ ] Enable fail2ban (optional)
- [ ] Setup monitoring (optional)
- [ ] Backup `acme.json` dan Portainer data

### ✅ Ongoing Maintenance

- [ ] Regular updates (monthly)
- [ ] Monitor logs untuk suspicious activity
- [ ] Backup data (weekly)
- [ ] Review access logs
- [ ] Update SSL certificates (auto-renewal check)

## 🛡️ Firewall Configuration

### UFW (Ubuntu)

```bash
# Install UFW
sudo apt-get install ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

### iptables

```bash
# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
```

## 📊 Monitoring & Logging

### View Logs

```bash
# Traefik logs
cd traefik-cloudflare
docker-compose logs -f

# Portainer logs
cd portainer
docker-compose logs -f

# System logs
tail -f traefik-cloudflare/logs/traefik.log
tail -f traefik-cloudflare/logs/access.log
```

### Monitoring Tools (Optional)

Tambahkan monitoring stack:

- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Loki** - Log aggregation

## 🔄 Backup & Restore

### Backup Script

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/traefik-$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup Traefik
cp traefik-cloudflare/data/acme.json $BACKUP_DIR/
cp traefik-cloudflare/.env $BACKUP_DIR/traefik.env

# Backup Portainer
tar -czf $BACKUP_DIR/portainer-data.tar.gz portainer/data/
cp portainer/.env $BACKUP_DIR/portainer.env

echo "Backup completed: $BACKUP_DIR"
```

### Restore

```bash
# Restore Traefik
cp backup/acme.json traefik-cloudflare/data/
chmod 600 traefik-cloudflare/data/acme.json

# Restore Portainer
tar -xzf backup/portainer-data.tar.gz -C portainer/
```

### Automated Backup (Cron)

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /path/to/backup.sh
```

## 🔄 Update Procedure

### Update Traefik

```bash
cd traefik-cloudflare

# Backup first
cp data/acme.json data/acme.json.backup

# Pull latest image
docker-compose pull

# Recreate container
docker-compose up -d

# Check logs
docker-compose logs -f
```

### Update Portainer

```bash
cd portainer

# Backup first
tar -czf data-backup.tar.gz data/

# Pull latest image
docker-compose pull

# Recreate container
docker-compose up -d

# Check logs
docker-compose logs -f
```

## 🐛 Common Issues

### 1. SSL Certificate tidak ter-generate

**Symptoms**: `acme.json` kosong atau error di logs

**Solutions**:
```bash
# Check Cloudflare API token
echo $CF_DNS_API_TOKEN

# Check DNS propagation
nslookup yourdomain.com

# Check Traefik logs
docker-compose logs traefik | grep acme

# Verify acme.json permission
ls -la data/acme.json  # Should be -rw-------
```

### 2. Dashboard tidak bisa diakses

**Solutions**:
```bash
# Check container running
docker ps | grep traefik

# Check DNS
nslookup traefik.yourdomain.com

# Check logs
docker-compose logs traefik
```

### 3. Portainer tidak bisa connect ke Docker

**Solutions**:
```bash
# Check Docker socket
ls -la /var/run/docker.sock

# Restart Portainer
docker-compose restart
```

## 📚 Adding New Services

Untuk menambah service baru:

### 1. Buat folder service

```bash
mkdir myapp
cd myapp
```

### 2. Buat docker-compose.yml

```yaml
version: '3.8'

services:
  myapp:
    image: myapp:latest
    container_name: myapp
    restart: unless-stopped
    networks:
      - proxy
    environment:
      - APP_ENV=production
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`myapp.${DOMAIN}`)"
      - "traefik.http.routers.myapp.entrypoints=https"
      - "traefik.http.routers.myapp.tls=true"
      - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
      - "traefik.http.services.myapp.loadbalancer.server.port=8080"
      - "traefik.docker.network=proxy"

networks:
  proxy:
    external: true
```

### 3. Setup DNS

```
A Record:
- Name: myapp, Content: YOUR_SERVER_IP, Proxy: DNS only
```

### 4. Deploy

```bash
docker-compose up -d
```

SSL certificate akan otomatis ter-generate oleh Traefik!

## 📖 Documentation

- [Traefik README](traefik-cloudflare/README.md) - Detail setup Traefik
- [Portainer README](portainer/README.md) - Detail setup Portainer
- [Traefik Docs](https://doc.traefik.io/traefik/)
- [Portainer Docs](https://docs.portainer.io/)

## 🆘 Support

Jika ada masalah:

1. Check README di masing-masing folder
2. Check logs: `docker-compose logs -f`
3. Check Traefik dashboard untuk routing issues
4. Check Portainer untuk container status

## ⚠️ Important Notes

1. **Cloudflare Proxy**: HARUS disabled (grey cloud) untuk semua DNS records
2. **API Token**: Gunakan DNS API Token, bukan Global API Key
3. **Rate Limits**: Let's Encrypt punya rate limits (50 certs/week per domain)
4. **Backup**: Backup `acme.json` dan Portainer data secara berkala
5. **Security**: Ganti semua password default!
6. **Updates**: Update images secara berkala untuk security patches

## 🎓 Next Steps

Setelah setup dasar:

1. ✅ Setup monitoring (Prometheus + Grafana)
2. ✅ Setup log aggregation (Loki)
3. ✅ Setup automated backups
4. ✅ Setup fail2ban untuk brute-force protection
5. ✅ Deploy aplikasi Anda sendiri
6. ✅ Setup CI/CD pipeline

---

**Production-Ready Setup by DevOps Team** 🚀

*Last updated: 2026-02-17*
