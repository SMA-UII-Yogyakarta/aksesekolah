---
layout: default
title: Beranda
---

# SMART Absen SMA UII

**Sistem Presensi Digital Terintegrasi** berbasis web dengan geolokasi, swafoto kamera, dan Single Sign-On.

SMART Absen adalah aplikasi presensi digital yang memungkinkan siswa melakukan absensi melalui swafoto dan verifikasi geolokasi, wali murid mengajukan izin secara digital, serta guru memonitor kehadiran secara **real-time**.

---

## Dokumentasi Teknis

Seluruh dokumentasi tersedia dalam repositori ini. Silakan navigasi sesuai peran Anda:

### 📘 Untuk Tim Administratif & Manajemen Sekolah

| Dokumen | Deskripsi |
|---|---|
| [Analisis Kebutuhan](03-requirement-analisis.md) | Daftar lengkap kebutuhan fungsional (14 fitur) dan non-fungsional (12 item) |
| [ERD & Struktur Database](04-erd-database.md) | Entity Relationship Diagram dan penjelasan 10 tabel database |
| [Modul & Alur Flow](05-modul-alur-flow.md) | Skenario lengkap penggunaan per tiap peran pengguna |
| [Anggaran & Timeline](08-budget-timeline-roadmap.md) | Rencana anggaran Rp 8.500.000 dan timeline 8 minggu |

### ⚙️ Untuk Tim Teknis & Developer

| Dokumen | Deskripsi |
|---|---|
| [Arsitektur Monorepo](01-arsitektur-monorepo.md) | Struktur monorepo, git submodules, tata letak direktori |
| [Lingkungan Development](02-lingkungan-development.md) | Setup Laragon, PHP 8.4, PostgreSQL 16, troubleshooting |
| [Keamanan & SSO](06-keamanan-sso.md) | Laravel Sanctum, RBAC, Triple-Layer Validation |
| [Git Workflow](07-git-workflow-submodule.md) | Branching strategy, manajemen submodule, CI/CD |
| [Deployment & Infrastruktur](09-deployment-infrastruktur.md) | VPS, Nginx, object storage S3, performance tuning |

---

## Memulai Cepat

### Prasyarat

| Tool | Keterangan |
|---|---|
| [Laragon](https://laragon.org) 6.0+ | Full-stack web environment (PHP, Apache) |
| PHP 8.3+ | Sudah termasuk Laragon |
| PostgreSQL 16 | Database (via NeonDB atau Laragon add-on) |
| Composer | Dependency manager PHP |
| Bun 1.2+ | Package manager & build tooling frontend |

### Clone & Setup

```bash
# Clone backend ke Laragon
cd C:\laragon\www
git clone git@github.com:SMA-UII-Yogyakarta/core.git smauii-core

# Install dependencies
composer install

# Environment & key
cp .env.example .env
php artisan key:generate

# Database
php artisan migrate --seed
```

Buka **http://smauii-core.test** — aplikasi siap digunakan.

---

## Ekosistem Repositori

```
SMA-UII-Yogyakarta/
├── aksesekolah/   → Monorepo entrypoint (repositori ini)
│   ├── apps/backend/ → core.git (Laravel 13)
│   ├── apps/frontend/ → webapp.git [planned]
│   ├── apps/mobile/ → flutter.git [planned]
│   ├── brief/     → Dokumen perencanaan awal
│   └── docs/      → Dokumentasi teknis ← Kamu di sini
├── core/          → Backend API Laravel 13
├── webapp/        → Frontend web [planned]
├── flutter/       → Mobile app [planned]
└── .github/       → Profil organisasi GitHub
```

---

## Dikembangkan Oleh

**PT Koneksi Jaringan Indonesia** — *Software House Agency Koneksi Digital*

Mitra resmi pengembangan teknologi informasi SMA UII Yogyakarta.

> **Hak Cipta & Lisensi** — Seluruh source code dalam proyek ini dikembangkan oleh PT Koneksi Jaringan Indonesia untuk SMA UII Yogyakarta. Hak cipta dilindungi undang-undang. Dilarang memperjualbelikan atau mendistribusikan ulang tanpa izin tertulis dari kedua belah pihak.

---

<p align="center">
  <strong>SMA UII Yogyakarta</strong><br />
  Jl. Taman Siswa No.158, Wirogunan, Mergangsan, Kota Yogyakarta, DIY 55151<br />
  Telp: (0274) 489693<br />
  <a href="https://www.instagram.com/smauiiofficial/">📸 Instagram</a> ·
  <a href="https://www.youtube.com/channel/UCaLhqaoGXpLHK-KlTwiS8aw">▶️ YouTube</a> ·
  <a href="https://www.tiktok.com/@smauiiofficial">🎵 TikTok</a><br />
  <em>Smart School · Bright Future</em>
</p>
