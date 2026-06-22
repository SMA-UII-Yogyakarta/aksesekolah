# Deployment & Infrastruktur

## Arsitektur Infrastruktur

### Diagram High-Level

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Browser    │────>│  VPS Server  │────>│  MySQL 8.0   │
│  (siswa,     │     │  (Apache/    │     │  (Laravel    │
│   guru, dll) │     │   Nginx)     │     │   App)       │
└──────────────┘     │  Laravel     │     └──────────────┘
                     │              │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │   Object     │
                     │   Storage    │
                     │  (S3 Proto)  │
                     └──────────────┘
```

### Spesifikasi VPS Minimum

| Komponen | Spesifikasi | Catatan |
|---|---|---|
| CPU | Minimal 4 Core | Untuk menangani 750+ CCU |
| RAM | Minimal 8 GB | Laravel + MySQL cukup berat |
| Storage | 100 GB SSD/NVMe | Kecepatan I/O kritis untuk presensi |
| Bandwidth | 1 Gbps | Peak traffic 06:30-07:00 |
| OS | Ubuntu 24.04 LTS | — |

### Stack Server Produksi

| Layer | Teknologi | Alasan |
|---|---|---|
| **Web Server** | Nginx + PHP-FPM | Lebih ringan dari Apache untuk production |
| **PHP** | 8.3+ (LTS) | Stabil, kompatibel dengan Laravel |
| **Database** | MySQL 8.0 / MariaDB 11 | — |
| **Cache** | Redis | Untuk session, cache, queue |
| **Queue** | Redis + Laravel Queue | Untuk background job (upload foto, dll) |
| **Object Storage** | MinIO / Wasabi / Backblaze B2 | S3-compatible |

---

## Konfigurasi Server

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

### Tuning MySQL (max_connections)

```ini
[mysqld]
max_connections = 1000
innodb_buffer_pool_size = 2G
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2
query_cache_type = 0
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

### Arsitektur

```
[Laravel App]
    │
    ├── Upload file → S3 Driver (Laravel Filesystem)
    │       │
    │       ▼
    │   [Object Storage]
    │       ├── presensi/{tahun}/{bulan}/{nis}_{timestamp}.jpg
    │       └── izin/{tahun}/{bulan}/{id_izin}_{dokumen}.pdf
    │
    └── Simpan URL ke database (foto_url, bukti_dokumen_url)
```

### Struktur Direktori Object Storage

```
bucket-smauii/
├── presensi/
│   ├── 2026/
│   │   ├── 06/
│   │   │   ├── 12345_20260622_064512.jpg
│   │   │   ├── 12346_20260622_064530.jpg
│   │   │   └── ...
│   │   └── ...
│   └── ...
├── izin/
│   ├── 2026/
│   │   ├── 06/
│   │   │   ├── IZ001_surat_dokter.pdf
│   │   │   └── ...
│   │   └── ...
│   └── ...
└── temp/  ← untuk file sementara sebelum divalidasi
```

### Provider Recommendations

| Provider | Harga | Kapasitas | Keterangan |
|---|---|---|---|
| **Wasabi** | ~$7/TB/bulan | Unlimited | Hot storage, tidak ada biaya egress |
| **Backblaze B2** | ~$6/TB/bulan | Unlimited | Ekonomis, ada biaya download |
| **Cloudflare R2** | ~$0.015/GB/bulan | Unlimited | Tidak ada biaya egress |
| **MinIO (self-hosted)** | Gratis | Tergantung server | Untuk on-premise |

### Integrasi Laravel

```php
// config/filesystems.php
'disks' => [
    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'url' => env('AWS_URL'),
        'endpoint' => env('AWS_ENDPOINT'), // Penting untuk MinIO/S3-compatible
        'use_path_style_endpoint' => true,
    ],
],

// Contoh upload
$path = $request->file('foto')->storeAs(
    'presensi/' . now()->format('Y/m'),
    $siswa->nis . '_' . now()->format('Ymd_His') . '.jpg',
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

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
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

## Checklist Deployment

- [ ] VPS sudah terinstall Ubuntu 24.04 LTS
- [ ] Nginx + PHP 8.3 + MySQL 8.0 sudah terinstall
- [ ] Redis sudah terinstall dan berjalan
- [ ] SSL certificate sudah aktif (Let's Encrypt / manual)
- [ ] Domain `absen.smauii.sch.id` sudah mengarah ke IP VPS
- [ ] Object Storage bucket sudah dibuat dan terkonfigurasi
- [ ] Environment variables production sudah diset
- [ ] `APP_DEBUG=false` dan `APP_ENV=production`
- [ ] `APP_KEY` sudah digenerate
- [ ] Migrasi database sudah dijalankan
- [ ] Queue worker sudah berjalan: `php artisan queue:work`
- [ ] Cache sudah dioptimasi: `php artisan optimize`
- [ ] Storage link sudah dibuat: `php artisan storage:link`
- [ ] Backup database sudah dikonfigurasi (cron job)
- [ ] Monitoring server (uptime, CPU, RAM, disk) sudah aktif
