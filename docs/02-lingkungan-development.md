# Lingkungan Development

## Ringkasan

Seluruh pengembangan **SMART Absen SMA UII** dilakukan di atas **Laragon 6.0.0** — sebuah *local development environment* untuk Windows yang menyediakan stack web server terintegrasi.

> **Catatan Penting:** Semua developer diwajibkan menggunakan Laragon agar lingkungan kerja seragam. Penggunaan XAMPP, WAMP, atau tool lain sangat tidak disarankan karena perbedaan konfigurasi dan path.

---

## Spesifikasi Lingkungan Terkini

### Laragon

| Item | Nilai |
|---|---|
| Versi | 6.0.0.2 |
| Instalasi | `C:\laragon` |
| Data | `C:\laragon\data` |
| Document Root | `C:\laragon\www` |
| Run at Startup | Ya |
| Auto VirtualHosts | Aktif (semua folder `.test` otomatis) |
| SSL | Aktif (port 443) |

### PHP

| Item | Nilai |
|---|---|
| **Versi Aktif** | **8.4.22** (NTS, VC++ 2022 x64) |
| Path | `C:\laragon\bin\php\php-8.4.22-nts-Win32-vs17-x64\` |
| Konfigurasi | `C:\laragon\bin\php\php-8.4.22-nts-Win32-vs17-x64\php.ini` |
| `memory_limit` | 512M |
| `max_execution_time` | 36000 (10 jam) |
| `post_max_size` | 2G |
| `upload_max_filesize` | 2G |
| `date.timezone` | **Asia/Jakarta** ✅ |
| Ekstensi Aktif | curl, gd, intl, mbstring, mysqli, pdo_mysql, **pdo_pgsql**, **pgsql**, zip, openssl, xsl |

**PHP Lain yang Tersedia:**

| Versi | Path | Xdebug |
|---|---|---|
| 8.2.27 | `C:\laragon\bin\php\php-8.2.27-nts-Win32-vs16-x64\` | ❌ |
| 8.1.31 | `C:\laragon\bin\php\php-8.1.31-nts-Win32-vs16-x64\` | ✅ 3.4.1 |
| 8.1.10 | `C:\laragon\bin\php\php-8.1.10-Win32-vs16-x64\` | ❌ |

> **Catatan Xdebug:** PHP 8.4.22 belum memiliki ekstensi Xdebug yang kompatibel. Untuk kebutuhan debugging, gunakan PHP 8.1.31 yang dapat dipilih melalui menu Laragon > PHP > Versi.

### Web Server

| Item | Nilai |
|---|---|
| **Server Aktif** | **Apache 2.4.54** (mod_fcgid) |
| Path Apache | `C:\laragon\bin\apache\httpd-2.4.54-win64-VS16\` |
| PHP Handler | FastCGI (mod_fcgid) via `C:\laragon\etc\apache2\fcgid.conf` |
| DocumentRoot | `C:\laragon\www` |
| Nginx (opsional) | 1.22.0 (tidak aktif) |

### Database

| Item | Nilai |
|---|---|
| **Database Aktif** | **PostgreSQL 16** — via NeonDB (production) / Laragon (local) |
| Path (Laragon) | `C:\laragon\bin\postgresql\postgresql-16.x-winx64\` (install manual) |
| Port | 5432 |
| Root User | `postgres` (tanpa password lokal) |
| NeonDB Production | `ep-xxx-xxxx.us-east-2.aws.neon.tech` (serverless) |
| NeonDB Branching | Tiap fitur/PR bisa punya DB branch sendiri |

**Catatan:** MySQL 8.0.30 masih tersedia di Laragon (`C:\laragon\bin\mysql\mysql-8.0.30-winx64\`, port 3306) untuk kompatibilitas, namun tidak digunakan oleh proyek ini. Seluruh environment menggunakan PostgreSQL.

### Instalasi PostgreSQL di Laragon

1. Download PostgreSQL zip dari [EnterpriseDB](https://www.enterprisedb.com/download-postgresql-binaries) atau [PostgreSQL Windows](https://www.postgresql.org/download/windows/)
2. Extract ke `C:\laragon\bin\postgresql\postgresql-16.x-winx64\`
3. Tambahkan service PostgreSQL via Laragon: Menu > Tools > Service/Port > PostgreSQL
4. Inisialisasi database cluster:
   ```bash
   "C:\laragon\bin\postgresql\postgresql-16.x-winx64\bin\initdb.exe" -D "C:/laragon/data/postgresql-16" --username=postgres
   ```
5. Jalankan PostgreSQL dari Laragon

### Tools Lain

| Tool | Versi | Path |
|---|---|---|---|
| Node.js | 22.14.0 | `C:\laragon\bin\nodejs\node-v22.14.0-win-x64\` |
| **Bun** | **latest** | `C:\Users\<user>\.bun\bin\bun.exe` (install: `powershell -c "irm bun.sh/install.ps1 | iex"`) |
| Composer | latest (portable) | `C:\laragon\bin\composer\composer.phar` |
| Redis | 5.0.14.1 | `C:\laragon\bin\redis\redis-x64-5.0.14.1\` |
| Memcached | 1.6.8 | `C:\laragon\bin\memcached\memcached-1.6.8-win64-mingw\` |
| pgAdmin / DBeaver | latest | GUI tools untuk PostgreSQL |
| Git | (via Windows PATH) | `C:\Program Files\Git\cmd\git.EXE` |
| GitHub CLI | 2.95.0 | `gh` |

> **Catatan:** Gunakan **Bun** untuk package management frontend (bukan npm). Bun lebih cepat, built-in TypeScript support, dan kompatibel dengan Vite 8.

---

## Proyek yang Ada di Document Root

Semua proyek di `C:\laragon\www` otomatis mendapatkan virtual host dengan domain `.test`:

| Domain | Folder | Deskripsi |
|---|---|---|
| `smauii-core.test` | `smauii-core` | Backend Laravel SMART Absen |
| `smauii-aksesekolah.test` | `smauii-aksesekolah` | Monorepo entrypoint (repositori ini) |

---

## Setup Lingkungan untuk Developer Baru

### 1. Instal Laragon

1. Unduh Laragon dari [https://laragon.org/download](https://laragon.org/download) (versi **Full** direkomendasikan)
2. Install ke `C:\laragon` (pastikan tidak ada spasi di path)
3. Jalankan Laragon, pastikan Apache + PostgreSQL menyala (tombol **Start All**)

### 2. Clone Repositori

```bash
# Clone monorepo
cd C:\laragon\www
git clone git@github.com:SMA-UII-Yogyakarta/aksesekolah.git smauii-aksesekolah

