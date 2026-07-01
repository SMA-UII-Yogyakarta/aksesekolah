# Development Environment

## Summary

All development for **SMART Absen SMA UII** is done on **Laragon 6.0.0** — a *local development environment* for Windows that provides an integrated web server stack.

> **Important Note:** All developers are required to use Laragon to ensure a uniform working environment. The use of XAMPP, WAMP, or other tools is strongly discouraged due to configuration and path differences.

---

## Current Environment Specifications

### Laragon

| Item | Value |
|---|---|
| Version | 6.0.0.2 |
| Installation | `C:\laragon` |
| Data | `C:\laragon\data` |
| Document Root | `C:\laragon\www` |
| Run at Startup | Yes |
| Auto VirtualHosts | Enabled (all `.test` folders automatic) |
| SSL | Enabled (port 443) |

### PHP

| Item | Value |
|---|---|
| **Active Version** | **8.4.22** (NTS, VC++ 2022 x64) |
| Path | `C:\laragon\bin\php\php-8.4.22-nts-Win32-vs17-x64\` |
| Configuration | `C:\laragon\bin\php\php-8.4.22-nts-Win32-vs17-x64\php.ini` |
| `memory_limit` | 512M |
| `max_execution_time` | 36000 (10 hours) |
| `post_max_size` | 2G |
| `upload_max_filesize` | 2G |
| `date.timezone` | **Asia/Jakarta** ✅ |
| Active Extensions | curl, gd, intl, mbstring, mysqli, pdo_mysql, **pdo_pgsql**, **pgsql**, zip, openssl, xsl |

**Other Available PHP Versions:**

| Version | Path | Xdebug |
|---|---|---|
| 8.2.27 | `C:\laragon\bin\php\php-8.2.27-nts-Win32-vs16-x64\` | ❌ |
| 8.1.31 | `C:\laragon\bin\php\php-8.1.31-nts-Win32-vs16-x64\` | ✅ 3.4.1 |
| 8.1.10 | `C:\laragon\bin\php\php-8.1.10-Win32-vs16-x64\` | ❌ |

> **Xdebug Note:** PHP 8.4.22 does not yet have a compatible Xdebug extension. For debugging needs, use PHP 8.1.31 which can be selected via Laragon menu > PHP > Version.

### Web Server

| Item | Value |
|---|---|
| **Active Server** | **Apache 2.4.54** (mod_fcgid) |
| Apache Path | `C:\laragon\bin\apache\httpd-2.4.54-win64-VS16\` |
| PHP Handler | FastCGI (mod_fcgid) via `C:\laragon\etc\apache2\fcgid.conf` |
| DocumentRoot | `C:\laragon\www` |
| Nginx (optional) | 1.22.0 (not active) |

### Database

| Item | Value |
|---|---|
| **Active Database** | **PostgreSQL 16** — via NeonDB (production) / Laragon (local) |
| Path (Laragon) | `C:\laragon\bin\postgresql\postgresql-16.x-winx64\` (manual install) |
| Port | 5432 |
| Root User | `postgres` (no local password) |
| NeonDB Production | `ep-xxx-xxxx.us-east-2.aws.neon.tech` (serverless) |
| NeonDB Branching | Each feature/PR can have its own DB branch |

**Note:** MySQL 8.0.30 is still available in Laragon (`C:\laragon\bin\mysql\mysql-8.0.30-winx64\`, port 3306) for compatibility, but is not used by this project. The entire environment uses PostgreSQL.

### PostgreSQL Installation in Laragon

1. Download PostgreSQL zip from [EnterpriseDB](https://www.enterprisedb.com/download-postgresql-binaries) or [PostgreSQL Windows](https://www.postgresql.org/download/windows/)
2. Extract to `C:\laragon\bin\postgresql\postgresql-16.x-winx64\`
3. Add PostgreSQL service via Laragon: Menu > Tools > Service/Port > PostgreSQL
4. Initialize database cluster:
   ```bash
   "C:\laragon\bin\postgresql\postgresql-16.x-winx64\bin\initdb.exe" -D "C:/laragon/data/postgresql-16" --username=postgres
   ```
5. Start PostgreSQL from Laragon

### Other Tools

| Tool | Version | Path |
|---|---|---|---|
| Node.js | 22.14.0 | `C:\laragon\bin\nodejs\node-v22.14.0-win-x64\` |
| **Bun** | **latest** | `C:\Users\<user>\.bun\bin\bun.exe` (install: `powershell -c "irm bun.sh/install.ps1 | iex"`) |
| Composer | latest (portable) | `C:\laragon\bin\composer\composer.phar` |
| Redis | 5.0.14.1 | `C:\laragon\bin\redis\redis-x64-5.0.14.1\` |
| Memcached | 1.6.8 | `C:\laragon\bin\memcached\memcached-1.6.8-win64-mingw\` |
| pgAdmin / DBeaver | latest | GUI tools for PostgreSQL |
| Git | (via Windows PATH) | `C:\Program Files\Git\cmd\git.EXE` |
| GitHub CLI | 2.95.0 | `gh` |

> **Note:** Use **Bun** for frontend package management (not npm). Bun is faster, has built-in TypeScript support, and is compatible with Vite 8.

---

## Projects in Document Root

All projects in `C:\laragon\www` automatically get a virtual host with `.test` domain:

| Domain | Folder | Description |
|---|---|---|
| `smauii-core.test` | `smauii-core` | SMART Absen Laravel Backend |
| `smauii-aksesekolah.test` | `smauii-aksesekolah` | Monorepo entrypoint (this repository) |

---

## Environment Setup for New Developers

### 1. Install Laragon

1. Download Laragon from [https://laragon.org/download](https://laragon.org/download) (**Full** version recommended)
2. Install to `C:\laragon` (make sure there are no spaces in the path)
3. Run Laragon, make sure Apache + PostgreSQL are running (**Start All** button)

### 2. Clone Repositories

```bash
# Clone monorepo
cd C:\laragon\www
git clone git@github.com:SMA-UII-Yogyakarta/aksesekolah.git smauii-aksesekolah

