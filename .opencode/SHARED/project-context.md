# Project Context — SMART Absen SMA UII

## Ringkasan
Sistem presensi digital SMA UII Yogyakarta berbasis web dengan geolokasi, swafoto kamera, dan SSO. Dikembangkan oleh PT Koneksi Jaringan Indonesia.

## Tech Stack
- **Backend**: PHP 8.4 / Laravel 13 — repositori: `core.git`
- **Frontend**: InertiaJS 3 + React 19 + TypeScript 5.7 + Tailwind CSS 4 + Vite 8
- **Package Manager**: Bun (frontend), Composer (backend)
- **Database**: PostgreSQL 16 via NeonDB (production), PostgreSQL 16 (local), SQLite (testing)
- **Cache & Queue**: Redis / Database driver
- **Object Storage**: S3-compatible (Wasabi / MinIO)
- **Web Server**: Apache (dev) / Nginx (production)
- **Auth**: Laravel Sanctum + Spatie Laravel Permission (RBAC)

## Arsitektur
```
aksesekolah.git (monorepo entrypoint)
├── apps/backend/ → submodule → core.git
├── brief/        → dokumen perencanaan
├── docs/         → dokumentasi teknis
├── .opencode/    → konfigurasi opencode
└── .openkb/      → knowledge base
```

## Tim
| Person | Role | Repo Fokus |
|---|---|---|
| sandikodev | Project Manager | Semua repo |
| Ahmad Hanif Hasan | Product Analyst | brief/, docs/ |
| Fathan Mubina | Junior Frontend Developer | core.git (Inertia+React) |
| Ihsan | Junior Backend Developer | core.git (Laravel API) |
| Azis | Learning Mentor | core.git, playbook (mentoring) |

## Database (10 tabel)
- **Master**: users, siswa, guru, wali_murid, kelas
- **Transaksi**: presensi, pengajuan_izin, jadwal_piket
- **Konfigurasi**: pengaturan_jam_presensi, kalender_akademik

## Aturan Penting
1. Semua perubahan `main` harus via PR dengan minimal 1 review
2. Commit menggunakan Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
3. Coding style: PSR-12 (Laravel Pint)
4. Test harus hijau sebelum merge
5. Kredit milik PT Koneksi Jaringan Indonesia — dilarang memperjualbelikan source code
