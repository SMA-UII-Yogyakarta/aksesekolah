# Rencana Anggaran, Timeline & Peta Jalan

## Rencana Anggaran Biaya

### Total Estimasi: **Rp 8.500.000**

| Komponen | Deskripsi | Biaya |
|---|---|---|
| **Arsitektur Database & SSO** | Perancangan ERD, relasi multi-user, fondasi login SSO mandiri | Rp 1.800.000 |
| **Modul Presensi Utama** | Fitur akses kamera (swafoto) + penguncian titik koordinat GPS siswa | Rp 1.500.000 |
| **Modul Admin & Master Data** | CRUD (single & bulk Excel) untuk data Siswa, Guru, Kelas | Rp 1.000.000 |
| **Modul Guru Piket & Wali Kelas** | Filter monitoring real-time, sistem verifikasi izin, rekap kelas | Rp 1.000.000 |
| **Modul UI/UX Siswa & Wali Murid** | Dashboard responsif, grafik kehadiran, form pengajuan izin | Rp 1.150.000 |
| **Sewa Object Storage (1 tahun)** | 50–250 GB, S3-compatible | Rp 850.000 |
| **Setup & Deployment Server** | Konfigurasi VPS, instalasi database, deployment aplikasi | Rp 1.200.000 |

### Breakdown Object Storage

| Metrik | Nilai |
|---|---|
| Kapasitas | 50–250 GB |
| Durasi | 1 tahun |
| Estimasi foto per hari | 760 siswa × 20 KB = ~15 MB |
| Estimasi per bulan | ~450 MB |
| Estimasi per tahun | ~5.4 GB (masih aman di 50 GB) |
| Penyedia rekomendasi | Wasabi, Backblaze B2, atau MinIO (self-hosted) |

---

## Timeline Pengerjaan — 8 Minggu

```
Minggu       | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
             |---|---|---|---|---|---|---|---|
Analisis     |███|   |   |   |   |   |   |   |
Wireframing  |   |███|   |   |   |   |   |   |
Desain ERD   |   |███|   |   |   |   |   |   |
Integrasi SSO|   |   |███|   |   |   |   |   |
Server Setup |   |   |███|   |   |   |   |   |
Admin CRUD   |   |   |   |███|   |   |   |   |
Enrolment    |   |   |   |███|   |   |   |   |
Presensi     |   |   |   |   |███|   |   |   |
Guru Piket   |   |   |   |   |   |███|   |   |
Integrasi    |   |   |   |   |   |███|   |   |
Laporan      |   |   |   |   |   |   |███|   |
Testing      |   |   |   |   |   |   |███|   |
Mobile Opt   |   |   |   |   |   |   |███|   |
Bug Fixing   |   |   |   |   |   |   |   |███|
UAT          |   |   |   |   |   |   |   |███|
Deploy       |   |   |   |   |   |   |   |███|
Training     |   |   |   |   |   |   |   |███|
```

### Detail Mingguan

#### Minggu 1 — Analisis Kebutuhan ✅ (Selesai)
- Wawancara stakeholder
- Dokumentasi kebutuhan fungsional & non-fungsional
- Analisis alur bisnis

#### Minggu 2 — Wireframing & ERD
- Desain UI/UX wireframe untuk 13 interface
- Finalisasi Entity Relationship Diagram
- Review desain dengan tim

#### Minggu 3 — SSO & Server
- Implementasi SSO login dengan Laravel Sanctum
- Setup server development
- Setup Object Storage integration

#### Minggu 4 — Admin Dashboard
- CRUD master data (Siswa, Guru, Wali Murid)
- Import/Export Excel
- Enrolment kelas

#### Minggu 5 — Modul Presensi
- Integrasi kamera (WebRTC)
- Integrasi geolokasi (Geolocation API)
- Kompresi gambar client-side
- Triple-layer validation

#### Minggu 6 — Guru Piket & Integrasi
- Dashboard monitoring real-time
- Filter kelas
- Panel verifikasi izin
- Integrasi sistem

#### Minggu 7 — Laporan & Testing
- Export PDF/Excel
- Internal testing
- Optimalisasi mobile

#### Minggu 8 — Finalisasi
- Bug fixing
- User Acceptance Testing (UAT)
- Deployment server produksi
- Pelatihan pengguna

---

## Peta Jalan Masa Depan (Roadmap)

### Fase 1: SSO sebagai Identity Provider Yayasan

Sistem login dari aplikasi ini akan dijadikan portal identitas utama. Ke depannya, semua aplikasi sekolah cukup menggunakan satu akun SSO.

```
Target: Akhir Tahun 1

┌─────────────────────────────────────────┐
│            SSO Identity Provider         │
│              (SMART Absen)               │
├─────────────────────────────────────────┤
│  ● e-Learning                           │
│  ● Sistem Keuangan / SPP                │
│  ● Perpustakaan Digital                 │
│  ● Aplikasi yayasan lainnya             │
└─────────────────────────────────────────┘
```

### Fase 2: "UII Satu Data" — Sentralisasi Data

Mewujudkan sentralisasi data untuk mencegah data ganda antar unit. Seluruh rekam jejak siswa dan guru terintegrasi pada satu ID.

```
Target: Akhir Tahun 2

┌──────────────┐
│   Satu ID    │  ← NISN/NIP sebagai identitas tunggal
└──────┬───────┘
       │
       ├── Presensi
       ├── Akademik (Nilai, Raport)
       ├── Administrasi (SPP, Beasiswa)
       ├── Perpustakaan
       └── Ekstrakurikuler
```

### Fase 3: Automasi & Teknologi Cerdas

| Fitur | Deskripsi |
|---|---|
| **Geofencing Radius** | Pembatasan radius lokasi otomatis — siswa hanya bisa presensi di dalam area sekolah |
| **WhatsApp Gateway** | Notifikasi absensi real-time ke wali murid via WhatsApp |
| **Face Recognition** | Verifikasi wajah otomatis dari foto presensi (AI/ML) |
| **Mobile Native App** | Aplikasi Android/iOS native dengan dukungan offline |

---

## Catatan untuk Tim

1. **Prioritas P0** harus selesai sebelum sistem bisa digunakan (SSO, Presensi, Admin)
2. **Budget fleksibel** — komponen dapat disesuaikan jika ada perubahan scope
3. **Timeline 8 minggu** adalah target ideal, bisa mundur jika ada kendala teknis
4. **Dokumentasi** — setiap selesai modul, update dokumentasi di repositori ini
