# Project Context — SMART Absen SMA UII

## Ringkasan
Sistem presensi digital SMA UII Yogyakarta berbasis web dengan geolokasi, swafoto kamera, dan SSO. Dikembangkan oleh PT Koneksi Jaringan Indonesia.

## Tech Stack
- **Backend**: PHP 8.5 / Laravel 13 — repositori: `core.git`
- **Database**: MySQL 8.0
- **Cache & Queue**: Redis / Database driver
- **Object Storage**: S3-compatible (Wasabi / MinIO)
- **Web Server**: Apache (dev) / Nginx (production)
- **Frontend**: Blade + Tailwind CSS 4 + Vite
- **Auth**: Laravel Sanctum (SSO / Identity Provider)

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
| Ahmad Hanif Hasan | Document Developer | brief/, docs/ |
| Fathan Mubina | Junior Developer | core.git (backend) |
| Ihsan | Junior Developer | core.git (backend) |

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
