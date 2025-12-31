# Production Setup Guide - Skibidoo E-Commerce

**Version:** 1.0.0  
**Letzte Aktualisierung:** 31. Dezember 2025  
**Geschätzte Setup-Zeit:** 2-4 Stunden

---

## Server-Anforderungen

### Minimum (< 1.000 Bestellungen/Monat)
- CPU: 2 vCPUs
- RAM: 4 GB
- Storage: 50 GB SSD
- OS: Ubuntu 22.04 LTS / Debian 12

### Empfohlen (< 10.000 Bestellungen/Monat)
- CPU: 4 vCPUs
- RAM: 8 GB
- Storage: 100 GB SSD
- Bandwidth: 1 TB/Monat

### Enterprise (> 10.000 Bestellungen/Monat)
- CPU: 8+ vCPUs
- RAM: 16+ GB
- Storage: 200+ GB SSD
- Load Balancer + CDN erforderlich

---

## Quick Start (15 Minuten)

```bash
# 1. Server vorbereiten
sudo apt update && sudo apt upgrade -y

# 2. Bun installieren
curl -fsSL https://bun.sh/install | bash

# 3. PostgreSQL & Redis installieren
sudo apt install -y postgresql-16 redis-server

# 4. Repository clonen
cd /opt
git clone https://github.com/casoon/skibidoo-core.git
cd skibidoo-core

# 5. Dependencies installieren
bun install

# 6. Environment konfigurieren
cp .env.example .env.production
nano .env.production  # Anpassen

# 7. Datenbank migrieren
bun run db:migrate

# 8. Service starten
sudo systemctl start skibidoo-core
```

---

## Detaillierte Anleitung

### 1. Server-Setup

```bash
# Essentials installieren
sudo apt install -y curl wget git build-essential nginx certbot

# Deploy-User erstellen
sudo useradd -m -s /bin/bash deploy
sudo usermod -aG sudo deploy
sudo su - deploy
```

### 2. Bun Runtime installieren

```bash
curl -fsSL https://bun.sh/install | bash
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
bun --version
```

### 3. PostgreSQL 16 Setup

```bash
# PostgreSQL installieren
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc
sudo apt update
sudo apt install -y postgresql-16

# Datenbank erstellen
sudo -u postgres psql << EOF
CREATE DATABASE skibidoo_production;
CREATE USER skibidoo WITH PASSWORD 'CHANGE_ME';
GRANT ALL PRIVILEGES ON DATABASE skibidoo_production TO skibidoo;
EOF
```

**Performance-Tuning** (`/etc/postgresql/16/main/postgresql.conf`):

```ini
shared_buffers = 2GB               # 25% of RAM
effective_cache_size = 6GB         # 75% of RAM
maintenance_work_mem = 512MB
work_mem = 64MB
max_connections = 100

wal_buffers = 16MB
checkpoint_completion_target = 0.9
random_page_cost = 1.1             # For SSD
effective_io_concurrency = 200
```

```bash
sudo systemctl restart postgresql
```

### 4. Redis Setup

```bash
# Redis installieren
sudo apt install -y redis-server

# Konfigurieren
sudo nano /etc/redis/redis.conf
```

**Änderungen:**
```ini
bind 127.0.0.1
requirepass YOUR_STRONG_REDIS_PASSWORD
maxmemory 512mb
maxmemory-policy allkeys-lru
```

```bash
sudo systemctl restart redis-server
sudo systemctl enable redis-server
```

### 5. MinIO (S3-kompatibel) installieren

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
sudo mkdir -p /data/minio
sudo chown deploy:deploy /data/minio
```

**Systemd Service** (`/etc/systemd/system/minio.service`):

```ini
[Unit]
Description=MinIO
After=network.target

[Service]
Type=simple
User=deploy
Environment="MINIO_ROOT_USER=admin"
Environment="MINIO_ROOT_PASSWORD=CHANGE_ME"
ExecStart=/usr/local/bin/minio server /data/minio --console-address ":9001"

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start minio
sudo systemctl enable minio
```

### 6. Nginx + SSL

**Nginx Config** (`/etc/nginx/sites-available/skibidoo`):

```nginx
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# SSL-Zertifikat holen
sudo certbot --nginx -d api.example.com

# Config aktivieren
sudo ln -s /etc/nginx/sites-available/skibidoo /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 7. Skibidoo Core deployen

```bash
cd /opt
sudo git clone https://github.com/casoon/skibidoo-core.git
sudo chown -R deploy:deploy skibidoo-core
cd skibidoo-core
```

