<div align="center">
  <img src="brief/ERD.png" alt="SMART Absen SMA UII" width="100%" style="max-width: 800px; border-radius: 12px;" />

  <h1>SMART Absen SMA UII</h1>
  <p><strong>Sistem Presensi Digital Terintegrasi berbasis Web dengan Geolokasi & Biometrik Kamera</strong></p>

  <p align="center">
    <a href="docs/01-arsitektur-monorepo.md"><img src="https://img.shields.io/badge/arsitektur-monorepo-6C5CE7?style=flat-square" /></a>
    <a href="docs/03-requirement-analisis.md"><img src="https://img.shields.io/badge/requirements-analisis-00B894?style=flat-square" /></a>
    <a href="docs/04-erd-database.md"><img src="https://img.shields.io/badge/database-ERD-0984E3?style=flat-square" /></a>
    <a href="docs/08-budget-timeline-roadmap.md"><img src="https://img.shields.io/badge/budget-Rp_8,5jt-FDCB6E?style=flat-square" /></a>
    <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-d63031?style=flat-square" /></a>
  </p>
</div>

---

## Tentang Proyek

**SMART Absen SMA UII** adalah sistem informasi presensi siswa berbasis web responsif yang mengotomatisasi pencatatan kehadiran menggunakan teknologi:

- **Geolocation API** — mengunci titik koordinat siswa saat presensi
- **WebRTC / Kamera** — mengambil swafoto langsung (bukan dari galeri)
- **Kompresi Client-Side** — gambar diperkecil hingga ≤20 KB sebelum dikirim
- **Single Sign-On** — satu akun untuk seluruh ekosistem aplikasi SMA UII

Dirancang untuk menangani **750+ siswa konkuren** pada jam sibuk (06:30–07:00 WIB) di lingkungan **SMA UII Yogyakarta**.

### Fitur Utama

| Modul | Pengguna | Fungsi |
|---|---|---|
| Presensi Live | Siswa | Selfie + GPS → rekam kehadiran real-time |
| Dashboard | Semua | Statistik & grafik sesuai peran |
| Manajemen Data | Admin | CRUD + Excel bulk import/export |
| Pengajuan Izin | Wali Murid | Ajukan izin sakit/acara/lomba + upload bukti |
| Verifikasi Izin | Wali Kelas | Approve/reject izin siswa |
| Monitoring | Guru Piket | Pantau kehadiran real-time per kelas |
| Laporan | Admin/Guru | Export PDF & Excel harian/bulanan/semester |
| Pengaturan | Admin | Atur jam presensi & kalender akademik |

---

## Arsitektur Monorepo

Repositori ini adalah **entrypoint monorepo** yang menggunakan **git submodules** untuk memisahkan concern aplikasi:

```
smauii-aksesekolah/
├── apps/
│   ├── backend/    →  core.git     (Laravel 13)
│   ├── frontend/   →  webapp.git   (React/Vue)  [future]
│   └── mobile/     →  flutter.git  (Android/iOS) [future]
├── packages/       →  libraries internal SMA UII [future]
├── brief/          →  dokumen perencanaan awal
└── docs/           →  dokumentasi teknis lengkap
```

| Repositori | URL | Keterangan |
|---|---|---|
| `aksesekolah.git` | [`SMA-UII-Yogyakarta/aksesekolah`](https://github.com/SMA-UII-Yogyakarta/aksesekolah) | Monorepo ini |
| `core.git` | [`SMA-UII-Yogyakarta/core`](https://github.com/SMA-UII-Yogyakarta/core) | Backend Laravel |

---

## Panduan Mulai Cepat

### Prasyarat

- [Laragon](https://laragon.org) 6.0+ (full stack web environment)
- PHP 8.3+, MySQL 8.0+, Composer
- Git + SSH key terdaftar di GitHub

### Setup Backend

```bash
# Clone core ke Laragon
cd C:\laragon\www
git clone git@github.com:SMA-UII-Yogyakarta/core.git smauii-core

# Install dependencies
composer install

# Konfigurasi environment
cp .env.example .env
php artisan key:generate

# Setup database & migrasi
php artisan migrate --seed
```

Buka `http://smauii-core.test` — aplikasi siap digunakan.

### Setup Monorepo (untuk Maintainer)

```bash
git clone --recurse-submodules git@github.com:SMA-UII-Yogyakarta/aksesekolah.git
```

---

## Dokumentasi Lengkap

Seluruh dokumentasi teknis tersedia di direktori [`docs/`](docs/README.md):

| Dokumen | Isi |
|---|---|
| [Arsitektur Monorepo](docs/01-arsitektur-monorepo.md) | Submodules, tata letak direktori, diagram |
| [Lingkungan Development](docs/02-lingkungan-development.md) | Laragon, PHP 8.5, MySQL 8.0, troubleshooting |
| [Analisis Kebutuhan](docs/03-requirement-analisis.md) | Fungsional (14 fitur) + Non-fungsional (12 item) |
| [ERD & Database](docs/04-erd-database.md) | 10 tabel, indexing strategy, SQL DDL |
| [Modul & Alur Flow](docs/05-modul-alur-flow.md) | Skenario lengkap tiap peran |
| [Keamanan & SSO](docs/06-keamanan-sso.md) | Sanctum, RBAC, Triple-Layer Validation |
| [Git Workflow](docs/07-git-workflow-submodule.md) | Branching, submodule management, CI |
| [Budget & Timeline](docs/08-budget-timeline-roadmap.md) | Rp 8,5jt, 8 minggu, roadmap 3 fase |
| [Deployment](docs/09-deployment-infrastruktur.md) | VPS, Nginx, tuning, object storage |

---

## Tech Stack

| Layer | Teknologi |
|---|---|
| **Backend** | PHP 8.5 / Laravel 13 |
| **Database** | MySQL 8.0 |
| **Cache & Queue** | Redis |
| **Object Storage** | S3-compatible (Wasabi / MinIO / Backblaze B2) |
| **Web Server** | Apache (dev) / Nginx (production) |
| **Frontend** | Blade + Tailwind CSS 4 + Vite |
| **Charts** | Chart.js / ApexCharts |
| **Data Tables** | DataTables (jQuery) |
| **Auth** | Laravel Sanctum (SSO / Identity Provider) |

---

## Lisensi

Proyek ini dikembangkan untuk **SMA UII Yogyakarta** di bawah lisensi MIT.

---

<div align="center">
  <p>Dikembangkan oleh Tim Pengembang SMA UII Yogyakarta</p>
  <p>
    <a href="https://github.com/SMA-UII-Yogyakarta">GitHub Organization</a>
  </p>
</div>
