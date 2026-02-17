# 🚀 Quick Start - Localhost Testing

Setup sederhana Traefik + Portainer untuk testing di localhost tanpa SSL.

## 📋 Yang Akan Jalan

- **Traefik** (latest version) - Reverse proxy dengan dashboard
- **Portainer** (latest version) - Container management GUI

Semua jalan di HTTP (tanpa SSL) untuk testing lokal.

## 🔧 Prerequisites

1. **Docker Desktop** - Harus sudah running
2. **Docker Compose** - Biasanya sudah include di Docker Desktop

Cek dengan:
```bash
docker --version
docker-compose --version
```

## 🚀 Cara Menjalankan

### Step 1: Start Docker Desktop

Pastikan Docker Desktop sudah running (lihat icon di menu bar).

### Step 2: Jalankan Traefik

```bash
# Masuk ke folder traefik
cd /Users/850165/pino/treafik/traefik-cloudflare

# Buat network
docker network create proxy

# Start Traefik
docker-compose up -d

# Lihat logs (optional)
docker-compose logs -f
```

### Step 3: Jalankan Portainer

```bash
# Masuk ke folder portainer
cd /Users/850165/pino/treafik/portainer

# Start Portainer
docker-compose up -d

# Lihat logs (optional)
docker-compose logs -f
```

### Step 4: Akses Dashboard

**Traefik Dashboard:**
- URL: http://localhost:8080
- Username: `admin`
- Password: `admin`

**Portainer:**
- URL: http://portainer.localhost

Jika `portainer.localhost` tidak work, coba:
- http://localhost:9000 (direct access)

### Step 5: Setup Portainer (First Time)

1. Buka http://portainer.localhost atau http://localhost:9000
2. Buat admin user (username + password)
3. **PENTING**: Anda punya 5 menit untuk setup!
4. Pilih **Docker** environment
5. Klik **Connect**

## 🎯 Testing

### Cek Traefik Dashboard

1. Buka http://localhost:8080
2. Login dengan `admin` / `admin`
3. Anda akan lihat:
   - **HTTP Routers** - List semua routes
   - **Services** - List semua services
   - **Middlewares** - List middleware yang aktif

### Cek Portainer

1. Buka http://portainer.localhost
2. Login dengan user yang Anda buat
3. Explore:
   - **Dashboard** - Overview containers
   - **Containers** - Manage containers
   - **Images** - Manage images
   - **Networks** - Manage networks
   - **Volumes** - Manage volumes

## 📊 Monitoring

### Lihat Logs

```bash
# Traefik logs
cd traefik-cloudflare
docker-compose logs -f traefik

# Portainer logs
cd portainer
docker-compose logs -f portainer
```

### Lihat Running Containers

```bash
docker ps
```

Anda akan lihat:
- `traefik` - Running di port 80 dan 8080
- `portainer` - Running di port 9000

## 🛑 Stop Services

```bash
# Stop Portainer
cd portainer
docker-compose down

# Stop Traefik
cd ../traefik-cloudflare
docker-compose down

# Hapus network (optional)
docker network rm proxy
```

## 🔄 Restart Services

```bash
# Restart Traefik
cd traefik-cloudflare
docker-compose restart

# Restart Portainer
cd portainer
docker-compose restart
```

## 🐛 Troubleshooting

### Port 80 sudah digunakan

```bash
# Cek apa yang pakai port 80
sudo lsof -i :80

# Stop service yang pakai port 80
# Atau ganti port di docker-compose.yml:
# ports:
#   - "8000:80"  # Ganti 80 jadi 8000
```

### Port 8080 sudah digunakan

```bash
# Cek apa yang pakai port 8080
sudo lsof -i :8080

# Atau ganti port di docker-compose.yml
```

### portainer.localhost tidak bisa diakses

**Option 1**: Edit `/etc/hosts`

```bash
sudo nano /etc/hosts
```

Tambahkan:
```
127.0.0.1 portainer.localhost
127.0.0.1 traefik.localhost
```

**Option 2**: Akses langsung via port

```
http://localhost:9000
```

### Docker daemon not running

```bash
# Start Docker Desktop
open -a Docker

# Tunggu sampai Docker icon di menu bar hijau
```

### Permission denied untuk Docker socket

```bash
# Tambahkan user ke docker group
sudo usermod -aG docker $USER

# Logout dan login lagi
```

## 📚 Next Steps

Setelah berhasil testing di localhost:

1. ✅ Explore Traefik dashboard
2. ✅ Explore Portainer features
3. ✅ Deploy container baru via Portainer
4. ✅ Lihat bagaimana Traefik auto-detect container baru
5. ✅ Belajar Traefik labels untuk routing

## 🎓 Menambah Service Baru

Contoh menambah Nginx via Portainer:

1. Buka Portainer → **Stacks** → **Add stack**
2. Nama: `nginx-test`
3. Paste YAML ini:

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx-test
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nginx.rule=Host(`nginx.localhost`)"
      - "traefik.http.routers.nginx.entrypoints=http"
      - "traefik.http.services.nginx.loadbalancer.server.port=80"

networks:
  proxy:
    external: true
```

4. Klik **Deploy the stack**
5. Akses http://nginx.localhost

## 💡 Tips

### 1. Lihat Logs Real-time

```bash
# Semua containers
docker-compose logs -f

# Specific container
docker logs -f traefik
docker logs -f portainer
```

### 2. Inspect Container

```bash
docker inspect traefik
docker inspect portainer
```

### 3. Access Container Shell

```bash
# Via Docker
docker exec -it traefik sh

# Via Portainer
Containers → Pilih container → Console → Connect
```

### 4. Clean Up

```bash
# Stop semua
docker-compose down

# Hapus unused images
docker image prune -f

# Hapus unused volumes
docker volume prune -f

# Hapus unused networks
docker network prune -f
```

## 📖 Struktur Folder

```
treafik/
├── traefik-cloudflare/
│   ├── docker-compose.yml    # Traefik config
│   ├── data/
│   │   ├── traefik.yml       # Main config
│   │   └── config.yml        # Middleware config
│   └── logs/                 # Traefik logs
│
├── portainer/
│   ├── docker-compose.yml    # Portainer config
│   └── data/                 # Portainer data
│
└── QUICKSTART-LOCALHOST.md   # File ini
```

## ⚙️ Konfigurasi

### Traefik

- **Image**: `traefik:latest`
- **Ports**: 80 (HTTP), 8080 (Dashboard)
- **Dashboard**: http://localhost:8080
- **Auth**: admin/admin

### Portainer

- **Image**: `portainer/portainer-ce:latest`
- **Port**: 9000
- **URL**: http://portainer.localhost atau http://localhost:9000

## 🔒 Security Notes

Ini adalah setup untuk **TESTING LOKAL** saja:

- ❌ Tidak ada SSL/HTTPS
- ❌ Password default (admin/admin)
- ❌ Debug mode enabled
- ❌ Tidak production-ready

Untuk production, lihat `README-PRODUCTION.md`.

## ✅ Checklist

- [ ] Docker Desktop running
- [ ] Network `proxy` created
- [ ] Traefik running
- [ ] Portainer running
- [ ] Traefik dashboard accessible
- [ ] Portainer accessible
- [ ] Portainer admin user created

---

**Happy Testing!** 🚀

Jika ada masalah, cek section **Troubleshooting** di atas.