# Clone backend core (untuk development harian)
git clone git@github.com:SMA-UII-Yogyakarta/core.git smauii-core
```

### 3. Setup Bun (Package Manager Frontend)

```bash
# Install Bun (Windows PowerShell)
powershell -c "irm bun.sh/install.ps1 | iex"

# Verifikasi
bun --version
```

### 4. Setup Backend (core)

```bash
cd C:\laragon\www\smauii-core

# Install PHP dependencies
composer install

# Install JS/TS dependencies (via Bun, bukan npm!)
bun install

# Copy environment
cp .env.example .env
# (sesuaikan konfigurasi database, lihat .env di bawah)

# Generate APP_KEY
php artisan key:generate

# Setup database
# Buat database smauii_core via pgAdmin / DBeaver, atau:
php artisan db:create  # (jika pakai package db-tools)

# Jalankan migrasi
php artisan migrate --seed

# Install Laravel Sanctum (token management)
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

# Install Spatie Laravel Permission (RBAC)
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

### 5. Inisialisasi Submodule Monorepo

```bash
cd C:\laragon\www\smauii-aksesekolah

# Init repositori
git init
git remote add origin git@github.com:SMA-UII-Yogyakarta/aksesekolah.git

# Jika submodule backend sudah terdaftar:
git submodule update --init --recursive
```

