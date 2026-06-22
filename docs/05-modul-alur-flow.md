# Modul & Alur Program

## Skenario Umum — SSO Login

```
┌─────────┐     ┌──────────────┐     ┌─────────────┐
│ Browser │────>│  SSO Portal  │────>│  Deteksi    │
│         │     │  /login      │     │  Role       │
└─────────┘     └──────────────┘     └──────┬──────┘
                                            │
                    ┌───────────────────────┼───────────────────────┐
                    │                       │                       │
                    ▼                       ▼                       ▼
            ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
            │ Role: Admin  │       │ Role: Siswa  │       │ Role: Guru   │
            │ /admin/*     │       │ /siswa/*     │       │ /guru/*      │
            └──────────────┘       └──────────────┘       └──────────────┘

                    ┌──────────────┐
                    │ Role: Wali   │
                    │ Murid        │
                    │ /wali/*      │
                    └──────────────┘
```

---

## 1. Alur Presensi Siswa

Ini adalah alur paling penting dan paling kritis dalam sistem. Harus dioptimasi untuk kecepatan dan keandalan.

```
[Siswa membuka browser]
        │
        ▼
[Login via SSO] ─── (deteksi role: siswa) ───> [Dashboard Siswa]
        │
        ▼
[Menu: Presensi]
        │
        ▼
[Sistem meminta izin:]
  ● Akses Kamera (WebRTC)
  ● Akses Lokasi (Geolocation API)
        │
        ▼
[Siswa mengambil selfie]
        │
        ▼
[Kompresi gambar client-side]
  ● Resize: 320x240 px
  ● Kualitas: JPEG 90%
  ● Target: ≤ 20 KB
        │
        ▼
[Kirim data ke server:]
  ● foto (Base64 terkompresi)
  ● latitude
  ● longitude
  ● timestamp
        │
        ▼
[Server melakukan Triple-Layer Validation]
  1. Cek kalender akademik (hari libur?)
  2. Cek hari aktif (Sabtu/Minggu?)
  3. Cek rentang jam:
     │
     ├── < jam_buka_masuk   → "Belum waktunya presensi"
     ├── ≤ batas_terlambat  → status: "Hadir"
     ├── ≤ jam_tutup_masuk  → status: "Terlambat"
     └── > jam_tutup_masuk  → "Presensi ditutup"
        │
        ▼
[Simpan ke database]
  ● Foto diupload ke Object Storage
  ● URL foto disimpan di tabel presensi
        │
        ▼
[Tampilkan konfirmasi ke siswa]
```

### Detail Teknis Presensi

| Langkah | Teknologi | Catatan |
|---|---|---|
| Akses kamera | `navigator.mediaDevices.getUserMedia()` | Wajib HTTPS |
| Tangkapan gambar | Canvas API: `canvas.toDataURL('image/jpeg', 0.9)` | Resize ke 320x240 |
| Geolokasi | `navigator.geolocation.getCurrentPosition()` | `enableHighAccuracy: true` |
| Validasi server | Laravel Request + Validation | Triple-layer |
| Upload foto | Laravel Filesystem + S3 driver | File disimpan, URL masuk DB |

---

## 2. Alur Wali Murid — Pantau & Izin

```
[Wali Murid login SSO]
        │
        ▼
[Dashboard Wali Murid]
        │
        ├── [Pilih profil anak] ─── [Lihat grafik kehadiran]
        │                              ● Harian
        │                              ● Bulanan
        │                              ● Semester
        │
        └── [Menu: Ajukan Izin]
                │
                ▼
        [Form Izin]
          ● Pilih anak (jika >1)
          ● Kategori: Sakit / Acara / Lomba
          ● Tanggal mulai – selesai
          ● Upload bukti (surat dokter, surat tugas)
          ● Submit
                │
                ▼
        [Status: Pending]
        (Menunggu verifikasi Wali Kelas)
```

### Aturan Bisnis Izin

| Aturan | Detail |
|---|---|
| Relasi | 1 Wali Murid → N Siswa (kakak-beradik) |
| Upload bukti | Format: JPG, PNG, PDF. Maks 2 MB per file |
| Status approval | Pending → Disetujui/Ditolak (oleh Wali Kelas) |
| Izin berturut-turut | Boleh, dengan rentang tanggal yang valid |

