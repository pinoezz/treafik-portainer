# 🚀 Traefik dengan Let's Encrypt (Cloudflare DNS)

Setup production-ready Traefik reverse proxy dengan SSL certificate otomatis dari Let's Encrypt menggunakan Cloudflare DNS Challenge.

## 📋 Fitur

- ✅ **Auto SSL Certificate** - Let's Encrypt dengan wildcard support
- ✅ **Cloudflare DNS Challenge** - Tidak perlu expose port 80/443 untuk validasi
- ✅ **Auto HTTPS Redirect** - Semua HTTP traffic otomatis redirect ke HTTPS
- ✅ **Security Headers** - HSTS, XSS Protection, dll
- ✅ **Rate Limiting** - Proteksi dari abuse
- ✅ **Dashboard GUI** - Monitoring visual
- ✅ **Docker Auto-discovery** - Deteksi container otomatis

## 🔧 Prerequisites

1. **Domain** - Domain yang sudah pointing ke Cloudflare
2. **Cloudflare Account** - Untuk DNS management
3. **Docker & Docker Compose** - Terinstall di server
4. **Server dengan IP Public** - Domain harus pointing ke IP server ini

## 📁 Struktur Folder

```
traefik-cloudflare/
├── docker-compose.yml          # Konfigurasi Traefik
├── .env                        # Environment variables (BUAT SENDIRI)
├── .env.example               # Template environment variables
├── data/
│   ├── traefik.yml           # Konfigurasi utama Traefik
│   ├── config.yml            # Security middleware
│   └── acme.json             # SSL certificates (auto-generated)
├── logs/                      # Traefik logs (auto-generated)
└── README.md                  # File ini
```

## 🚀 Cara Setup

### Step 1: Dapatkan Cloudflare API Token

