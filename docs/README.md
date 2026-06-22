# Dokumentasi Teknis — SMART Absen SMA UII

Selamat datang di repositori **SMART Absen SMA UII**. Dokumen ini adalah sumber pengetahuan utama (*single source of truth*) bagi seluruh tim pengembang dan *maintainer* sistem presensi digital terintegrasi berbasis web ini.

## Tentang Repositori Ini

Repositori `aksesekolah.git` adalah **monorepo entrypoint** yang menjadi gerbang utama (*entrypoint*) bagi seluruh ekosistem aplikasi SMA UII. Repositori ini tidak berisi kode aplikasi secara langsung, melainkan:

- **Dokumentasi teknis** dan *brief* pengembangan
- **Git submodules** yang menunjuk ke repositori aplikasi spesifik
- **Package libraries** utilitas internal milik yayasan

## Struktur Dokumentasi

| Dokumen | Deskripsi |
|---|---|
| [01-arsitektur-monorepo.md](./01-arsitektur-monorepo.md) | Arsitektur monorepo, submodules, dan tata letak direktori |
| [02-lingkungan-development.md](./02-lingkungan-development.md) | Panduan setup lingkungan development (Laragon & toolchain) |
| [03-requirement-analisis.md](./03-requirement-analisis.md) | Analisis kebutuhan fungsional dan non-fungsional |
| [04-erd-database.md](./04-erd-database.md) | Perancangan basis data dan Entity Relationship Diagram |
| [05-modul-alur-flow.md](./05-modul-alur-flow.md) | Alur program dan skenario tiap modul |
| [06-keamanan-sso.md](./06-keamanan-sso.md) | Arsitektur keamanan, SSO, dan Role-Based Access Control |
| [07-git-workflow-submodule.md](./07-git-workflow-submodule.md) | Panduan git workflow dan manajemen submodule |
| [08-budget-timeline-roadmap.md](./08-budget-timeline-roadmap.md) | Rencana anggaran, timeline 8 minggu, dan peta jalan masa depan |
| [09-deployment-infrastruktur.md](./09-deployment-infrastruktur.md) | Spesifikasi infrastruktur, deployment, dan object storage |

## Repositori Terkait

| Repositori | URL | Deskripsi |
|---|---|---|
| **aksesekolah.git** | `git@github.com:SMA-UII-Yogyakarta/aksesekolah.git` | Monorepo entrypoint (repositori ini) |
| **core.git** | `git@github.com:SMA-UII-Yogyakarta/core.git` | Backend Laravel (submodule di `apps/backend`) |
| **webapp.git** | `git@github.com:SMA-UII-Yogyakarta/webapp.git` | Frontend terpisah (submodule di `apps/frontend`) — *future* |
| **flutter.git** | `git@github.com:SMA-UII-Yogyakarta/flutter.git` | Mobile app native (submodule di `apps/mobile`) — *future* |

## Prasyarat Minimum

Sebelum memulai pengembangan, pastikan telah membaca [02-lingkungan-development.md](./02-lingkungan-development.md) untuk menyiapkan lingkungan kerja yang sesuai.
