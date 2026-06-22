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
| **Versi Aktif** | **8.5.7** (NTS, VC++ 2022 x64) |
| Path | `C:\laragon\bin\php\php-8.5-nts-Win32-vs17-x64-latest\` |
| Konfigurasi | `C:\laragon\bin\php\php-8.5-nts-Win32-vs17-x64-latest\php.ini` |
| `memory_limit` | 512M |
| `max_execution_time` | 36000 (10 jam) |
| `post_max_size` | 2G |
| `upload_max_filesize` | 2G |
| `date.timezone` | **Asia/Jakarta** ✅ |
| Ekstensi Aktif | curl, gd, intl, mbstring, mysqli, pdo_mysql, zip, openssl, xsl |

**PHP Lain yang Tersedia:**

| Versi | Path | Xdebug |
|---|---|---|
| 8.2.27 | `C:\laragon\bin\php\php-8.2.27-nts-Win32-vs16-x64\` | ❌ |
| 8.1.31 | `C:\laragon\bin\php\php-8.1.31-nts-Win32-vs16-x64\` | ✅ 3.4.1 |
| 8.1.10 | `C:\laragon\bin\php\php-8.1.10-Win32-vs16-x64\` | ❌ |

> **Catatan Xdebug:** PHP 8.5.7 belum memiliki ekstensi Xdebug yang kompatibel. Untuk kebutuhan debugging, gunakan PHP 8.1.31 yang dapat dipilih melalui menu Laragon > PHP > Versi.

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
| **Database Aktif** | **MySQL 8.0.30** — Community Server |
| Path | `C:\laragon\bin\mysql\mysql-8.0.30-winx64\` |
| Port | 3306 |
| Data Directory | `C:/laragon/data/mysql-8` |
| Root User | `root` (tanpa password) |
| `max_connections` | Default — **harus dinaikkan ke 1000** untuk produksi |

### Tools Lain

| Tool | Versi | Path |
|---|---|---|
| Node.js | 22.14.0 | `C:\laragon\bin\nodejs\node-v22.14.0-win-x64\` |
| npm | 10.9.2 | — |
| Composer | latest (portable) | `C:\laragon\bin\composer\composer.phar` |
| Redis | 5.0.14.1 | `C:\laragon\bin\redis\redis-x64-5.0.14.1\` |
| Memcached | 1.6.8 | `C:\laragon\bin\memcached\memcached-1.6.8-win64-mingw\` |
| Git | (via Windows PATH) | `C:\Program Files\Git\cmd\git.EXE` |
| GitHub CLI | 2.95.0 | `gh` |

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
3. Jalankan Laragon, pastikan Apache + MySQL menyala (tombol **Start All**)

### 2. Clone Repositori

```bash
# Clone monorepo
cd C:\laragon\www
git clone git@github.com:SMA-UII-Yogyakarta/aksesekolah.git smauii-aksesekolah

# Clone backend core (untuk development harian)
git clone git@github.com:SMA-UII-Yogyakarta/core.git smauii-core
```

### 3. Setup Backend (core)

```bash
cd C:\laragon\www\smauii-core

# Install dependencies
composer install

# Copy environment
cp .env.example .env
# (sesuaikan konfigurasi database, lihat .env di bawah)

# Generate APP_KEY
php artisan key:generate

# Jalankan migrasi
php artisan migrate --seed
```

### 4. Inisialisasi Submodule Monorepo

```bash
cd C:\laragon\www\smauii-aksesekolah

# Init repositori
git init
git remote add origin git@github.com:SMA-UII-Yogyakarta/aksesekolah.git

# Jika submodule backend sudah terdaftar:
git submodule update --init --recursive
```

### 5. Konfigurasi .env (Backend)

```env
APP_NAME="SMAUII Core"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://smauii-core.test

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=smauii-core
DB_USERNAME=root
DB_PASSWORD=

SESSION_DRIVER=database
QUEUE_CONNECTION=database
CACHE_STORE=database
```

### 6. Konfigurasi Virtual Host

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

### 7. Verifikasi

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
| **MySQL connection refused** | Pastikan MySQL menyala di Laragon (tombol Start All) |
| **Composer not found** | Gunakan `C:\laragon\bin\composer\composer.bat` atau langsung `php C:\laragon\bin\composer\composer.phar` |
| **Port 80/3306 sudah dipakai** | Cek aplikasi lain (Skype, Docker, IIS) yang menggunakan port tersebut |
| **APP_KEY not set** | Jalankan `php artisan key:generate` |