**Environment** (`.env.production`):

```bash
# Database
DATABASE_URL="postgresql://skibidoo:PASSWORD@localhost:5432/skibidoo_production"

# Redis
REDIS_URL="redis://:PASSWORD@localhost:6379"

# S3 Storage
S3_ENDPOINT="http://localhost:9000"
S3_ACCESS_KEY_ID="admin"
S3_SECRET_ACCESS_KEY="PASSWORD"
S3_BUCKET="skibidoo-uploads"

# JWT (generate with: openssl rand -hex 32)
JWT_SECRET="your-jwt-secret-here"
JWT_REFRESH_SECRET="your-refresh-secret-here"

# Stripe
STRIPE_SECRET_KEY="sk_live_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# PayPal
PAYPAL_CLIENT_ID="..."
PAYPAL_CLIENT_SECRET="..."
PAYPAL_MODE="live"

# Email
SMTP_HOST="smtp.sendgrid.net"
SMTP_PORT="587"
SMTP_USER="apikey"
SMTP_PASS="SG...."
EMAIL_FROM="noreply@example.com"

# App
NODE_ENV="production"
PORT="3000"
CORS_ORIGIN="https://shop.example.com"
```

**Datenbank migrieren:**

```bash
bun install
bun run db:migrate
```

**Systemd Service** (`/etc/systemd/system/skibidoo-core.service`):

```ini
[Unit]
Description=Skibidoo Core API
After=network.target postgresql.service redis.service

[Service]
Type=simple
User=deploy
WorkingDirectory=/opt/skibidoo-core
Environment="NODE_ENV=production"
EnvironmentFile=/opt/skibidoo-core/.env.production
ExecStart=/home/deploy/.bun/bin/bun run src/index.ts
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start skibidoo-core
sudo systemctl enable skibidoo-core
sudo systemctl status skibidoo-core
```

### 8. Frontends deployen (Cloudflare Pages)

```bash
# Admin
cd ~/skibidoo-admin
bun run build
npx wrangler pages deploy dist --project-name=skibidoo-admin

# Storefront
cd ~/skibidoo-storefront
bun run build
npx wrangler pages deploy dist --project-name=skibidoo-storefront
```

**Environment Variables (Cloudflare Dashboard):**
- `API_URL`: https://api.example.com

### 9. Monitoring (Prometheus + Grafana)

**Prometheus** (`/opt/prometheus/prometheus.yml`):

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'skibidoo-core'
    static_configs:
      - targets: ['localhost:3000']
    metrics_path: '/metrics'
```

**Grafana installieren:**

```bash
sudo apt install -y grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Zugriff: http://server-ip:3000 (admin/admin)

### 10. Backup-Strategie

**PostgreSQL Backup** (`/opt/backup-db.sh`):

```bash
#!/bin/bash
BACKUP_DIR="/backups/postgres"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
pg_dump -U skibidoo skibidoo_production | gzip > $BACKUP_DIR/skibidoo_$DATE.sql.gz
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
```

**Crontab:**

```bash
crontab -e
# Add:
0 2 * * * /opt/backup-db.sh
```

### 11. Sicherheits-Checkliste

- [ ] Firewall konfiguriert (UFW: nur 22, 80, 443)
- [ ] SSH Key-only Auth
- [ ] Starke Passwörter (>32 chars)
- [ ] SSL-Zertifikate aktiv
- [ ] CORS origins beschränkt
- [ ] Fail2Ban installiert
- [ ] Backups automatisiert
- [ ] Monitoring aktiv

---

## Health Checks

```bash
# Core API
curl https://api.example.com/health
# Expected: {"status":"ok"}

# Database
psql -U skibidoo -d skibidoo_production -c "SELECT version();"

# Redis
redis-cli -a PASSWORD ping
# Expected: PONG
```

---

## Troubleshooting

### Core API startet nicht

```bash
sudo journalctl -u skibidoo-core -n 100
sudo netstat -tuln | grep 3000
```

### Database-Verbindungsfehler

```bash
sudo systemctl status postgresql
sudo -u postgres psql -c "SELECT count(*) FROM pg_stat_activity;"
```

### Hohe Last

```bash
# CPU/Memory prüfen
htop

# Slow Queries
sudo -u postgres psql skibidoo_production -c "SELECT query, calls, total_time FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"
```

---

**Support:** https://github.com/casoon/skibidoo-docs/issues
