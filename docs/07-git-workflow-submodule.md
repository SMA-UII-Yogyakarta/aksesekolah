# Git Workflow & Manajemen Submodule

## Filosofi Branching

Proyek ini menggunakan **Git Flow** yang disederhanakan dengan tiga branch utama:

```
main           ───●────────────────────●───────────►
                  │                    │
develop          ──●──●──●──●──●──●──●──●──●──●──►
                  │  │  │  │  │  │  │  │  │  │
feature/*        ──┘  └──┘  └──┘  └──┘  └──┘
```

| Branch | Fungsi | Deploy |
|---|---|---|
| `main` | Kode stabil, siap produksi | ✅ Ya |
| `develop` | Integrasi fitur, pengembangan harian | ❌ Tidak |
| `feature/*` | Fitur spesifik, branch dari `develop` | ❌ Tidak |
| `hotfix/*` | Perbaikan darurat, branch dari `main` | ✅ Ya (setelah merge) |

---

## Workflow Harian Developer

### 1. Memulai Fitur Baru

```bash
# Pastikan develop terbaru
git checkout develop
git pull origin develop

# Buat branch fitur
git checkout -b feature/nama-fitur

# Kerja seperti biasa...
git add .
git commit -m "feat: menambahkan fitur X"
```

### 2. Menyelesaikan Fitur

```bash
# Update branch fitur dengan develop terbaru
git checkout develop
git pull origin develop
git checkout feature/nama-fitur
git rebase develop

# Selesaikan konflik jika ada
# Force push (hati-hati, hanya untuk branch sendiri!)
git push origin feature/nama-fitur --force-with-lease

# Buat Pull Request ke develop
gh pr create --base develop --head feature/nama-fitur
```

### 3. Code Review & Merge

```bash
# Setelah PR di-review dan disetujui:
gh pr merge feature/nama-fitur --squash

# Hapus branch fitur
git branch -d feature/nama-fitur
git push origin --delete feature/nama-fitur
```

### 4. Release ke Produksi

```bash
# Merge develop ke main
git checkout main
git pull origin main
git merge develop
git push origin main
```

---

## Manajemen Submodule

### Konsep

Submodule adalah referensi git ke repositori lain di dalam repositori induk. Monorepo `aksesekolah.git` menggunakan submodule untuk merujuk ke repositori aplikasi yang terpisah.

```
smauii-aksesekolah/           ← repositori induk
├── apps/
│   ├── backend/              ← submodule → core.git (SHA pointer)
│   ├── frontend/             ← submodule → webapp.git (SHA pointer) [future]
│   └── mobile/               ← submodule → flutter.git (SHA pointer) [future]
├── .gitmodules               ← daftar submodule
└── .git/config               ← URL remote submodule
```

### Menambahkan Submodule Baru

```bash
# Dari root repositori induk
cd C:\laragon\www\smauii-aksesekolah

# Tambah submodule backend
git submodule add git@github.com:SMA-UII-Yogyakarta/core.git apps/backend

# Commit perubahan .gitmodules
git add .gitmodules apps/backend
git commit -m "chore: add backend submodule"
git push
```

### Clone Repositori dengan Submodule

```bash
# Cara 1: Clone langsung dengan submodule
git clone --recurse-submodules git@github.com:SMA-UII-Yogyakarta/aksesekolah.git

# Cara 2: Clone dulu, lalu init submodule
git clone git@github.com:SMA-UII-Yogyakarta/aksesekolah.git
cd aksesekolah
git submodule update --init --recursive
```

### Update Submodule ke Versi Terbaru

```bash
# Cara 1: Update semua submodule
git submodule update --remote --recursive

# Cara 2: Update submodule spesifik
git submodule update --remote apps/backend

# Setelah update, commit perubahan pointer
git add apps/backend
git commit -m "chore: update backend submodule"
git push
```

### Bekerja di Dalam Submodule

Submodule pada dasarnya adalah repositori git biasa. Developer bisa masuk dan bekerja di dalamnya:

```bash
# Masuk ke submodule
cd apps/backend

# Buat branch sendiri
git checkout -b feature/presensi

# Kerja seperti biasa
git add .
git commit -m "feat: implementasi presensi"
git push origin feature/presensi

# Buat PR ke core.git (bukan ke aksesekolah.git!)
```

> **Catatan Penting:** Submodule berada dalam *detached HEAD* secara default.
> Selalu buat branch sebelum mulai bekerja di dalam submodule!

### Tips Submodule

| Situasi | Perintah |
|---|---|
| Clone project baru | `git clone --recurse-submodules <url>` |
| Pull + update submodule | `git pull --recurse-submodules` |
| Update semua submodule | `git submodule update --remote` |
| Submodule di branch tertentu | `git submodule foreach 'git checkout main'` |
| Lihat status submodule | `git submodule status` |

---

## Commit Message Convention

Gunakan konvensi [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <deskripsi singkat>

Contoh:
feat: tambah fitur presensi dengan geolokasi
fix: perbaiki validasi tanggal libur
chore: update dependency laravel/framework
docs: update dokumentasi arsitektur monorepo
style: perbaiki indentasi blade template
refactor: extract validation ke service class
test: tambah unit test untuk triple-layer validation
```

| Type | Keterangan |
|---|---|
| `feat` | Fitur baru |
| `fix` | Perbaikan bug |
| `chore` | Tugas rutin (update dependency, config) |
| `docs` | Dokumentasi |
| `style` | Perubahan format (tidak mengubah logika) |
| `refactor` | Refaktor kode (tidak mengubah fungsionalitas) |
| `test` | Penambahan test |

---

## GitHub Actions (Continuous Integration)

Rekomendasi pipeline CI untuk setiap repositori:

### core.git (Backend)

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  laravel:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction
      - run: php artisan key:generate
      - run: php artisan test
      - run: ./vendor/bin/phpstan analyse
```

---

## Alur Update Monorepo ke Submodule

```
[Backend Developer selesai fitur di core.git]
        │
        ▼
[Push ke core.git → branch main]
        │
        ▼
[Maintainer monorepo update submodule]
  cd smauii-aksesekolah
  git submodule update --remote apps/backend
  git add apps/backend
  git commit -m "chore: update backend submodule to latest"
  git push
        │
        ▼
[Tim lain melakukan git pull di aksesekolah.git]
  git pull --recurse-submodules
  git submodule update --init --recursive
```

---

## .gitignore Monorepo

```gitignore
# Node
node_modules/
npm-debug.log
yarn-error.log

# OS
.DS_Store
Thumbs.db
*.swp
*.swo

# IDE
.idea/
.vscode/
*.sublime-project
*.sublime-workspace
.zed/

# Env
.env
.env.local

# Log
*.log

# Temporary
tmp/
temp/
```