# Clone backend core (for daily development)
git clone git@github.com:SMA-UII-Yogyakarta/core.git smauii-core
```

### 3. Setup Bun (Frontend Package Manager)

```bash
# Install Bun (Windows PowerShell)
powershell -c "irm bun.sh/install.ps1 | iex"

# Verify
bun --version
```

### 4. Setup Backend (core)

```bash
cd C:\laragon\www\smauii-core

# Install PHP dependencies
composer install

# Install JS/TS dependencies (via Bun, not npm!)
bun install

# Copy environment
cp .env.example .env
# (adjust database configuration, see .env below)

# Generate APP_KEY
php artisan key:generate

# Setup database
# Create smauii_core database via pgAdmin / DBeaver, or:
php artisan db:create  # (if using db-tools package)

# Run migrations
php artisan migrate --seed

# Install Laravel Sanctum (token management)
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

# Install Spatie Laravel Permission (RBAC)
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

### 5. Initialize Monorepo Submodule

```bash
cd C:\laragon\www\smauii-aksesekolah

# Init repository
git init
git remote add origin git@github.com:SMA-UII-Yogyakarta/aksesekolah.git

# If backend submodule is already registered:
git submodule update --init --recursive
```

### 6. .env Configuration (Backend)

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

# Production (NeonDB) — uncomment when deploying
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

### 7. Virtual Host Configuration

Laragon will automatically create the virtual host `smauii-core.test` when the `smauii-core` folder is detected.
However, Laravel requires `DocumentRoot` to point to `public/`. If directory listing occurs,
adjust the file at:

```
C:\laragon\etc\apache2\sites-enabled\auto.smauii-core.test.conf
```

Make sure `DocumentRoot` and `<Directory>` point to `.../smauii-core/public`:

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

After changing, restart Apache from the Laragon menu.

### 8. Verification

- Open `http://smauii-core.test` — should display the Laravel welcome page
- Open `http://smauii-aksesekolah.test` — should display the documentation

---

## SSH & GitHub Authentication

Developers must have SSH access to the GitHub organization `SMA-UII-Yogyakarta`.

### SSH Key Setup

```bash
# Generate SSH key (if you don't have one)
ssh-keygen -t ed25519 -C "your@email.com"

# View public key
cat ~/.ssh/id_ed25519.pub
```

Add the public key to GitHub:
1. Open https://github.com/settings/keys
2. Click **New SSH Key**
3. Paste the public key

### GitHub CLI Login

```bash
gh auth login
# Select: GitHub.com
# Select: SSH
# Upload SSH key to GitHub account
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| **Directory listing** when opening `smauii-core.test` | DocumentRoot is not pointed to `/public` — follow step 6 above |
| **PostgreSQL connection refused** | Make sure PostgreSQL is running in Laragon (Menu > Tools > Service/Port > PostgreSQL) |
| **pdo_pgsql not found** | Make sure PHP 8.4 has pdo_pgsql and pgsql extensions (check `php -m`). Add `extension=pdo_pgsql` to php.ini if missing. |
| **Composer not found** | Use `C:\laragon\bin\composer\composer.bat` or directly `php C:\laragon\bin\composer\composer.phar` |
| **Bun command not found** | Make sure Bun is installed: `bun --version`. Install: `powershell -c "irm bun.sh/install.ps1 | iex"` |
| **Port 80/5432 already in use** | Check other applications (Skype, Docker, IIS) using that port |
| **APP_KEY not set** | Run `php artisan key:generate` |
| **Bun install fails on Windows** | Try running PowerShell as Administrator, or reinstall: `powershell -c "irm bun.sh/install.ps1 | iex"` |