---

## 3. Alur Wali Kelas — Verifikasi & Monitoring

```
[Wali Kelas login SSO]
        │
        ▼
[Dashboard Wali Kelas]
        │
        ├── [Notifikasi: Izin Masuk]
        │       │
        │       ▼
        │   [Panel Verifikasi Izin]
        │     ● Daftar pengajuan dari wali murid
        │     ● Lihat detail + dokumen bukti
        │     ● Approve / Reject
        │
        └── [Rekap Kelas]
              ● Filter: Hari ini / Bulan ini / Semester
              ● Lihat daftar siswa + status kehadiran
              ● Ekspor PDF/Excel
```

---

## 4. Alur Guru Piket — Monitoring Real-Time

```
[Guru Piket login SSO]
        │
        ▼
[Dashboard Guru Piket]
        │
        ▼
[Monitoring Harian]
        │
        ▼
[Filter Cepat — Pilih Kelas]
        │
        ▼
[Tampilkan:]
  ● Total siswa di kelas
  ● ✅ Sudah presensi (Hadir)
  ● ⚠️ Terlambat
  ● ❌ Belum presensi / Alpa
  ● Waktu presensi masing-masing
```

---

## 5. Alur Admin — Manajemen Pusat

```
[Admin login SSO]
        │
        ▼
[Dashboard Admin — Statistik Sekolah]
        │
        ├── [Manajemen Data Master]
        │     ├── Siswa   → CRUD + Import/Export Excel
        │     ├── Guru    → CRUD + Import/Export Excel
        │     ├── Wali Murid → CRUD + Import/Export Excel
        │     └── Kelas   → CRUD + Enrolment
        │
        ├── [Manajemen Presensi & Izin]
        │     ├── Lihat rekap harian/bulanan/semester
        │     ├── Koreksi / override data (bypass)
        │     └── Ekspor PDF/Excel
        │
        ├── [Pengaturan]
        │     ├── Jam Presensi → Buka/Terlambat/Tutup per hari
        │     └── Kalender Akademik → Daftar hari libur
        │
        └── [Drill-Down Analytics]
              Level 1 → [Grafik Seluruh Sekolah]
                    ↓ klik
              Level 2 → [Grafik per Kelas]
                    ↓ klik
              Level 3 → [Statistik per Siswa]
                    ↓ klik
              Level 4 → [Detail Kehadiran by Name]
```

### Fitur Import/Export Excel

| Fitur | Detail |
|---|---|
| Template download | Sistem menyediakan template Excel dengan kolom yang benar |
| Import | Upload file Excel → validasi → insert/update batch |
| Export | Download data master atau rekap ke Excel |
| Library rekomendasi | PhpSpreadsheet (untuk Laravel) |

---

## 6. Alur Laporan & Export

```
[User: Admin / Wali Kelas / Guru Piket]
        │
        ▼
[Menu: Laporan]
        │
        ├── [Filter]
        │     ● Rentang: Harian / Bulanan / Semester
        │     ● Kelas: (opsional, filter)
        │     ● Status: (opsional, filter)
        │
        └── [Export]
              ● PDF → untuk cetak resmi
              ● Excel → untuk olah data lanjutan
```

---

## Ringkasan Interface

| No | Interface | Role | Prioritas |
|---|---|---|---|
| 1 | Login SSO | Semua | P0 |
| 2 | Dashboard Admin | Admin | P0 |
| 3 | Manajemen Data Master | Admin | P0 |
| 4 | Manajemen Kelas & Enrolment | Admin | P0 |
| 5 | Pengaturan Waktu & Libur | Admin | P1 |
| 6 | Dashboard Siswa | Siswa | P0 |
| 7 | Form Presensi Live | Siswa | P0 |
| 8 | Dashboard Wali Murid | Wali Murid | P0 |
| 9 | Form Pengajuan Izin | Wali Murid | P0 |
| 10 | Dashboard Wali Kelas | Wali Kelas | P0 |
| 11 | Panel Verifikasi Izin | Wali Kelas | P1 |
| 12 | Dashboard Guru Piket | Guru Piket | P0 |
| 13 | Laporan & Export | Admin, Wali Kelas, Guru Piket | P1 |