### 6. Konfigurasi .env (Backend)

```env
APP_NAME="SMAUII Core"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://smauii-core.test

# Local development (PostgreSQL via Laragon)
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=smauii_core
DB_USERNAME=postgres
DB_PASSWORD=

# Production (NeonDB) — uncomment saat deploy
# DB_HOST=ep-xxx-xxxx.us-east-2.aws.neon.tech
# DB_PORT=5432
# DB_DATABASE=smauii_production
# DB_USERNAME=smauii_owner
# DB_PASSWORD=xxxxxx
# DB_SSLMODE=require

SESSION_DRIVER=database
QUEUE_CONNECTION=database
CACHE_STORE=database
```

### 7. Konfigurasi Virtual Host

Laragon akan otomatis membuat virtual host `smauii-core.test` saat folder `smauii-core` terdeteksi.
Namun, Laravel memerlukan `DocumentRoot` mengarah ke `public/`. Jika terjadi *directory listing*,
sesuaikan file di:

```
C:\laragon\etc\apache2\sites-enabled\auto.smauii-core.test.conf
```

Pastikan `DocumentRoot` dan `<Directory>` mengarah ke `.../smauii-core/public`:

```apache
<VirtualHost *:80>
    DocumentRoot "C:/laragon/www/smauii-core/public"
    ServerName smauii-core.test
    ServerAlias *.smauii-core.test
    <Directory "C:/laragon/www/smauii-core/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Setelah diubah, restart Apache dari menu Laragon.

### 8. Verifikasi

- Buka `http://smauii-core.test` — seharusnya menampilkan halaman selamat datang Laravel
- Buka `http://smauii-aksesekolah.test` — seharusnya menampilkan dokumentasi

---

## SSH & GitHub Authentication

Developer harus memiliki akses SSH ke GitHub组织 `SMA-UII-Yogyakarta`.

### Setup SSH Key

```bash
# Generate SSH key (jika belum punya)
ssh-keygen -t ed25519 -C "email@anda.com"

# Lihat public key
cat ~/.ssh/id_ed25519.pub
```

Tambahkan public key ke GitHub:
1. Buka https://github.com/settings/keys
2. Klik **New SSH Key**
3. Tempel public key

### Login GitHub CLI

```bash
gh auth login
# Pilih: GitHub.com
# Pilih: SSH
# Upload SSH key ke akun GitHub
```

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| **Directory listing** saat buka `smauii-core.test` | DocumentRoot belum diarahkan ke `/public` — ikuti langkah 6 di atas |
| **PostgreSQL connection refused** | Pastikan PostgreSQL menyala di Laragon (Menu > Tools > Service/Port > PostgreSQL) |
| **pdo_pgsql not found** | Pastikan PHP 8.4 punya ekstensi pdo_pgsql dan pgsql (cek `php -m`). Tambahkan `extension=pdo_pgsql` di php.ini jika belum. |
| **Composer not found** | Gunakan `C:\laragon\bin\composer\composer.bat` atau langsung `php C:\laragon\bin\composer\composer.phar` |
| **Bun command not found** | Pastikan Bun terinstal: `bun --version`. Install: `powershell -c "irm bun.sh/install.ps1 | iex"` |
| **Port 80/5432 sudah dipakai** | Cek aplikasi lain (Skype, Docker, IIS) yang menggunakan port tersebut |
| **APP_KEY not set** | Jalankan `php artisan key:generate` |
| **Bun install gagal di Windows** | Coba jalankan PowerShell sebagai Administrator, atau install ulang: `powershell -c "irm bun.sh/install.ps1 | iex"` |
