# 🐳 Portainer dengan Traefik SSL

Portainer adalah web-based container management tool yang powerful untuk mengelola Docker containers, images, networks, dan volumes.

## 📋 Fitur

- ✅ **Web GUI** - Interface yang user-friendly
- ✅ **Container Management** - Start, stop, restart containers
- ✅ **Image Management** - Pull, build, push images
- ✅ **Volume Management** - Manage Docker volumes
- ✅ **Network Management** - Manage Docker networks
- ✅ **Auto SSL** - HTTPS via Traefik + Let's Encrypt
- ✅ **Stack Deployment** - Deploy Docker Compose stacks

## 🔧 Prerequisites

1. **Traefik sudah running** - Setup dari folder `traefik-cloudflare`
2. **Network proxy** - Network `proxy` sudah dibuat
3. **Domain** - Subdomain untuk Portainer (e.g., `portainer.yourdomain.com`)

## 📁 Struktur Folder

```
portainer/
├── docker-compose.yml     # Konfigurasi Portainer
├── .env                   # Environment variables (BUAT SENDIRI)
├── .env.example          # Template environment variables
├── data/                 # Portainer data (auto-generated)
└── README.md             # File ini
```

## 🚀 Cara Setup

### Step 1: Setup Environment Variables

```bash
# Copy template
cp .env.example .env

# Edit dengan domain Anda
nano .env
```

Isi file `.env`:

```env
DOMAIN=yourdomain.com
```

**PENTING**: Domain harus sama dengan yang digunakan di Traefik!

### Step 2: Setup DNS

Tambahkan A record di Cloudflare:

```
Type: A
Name: portainer
Content: YOUR_SERVER_IP
Proxy: Disabled (DNS only)
```

### Step 3: Pastikan Traefik Running

```bash
# Check Traefik
docker ps | grep traefik

# Check network proxy
docker network ls | grep proxy
```

Jika network `proxy` belum ada:

```bash
docker network create proxy
```

### Step 4: Jalankan Portainer

```bash
# Start Portainer
docker-compose up -d

# Lihat logs
docker-compose logs -f
```

### Step 5: Initial Setup

1. Akses `https://portainer.yourdomain.com`
2. Buat admin user (username & password)
3. Pilih **Docker** sebagai environment
4. Klik **Connect**

**PENTING**: Anda punya 5 menit untuk membuat admin user setelah first start!

## 🎯 Akses Portainer

```
URL: https://portainer.yourdomain.com
```

SSL certificate otomatis di-handle oleh Traefik via Let's Encrypt.

## 📖 Cara Menggunakan Portainer

### 1. Dashboard

Dashboard menampilkan overview:
- Total containers (running/stopped)
- Total images
- Total volumes
- Total networks
- System resources

### 2. Container Management

**Containers** → Pilih container:
- **Start/Stop/Restart** - Control container lifecycle
- **Logs** - View container logs
- **Inspect** - View container details
- **Stats** - View resource usage
- **Console** - Access container shell
- **Duplicate** - Clone container configuration

### 3. Deploy Stack (Docker Compose)

**Stacks** → **Add stack**:

1. Pilih nama stack
2. Paste Docker Compose YAML atau upload file
3. Set environment variables (optional)
4. Klik **Deploy the stack**

Contoh stack sederhana:

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nginx.rule=Host(`nginx.${DOMAIN}`)"
      - "traefik.http.routers.nginx.entrypoints=https"
      - "traefik.http.routers.nginx.tls=true"
      - "traefik.http.routers.nginx.tls.certresolver=cloudflare"

networks:
  proxy:
    external: true
