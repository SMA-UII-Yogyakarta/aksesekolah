# Deployment & Infrastructure

## Infrastructure Architecture

### High-Level Diagram

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│   Browser    │────>│  VPS Server  │────>│  NeonDB          │
│  (students,  │     │  (Nginx/     │     │  (PostgreSQL 16) │
│   teachers,  │     │   PHP-FPM)   │     │  serverless      │
│   etc.)      │     │  Laravel     │     └──────────────────┘
└──────────────┘     │              │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │   Object     │
                     │   Storage    │
                     │  (S3 Proto)  │
                     └──────────────┘
```

### Minimum VPS Specifications

| Component | Specification | Notes |
|---|---|---|
| CPU | Minimum 4 Core | To handle 750+ CCU |
| RAM | Minimum 8 GB | Laravel + PostgreSQL is quite heavy |
| Storage | 100 GB SSD/NVMe | I/O speed critical for attendance |
| Bandwidth | 1 Gbps | Peak traffic 06:30-07:00 |
| OS | Ubuntu 24.04 LTS | — |

### Production Server Stack

| Layer | Technology | Reason |
|---|---|---|
| **Web Server** | Nginx + PHP-FPM | Lighter than Apache for production |
| **PHP** | 8.4+ | Stable, compatible with Laravel 13 |
| **Database** | PostgreSQL 16 via NeonDB | Serverless, auto-scaling, branching, built-in PgBouncer |
| **Cache** | Redis | For session, cache, queue |
| **Queue** | Redis + Laravel Queue | For background jobs (photo upload, etc.) |
| **Object Storage** | RustFS (Dev/Demo) / Cloudflare R2 / AWS S3 | S3-compatible, ultra-lightweight |

### Ekosistem Digital Terpadu SMA UII Yogyakarta (Central IdP & Master Data)

`smauii-core` bertindak sebagai Central Identity Provider (IdP) & Dapodik master data hub bagi seluruh platform digital sekolah:

| Platform | Subdomain | Stack / Engine | Integrasi Data Master |
|---|---|---|---|
| **SMART Absen (Core)** | `preview.smauiiyk.sch.id` / `app.smauiiyk.sch.id` | Laravel 13 + React Inertia | Pusat data akun pengguna, role RBAC, rombel, & presensi |
| **Moodle E-Learning** | `elearning.smauiiyk.sch.id` | Moodle (Container host) | SSO akun siswa/guru & sinkronisasi peserta kursus/mapel |
| **SLiMS Perpustakaan** | `library.smauiiyk.sch.id` | SLiMS (Container host) | Sinkronisasi data keanggotaan perpustakaan (NIS/NIP) |
| **Digital Lab & Research**| `lab.smauiiyk.sch.id` | Lab Management Portal | Autentikasi workstation & jadwal praktikum sains/TIK |

---

## Server Configuration

### Nginx Virtual Host

```nginx
server {
    listen 80;
    server_name absen.smauii.sch.id;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name absen.smauii.sch.id;

    root /var/www/smauii-core/public;
    index index.php;

    ssl_certificate     /etc/ssl/certs/smauii.crt;
    ssl_certificate_key /etc/ssl/private/smauii.key;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(ht|env) {
        deny all;
    }
}
```

### PostgreSQL Tuning (Production)

NeonDB handles tuning server-side automatically. For self-managed PostgreSQL:

```ini
# postgresql.conf
max_connections = 200
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 64MB
maintenance_work_mem = 256MB
random_page_cost = 1.1          # SSD
effective_io_concurrency = 200  # SSD
wal_buffers = 16MB
```

### PHP-FPM Tuning

```ini
[www]
pm = dynamic
pm.max_children = 150
pm.start_servers = 20
pm.min_spare_servers = 10
pm.max_spare_servers = 40
pm.max_requests = 500
```

---

## Object Storage

### Architecture

```
[Laravel App]
    │
    ├── Upload file → S3 Driver (Laravel Filesystem)
    │       │
    │       ▼
    │   [Object Storage]
    │       ├── attendances/{year}/{month}/{nis}_{timestamp}.jpg
    │       └── leave_requests/{year}/{month}/{id_leave}_{document}.pdf
    │
    └── Save URL to database (photo_url, evidence_url)
