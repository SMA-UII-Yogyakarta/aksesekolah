# Analisis Kebutuhan Sistem

## Deskripsi Umum

**SMART Absen SMA UII** adalah sistem informasi presensi siswa terintegrasi berbasis web responsif yang dirancang untuk mengotomatisasi, mengamankan, dan mensentralisasi pencatatan kehadiran siswa di lingkungan SMA UII.

Sistem ini mengintegrasikan:
- **Pelacakan lokasi** (Geolocation API)
- **Pengambilan gambar biometrik** (WebRTC camera)
- **Kompresi gambar client-side** untuk efisiensi bandwidth
- **Single Sign-On (SSO)** sebagai fondasi identitas digital terpusat

### Target Beban

| Metrik | Nilai |
|---|---|
| Concurrent users (CCU) puncak | 750+ siswa |
| Waktu puncak | 06:30 – 07:00 WIB |
| Ukuran foto per siswa (setelah kompresi) | Maks 20 KB |
| Total payload 760 siswa serentak | ~15,2 MB |

---

## Kebutuhan Fungsional

### 1. Modul Pintu Masuk — Universal SSO Portal

| ID | Kebutuhan |
|---|---|
| F-01 | Sistem harus menyediakan satu portal autentikasi (login) tunggal untuk seluruh kategori pengguna |
| F-02 | Sistem harus otomatis mendeteksi peran (role) setelah login berhasil dan mengarahkan ke dashboard masing-masing |
| F-03 | Arsitektur login harus dirancang sebagai **Identity Provider (IdP)** menggunakan Laravel Sanctum agar dapat digunakan oleh aplikasi yayasan lain di masa depan |

### 2. Modul Admin — Pengelola Pusat

| ID | Kebutuhan |
|---|---|
| F-04 | **Manajemen Data Master:** CRUD satuan & bulk (Excel upload/download) untuk entitas Siswa, Guru, Wali Murid |
| F-05 | **Enrolment Kelas:** Membuat kelas, menunjuk wali kelas, memasukkan siswa ke kelas secara bulk |
| F-06 | **Manajemen Presensi & Izin:** Hak akses penuh CRUD untuk koreksi/override data kehadiran |
| F-07 | **Dashboard Analitik (Drill-Down 4 Level):** Level 1 (sekolah) → Level 2 (per kelas) → Level 3 (per siswa) → Level 4 (detail by name) |

### 3. Modul Siswa

| ID | Kebutuhan |
|---|---|
| F-08 | **Presensi Live:** Ambil selfie (wajib kamera) + kunci titik koordinat GPS real-time |
| F-09 | **Laporan Pribadi:** Grafik dan rekap kehadiran mandiri filter Harian/Bulanan/Semester |

### 4. Modul Wali Murid

| ID | Kebutuhan |
|---|---|
| F-10 | **Multi-Profil Anak:** Satu akun dapat mengampu >1 siswa (kakak-beradik), dengan fitur switch-profile |
| F-11 | **Pengajuan Izin Digital:** Form izin (Sakit/Acara/Lomba) + pilih anak + durasi + upload bukti (gambar/PDF) |

### 5. Modul Wali Kelas

| ID | Kebutuhan |
|---|---|
| F-12 | **Panel Verifikasi Izin:** Notifikasi masuk pengajuan izin dari wali murid, dapat Approve/Reject |
| F-13 | **Monitoring & Rekap Kelas:** Laporan kehadiran seluruh siswa di kelasnya (Harian/Bulanan/Semester) |

### 6. Modul Guru Piket

| ID | Kebutuhan |
|---|---|
| F-14 | **Live Monitoring Filtered:** Rekap kehadiran seluruh siswa hari berjalan dengan filter cepat per kelas |

---

## Kebutuhan Non-Fungsional

### Performa

| ID | Kebutuhan | Spesifikasi |
|---|---|---|
| NF-01 | **Frontend Image Compression** | Resolusi 320x240, JPEG 90%, maks 20 KB per foto, via JavaScript Canvas API |
| NF-02 | **Database max_connections** | Minimal 1000 koneksi serentak |
| NF-03 | **Database Indexing** | Wajib pada kolom id_siswa, tanggal, status_kehadiran, nisn, nama |
| NF-04 | **Storage Offloading** | Dilarang menyimpan BLOB/Base64 di database — gunakan URL ke Object Storage eksternal |

### Usability

| ID | Kebutuhan |
|---|---|
| NF-05 | **Responsive Web Design** — Siswa & Wali Murid dioptimalkan untuk mobile; Admin, Guru untuk desktop |
| NF-06 | **Tanpa instalasi** — cukup browser, tidak perlu unduh dari Play Store/App Store |
| NF-07 | **Pembaruan instan** — setiap perubahan langsung tersedia tanpa update manual |

### Keamanan

| ID | Kebutuhan |
|---|---|
| NF-08 | **SSO Tokenization** — sistem token yang aman untuk mencegah manipulasi sesi login |
| NF-09 | **Role-based middleware** — pengguna tidak bisa mengakses URL/memanipulasi data di luar hak aksesnya |
| NF-10 | **Validasi Tiga Lapis (Triple-Layer)** sebelum menyimpan presensi — lihat dokumen keamanan |

### Keandalan

| ID | Kebutuhan |
|---|---|
| NF-11 | **Ekspor PDF & Excel** untuk laporan Harian/Bulanan/Semester |
| NF-12 | **Server SSD/NVMe** untuk menjamin kecepatan I/O |

---

## Matriks Role vs Fitur

| Fitur | Admin | Siswa | Wali Murid | Wali Kelas | Guru Piket |
|---|---|---|---|---|---|
| Login SSO | ✅ | ✅ | ✅ | ✅ | ✅ |
| CRUD Master Data | ✅ | — | — | — | — |
| Upload/Download Excel | ✅ | — | — | — | — |
| Enrolment Kelas | ✅ | — | — | — | — |
| Atur Jam Presensi | ✅ | — | — | — | — |
| Atur Kalender Libur | ✅ | — | — | — | — |
| Presensi Live (Selfie+GPS) | — | ✅ | — | — | — |
| Lihat Rekap Sendiri | ✅ | ✅ | — | — | — |
| Multi-Profil Anak | — | — | ✅ | — | — |
| Ajukan Izin | — | — | ✅ | — | — |
| Verifikasi Izin | ✅ | — | — | ✅ | — |
| Monitoring Kelas | ✅ | — | — | ✅ | ✅ |
| Filter Cepat per Kelas | ✅ | — | — | ✅ | ✅ |
| Drill-Down Analytics | ✅ | — | — | — | — |
| Ekspor PDF/Excel | ✅ | — | — | ✅ | ✅ |
| Bypass/Override Data | ✅ | — | — | — | — |