```

### 4. Image Management

**Images** → Manage images:
- **Pull** - Download image dari registry
- **Build** - Build dari Dockerfile
- **Remove** - Hapus unused images
- **Export** - Export image ke tar file

### 5. Volume Management

**Volumes** → Manage volumes:
- **Create** - Buat volume baru
- **Browse** - Lihat isi volume
- **Remove** - Hapus volume

### 6. Network Management

**Networks** → Manage networks:
- **Create** - Buat network baru
- **Remove** - Hapus network

## 🔒 Security Best Practices

### 1. Strong Password

Gunakan password yang kuat untuk admin user:
- Minimal 12 karakter
- Kombinasi huruf besar, kecil, angka, dan simbol
- Jangan gunakan password yang sama dengan service lain

### 2. Regular Updates

```bash
# Update Portainer
docker-compose pull
docker-compose up -d
```

### 3. Backup Data

```bash
# Backup Portainer data
tar -czf portainer-backup-$(date +%Y%m%d).tar.gz data/

# Restore
tar -xzf portainer-backup-YYYYMMDD.tar.gz
```

### 4. Access Control

Portainer mendukung:
- **Teams** - Group users
- **Roles** - Assign permissions
- **RBAC** - Role-based access control

Setup di: **Settings** → **Users** / **Teams**

### 5. Audit Logs

Lihat semua aktivitas di: **Settings** → **Activity logs**

## 🔍 Monitoring

### Container Stats

**Containers** → Pilih container → **Stats**

Menampilkan:
- CPU usage
- Memory usage
- Network I/O
- Block I/O

### System Resources

**Dashboard** → **Environment details**

Menampilkan:
- Total CPU
- Total Memory
- Docker version
- System info

## 🐛 Troubleshooting

### Portainer tidak bisa diakses

```bash
# Check container running
docker ps | grep portainer

# Check logs
docker-compose logs portainer

# Check DNS
nslookup portainer.yourdomain.com
```

### "Unable to connect to Docker"

```bash
# Check Docker socket permission
ls -la /var/run/docker.sock

# Restart Portainer
docker-compose restart
```

### SSL Certificate error

```bash
# Check Traefik logs
cd ../traefik-cloudflare
docker-compose logs traefik | grep portainer

# Pastikan domain benar di .env
cat .env
```

### Lupa admin password

```bash
# Stop Portainer
docker-compose down

# Reset admin password
docker run --rm -v portainer_data:/data portainer/helper-reset-password

# Start Portainer
docker-compose up -d
```

## 🔄 Update Portainer

```bash
# Pull latest image
docker-compose pull

# Recreate container
docker-compose up -d

# Cleanup old images
docker image prune -f
```

## 🗑️ Uninstall

```bash
# Stop dan hapus container
docker-compose down

# Hapus data (HATI-HATI!)
rm -rf data/
```

## 📚 Tips & Tricks

### 1. Quick Actions

Gunakan **Quick actions** di container list untuk:
- Start/Stop multiple containers
- Remove multiple containers
- Export container logs

### 2. Templates

Gunakan **App Templates** untuk deploy aplikasi populer dengan 1 klik:
- WordPress
- MySQL
- PostgreSQL
- Redis
- MongoDB
- dll

### 3. Webhooks

Setup webhooks untuk auto-update containers:

**Containers** → Pilih container → **Webhook**

Copy webhook URL dan trigger dengan:

```bash
curl -X POST https://portainer.yourdomain.com/api/webhooks/xxx
```

### 4. Edge Agent

Untuk manage remote Docker hosts:

**Environments** → **Add environment** → **Edge Agent**

### 5. Registry Management

Tambahkan private registry:

**Registries** → **Add registry**

Support:
- Docker Hub
- GitLab
- GitHub Container Registry
- AWS ECR
- Azure ACR
- Custom registry

## 📖 Resources

- [Portainer Documentation](https://docs.portainer.io/)
- [Portainer GitHub](https://github.com/portainer/portainer)
- [Portainer Community](https://www.portainer.io/community)

## ⚠️ Important Notes

1. **First Setup**: Anda punya 5 menit untuk create admin user setelah first start
2. **Backup**: Backup folder `data/` secara berkala
3. **Updates**: Selalu backup sebelum update
4. **Security**: Jangan expose Portainer ke public tanpa authentication
5. **Network**: Pastikan container di network `proxy` untuk Traefik integration

---

**Happy Container Management!** 🐳