```

### Object Storage Directory Structure

```
bucket-smauii/
├── attendances/
│   ├── 2026/
│   │   ├── 06/
│   │   │   ├── 12345_20260622_064512.jpg
│   │   │   ├── 12346_20260622_064530.jpg
│   │   │   └── ...
│   │   └── ...
│   └── ...
├── leave_requests/
│   ├── 2026/
│   │   ├── 06/
│   │   │   ├── IZ001_doctor_letter.pdf
│   │   │   └── ...
│   │   └── ...
│   └── ...
└── temp/  ← for temporary files before validation
```

### Provider Recommendations

| Provider | Role / Environment | Price | Notes |
|---|---|---|---|
| **RustFS (self-hosted)** | **Development (Lerd) & Preview Demo** | Free (Open Source) | **Primary/Wajib** untuk dev & preview demo — native kompatibel dengan `lerd`, ultra-ringan (<20MB RAM), performa tinggi berbasis Rust |
| **MinIO (self-hosted)** | **Development (Alternative)** | Free | Alternatif opsional untuk on-premise/local jika developer sudah memiliki MinIO |
| **Cloudflare R2** | **Production Cloud Target** | ~$0.015/GB/month | **Diutamakan** — S3 API kompatibel, zero egress fee, terhubung Cloudflare CDN |
| **AWS S3** | **Production Cloud Target** | Pay-as-you-go | Standar industri high durability (99.999999999%) + CloudFront |
| **Wasabi / Backblaze B2** | **Production Cloud Alternative** | ~$6-$7/TB/month | Opsi cloud storage ekonomis |

### Laravel Integration

```php
// config/filesystems.php
'disks' => [
    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'bucket' => env('AWS_BUCKET'),
        'url' => env('AWS_URL'),
        'endpoint' => env('AWS_ENDPOINT'), // Penting untuk RustFS/MinIO/R2
        'use_path_style_endpoint' => filter_var(env('AWS_USE_PATH_STYLE_ENDPOINT', false), FILTER_VALIDATE_BOOLEAN),
    ],
],

// Upload example via StorageService
$path = $request->file('photo')->storeAs(
    'attendances/' . now()->format('Y/m'),
    $student->nis . '_' . now()->format('Ymd_His') . '.jpg',
    's3'
);
```

---

## Environment Variables (Production)

```env
APP_NAME="SMART Absen SMA UII"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://absen.smauii.sch.id

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=smauii_core
DB_USERNAME=smauii_user
DB_PASSWORD=********

SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
CACHE_STORE=redis

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

FILESYSTEM_DISK=s3

AWS_ACCESS_KEY_ID=********
AWS_SECRET_ACCESS_KEY=********
AWS_DEFAULT_REGION=auto
AWS_BUCKET=smauii-presensi
AWS_ENDPOINT=https://s3.wasabisys.com
AWS_URL=https://smauii-presensi.s3.wasabisys.com
```

---

## Deployment Checklist

- [ ] VPS has Ubuntu 24.04 LTS installed
- [ ] Nginx + PHP 8.4 + PostgreSQL 16 installed
- [ ] Redis installed and running
- [ ] SSL certificate active (Let's Encrypt / manual)
- [ ] Domain `absen.smauii.sch.id` points to VPS IP
- [ ] Object Storage bucket created and configured
- [ ] Production environment variables set
- [ ] `APP_DEBUG=false` and `APP_ENV=production`
- [ ] `APP_KEY` generated
- [ ] Database migrations already run
- [ ] Queue worker running: `php artisan queue:work`
- [ ] Cache optimized: `php artisan optimize`
- [ ] Storage link created: `php artisan storage:link`
- [ ] Database backup configured (cron job)
- [ ] Server monitoring (uptime, CPU, RAM, disk) active