1. Login ke [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Klik profile icon → **My Profile** → **API Tokens**
3. Klik **Create Token**
4. Gunakan template **Edit zone DNS** atau buat custom dengan permissions:
   - **Zone** → **DNS** → **Edit**
   - **Zone Resources** → **Include** → **Specific zone** → Pilih domain Anda
5. Copy token yang dihasilkan

### Step 2: Setup Environment Variables

```bash
# Copy template
cp .env.example .env

# Edit dengan credentials Anda
nano .env
```

Isi file `.env`:

```env
DOMAIN=yourdomain.com
CF_API_EMAIL=your-email@example.com
CF_DNS_API_TOKEN=your-cloudflare-dns-api-token
TRAEFIK_DASHBOARD_USERS=admin:$$apr1$$8EVjn/nj$$GiLUZqcbueTFeD23SuB6x0
```

### Step 3: Generate Password untuk Dashboard

```bash
# Install htpasswd (jika belum ada)
# macOS:
brew install apache2-utils

# Linux:
sudo apt-get install apache2-utils

# Generate password
htpasswd -nb admin your-password-here
```

Copy output dan paste ke `.env` di variable `TRAEFIK_DASHBOARD_USERS`

**PENTING**: Escape semua `$` dengan `$$` di file `.env`!

Contoh:
```
# Output dari htpasswd:
admin:$apr1$8EVjn/nj$GiLUZqcbueTFeD23SuB6x0

# Di .env harus jadi:
TRAEFIK_DASHBOARD_USERS=admin:$$apr1$$8EVjn/nj$$GiLUZqcbueTFeD23SuB6x0
```

### Step 4: Setup Permissions

```bash
# Buat file acme.json dengan permission yang benar
touch data/acme.json
chmod 600 data/acme.json

# Buat folder logs
mkdir -p logs
```

### Step 5: Setup DNS di Cloudflare

Tambahkan A record di Cloudflare DNS:

```
Type: A
Name: @
Content: YOUR_SERVER_IP
Proxy: Disabled (DNS only) - PENTING!

Type: A
Name: *
Content: YOUR_SERVER_IP
Proxy: Disabled (DNS only) - PENTING!

Type: A
Name: traefik
Content: YOUR_SERVER_IP
Proxy: Disabled (DNS only) - PENTING!
```

**PENTING**: Set proxy ke **DNS only** (grey cloud), bukan **Proxied** (orange cloud)!

### Step 6: Jalankan Traefik

```bash
# Buat network proxy
docker network create proxy

# Start Traefik
docker-compose up -d

# Lihat logs
docker-compose logs -f
```

### Step 7: Verifikasi

Tunggu beberapa menit untuk SSL certificate generation, lalu akses:

```
https://traefik.yourdomain.com
```

Login dengan username dan password yang Anda set di Step 3.

## 🔍 Monitoring

### Lihat Logs

```bash
# Real-time logs
docker-compose logs -f

# Logs dari file
tail -f logs/traefik.log
tail -f logs/access.log
```

### Check SSL Certificate

```bash
# Lihat isi acme.json
cat data/acme.json | jq

# Test SSL dengan curl
curl -vI https://traefik.yourdomain.com
```

### Dashboard

Akses dashboard di: `https://traefik.yourdomain.com`

Di dashboard Anda bisa lihat:
- Semua routers dan services
- SSL certificates
- Middlewares
- Real-time metrics

## 🔒 Security Features

### 1. Let's Encrypt SSL
- Wildcard certificate support (`*.yourdomain.com`)
- Auto-renewal setiap 60 hari
- DNS Challenge (tidak perlu expose port untuk validation)

### 2. Security Headers
- **HSTS** - Force HTTPS untuk 2 tahun
- **X-Frame-Options** - Prevent clickjacking
- **X-Content-Type-Options** - Prevent MIME sniffing
- **X-XSS-Protection** - XSS filter
- **Referrer-Policy** - Control referrer information
- **Permissions-Policy** - Disable unnecessary browser features

### 3. TLS Configuration
- Minimum TLS 1.2
- Strong cipher suites only
- SNI strict mode

### 4. Rate Limiting
- 100 requests per second average
- Burst up to 50 requests

### 5. Container Security
- `no-new-privileges:true` - Prevent privilege escalation
- Read-only volumes where possible
- Non-root user (Traefik default)

## 🐛 Troubleshooting

### SSL Certificate tidak ter-generate

```bash
# Check logs
docker-compose logs traefik | grep acme

# Pastikan:
# 1. CF_DNS_API_TOKEN benar
# 2. DNS sudah pointing ke server
# 3. Cloudflare proxy DISABLED (grey cloud)
# 4. acme.json permission 600
```

### Dashboard tidak bisa diakses

```bash
# Check DNS
nslookup traefik.yourdomain.com

# Check Traefik running
docker ps | grep traefik

# Check logs
docker-compose logs traefik
```

### Error: "unable to generate a certificate"

Kemungkinan penyebab:
1. Cloudflare API token tidak valid atau expired
2. DNS belum propagate (tunggu 5-10 menit)
3. Rate limit Let's Encrypt (max 5 failures per hour)

### Error: "permission denied" untuk acme.json

```bash
chmod 600 data/acme.json
```

## 📚 Menambah Service Baru

Untuk menambah service baru yang menggunakan Traefik:

1. Tambahkan service ke network `proxy`
2. Tambahkan labels Traefik

Contoh:

```yaml
services:
  myapp:
    image: myapp:latest
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`myapp.${DOMAIN}`)"
      - "traefik.http.routers.myapp.entrypoints=https"
      - "traefik.http.routers.myapp.tls=true"
      - "traefik.http.routers.myapp.tls.certresolver=cloudflare"
      - "traefik.http.services.myapp.loadbalancer.server.port=8080"

networks:
  proxy:
    external: true
```

## 🔄 Update Traefik

```bash
# Pull image terbaru
docker-compose pull

# Restart dengan image baru
docker-compose up -d

# Cleanup old images
docker image prune -f
```

## 🗑️ Uninstall

```bash
# Stop dan hapus containers
docker-compose down

# Hapus network (jika tidak ada service lain)
docker network rm proxy

# Hapus data (HATI-HATI!)
rm -rf data/acme.json logs/
```

## 📖 Resources

- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Cloudflare API Tokens](https://developers.cloudflare.com/api/tokens/create)
- [Let's Encrypt Rate Limits](https://letsencrypt.org/docs/rate-limits/)

## ⚠️ Important Notes

1. **Cloudflare Proxy**: HARUS disabled (grey cloud) untuk DNS records
2. **API Token**: Gunakan DNS API Token, bukan Global API Key
3. **Rate Limits**: Let's Encrypt punya rate limits, jangan terlalu sering test
4. **Backup**: Backup `acme.json` secara berkala
5. **Security**: Ganti password default untuk dashboard!

---

**Setup by DevOps Team** 🚀
