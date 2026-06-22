# Glossary — Istilah & Singkatan

| Istilah | Kepanjangan | Deskripsi |
|---|---|---|
| **SMART Absen** | — | Nama sistem presensi digital SMA UII |
| **SSO** | Single Sign-On | Login satu akun untuk semua aplikasi SMA UII |
| **RBAC** | Role-Based Access Control | Sistem hak akses berdasarkan peran (Admin, Siswa, Guru, dll) |
| **IdP** | Identity Provider | Penyedia identitas untuk SSO (Laravel Sanctum) |
| **Presensi Live** | — | Fitur presensi real-time dengan selfie + GPS |
| **Triple-Layer Validation** | — | Validasi 3 lapis: kalender akademik → hari aktif → rentang jam |
| **Object Storage** | — | Penyimpanan file (foto) di S3-compatible, bukan di DB |
| **Wali Murid** | — | Orang tua/wali siswa yang bisa mengajukan izin |
| **Wali Kelas** | — | Guru yang memverifikasi izin siswa di kelasnya |
| **Guru Piket** | — | Guru yang memonitor presensi real-time per kelas |
| **Monorepo** | — | Satu repo dengan submodules untuk multiple apps |
| **Laragon** | — | Development environment Windows (PHP, MySQL, Apache) |

## Nama Repositori

| Nama Pendek | Repo GitHub | Isi |
|---|---|---|
| aksesekolah | `SMA-UII-Yogyakarta/aksesekolah` | Monorepo entrypoint + docs |
| core | `SMA-UII-Yogyakarta/core` | Backend Laravel |
| webapp | `SMA-UII-Yogyakarta/webapp` | Frontend web *(planned)* |
| flutter | `SMA-UII-Yogyakarta/flutter` | Mobile app *(planned)* |
| .github | `SMA-UII-Yogyakarta/.github` | Profil organisasi & templates |

## Domain

| Domain | Fungsi |
|---|---|
| `smauii-core.test` | Dev lokal |
| `absen.smauii.sch.id` | Production *(planned)* |
| `SMA-UII-Yogyakarta.github.io/aksesekolah` | GitHub Pages (dokumentasi) |
