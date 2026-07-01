# Perancangan Basis Data & Entity Relationship Diagram

## Filosofi Desain

Basis data SMART Absen SMA UII dirancang dengan prinsip:

1. **Index-first** — semua kolom yang digunakan dalam pencarian (WHERE, JOIN, ORDER BY) wajib memiliki index
2. **No BLOB** — file gambar dan dokumen tidak disimpan di database, cukup URL ke Object Storage
3. **Enum ketat** — menggunakan tipe VARCHAR + CHECK constraint (PostgreSQL tidak memiliki ENUM native seperti MySQL, jadi gunakan Laravel enum casting + check constraint)
4. **JSONB untuk data semi-structured** — gunakan JSONB (PostgreSQL) untuk kolom yang strukturnya dinamis (misal: metadata presensi, preferensi pengguna)
5. **Relasi terindeks** — setiap Foreign Key harus memiliki index untuk performa JOIN

---

## Entity Relationship Diagram

Lihat file `../brief/ERD.png` untuk visualisasi ERD.

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│    users     │     │   wali_murid     │     │    guru      │
│──────────────│     │─────────────────│     │──────────────│
│ id (PK)      │────>│ id_wali_murid   │     │ id_guru (PK) │
│ username     │     │ id_user (FK)     │     │ id_user (FK) │
│ email        │     │ nama             │     │ nama         │
│ password     │     │ no_hp            │     │ kode_guru    │
│ role         │     │ alamat           │     └──────┬───────┘
└──────────────┘                                    │
       │                                            │
       │                                            │
       │    ┌──────────────────┐                    │
       │    │     siswa        │                    │
       │    │──────────────────│                    │
       └────│ id_siswa (PK)     │                    │
            │ id_user (FK)      │                    │
            │ id_kelas (FK) ────│────────────────┐   │
            │ id_wali_murid (FK)│                │   │
            │ nis (unique)      │                │   │
            │ nisn (INDEX)      │                │   │
            │ nama (INDEX)      │                │   │
            │ tanggal_lahir     │                │   │
            │ no_hp             │                │   │
            │ alamat            │                │   │
            │ tahun_angkatan    │                │   │
            │ status (INDEX)    │                │   │
            └────────┬──────────┘                │   │
                     │                           │   │
                     │ 1:N                       │   │
                     ▼                           │   │
            ┌──────────────────┐                 │   │
            │    presensi      │                 │   │
            │──────────────────│                 │   │
            │ id_presensi (PK) │                 │   │
            │ id_siswa (INDEX) │                 │   │
            │ tanggal (INDEX)  │                 │   │
            │ waktu_masuk      │                 │   │
            │ latitude         │                 │   │
            │ longitude        │                 │   │
            │ foto_url         │                 │   │
            │ status_kehadiran │                 │   │
            └──────────────────┘                 │   │
                     │                           │   │
                     │ 1:N                       │   │
                     ▼                           │   │
            ┌──────────────────┐                 │   │
            │ pengajuan_izin   │                 │   │
            │──────────────────│                 │   │
            │ id_izin (PK)     │                 │   │
            │ id_siswa (INDEX) │                 │   │
            │ id_wali_murid    │                 │   │
            │ kategori         │                 │   │
            │ tanggal_mulai    │                 │   │
            │ tanggal_selesai  │                 │   │
            │ bukti_dok_url    │                 │   │
            │ status_approval  │                 │   │
            └──────────────────┘                 │   │
                                                 │   │
            ┌──────────────────┐                 │   │
            │   kelas          │◄────────────────┘   │
            │──────────────────│                     │
            │ id_kelas (PK)    │                     │
            │ id_guru (FK) ────│─────────────────────┘
            │ nama_kelas       │
            └──────────────────┘

            ┌──────────────────────┐
            │ jadwal_piket         │
            │──────────────────────│
            │ id_jadwal (PK)       │
            │ id_guru (INDEX)      │
            │ hari_tugas (INDEX)   │
            └──────────────────────┘

            ┌──────────────────────────┐
            │ pengaturan_jam_presensi  │
            │──────────────────────────│
            │ id_pengaturan (PK)       │
            │ hari                     │
            │ jam_buka_masuk           │
            │ batas_terlambat          │
            │ jam_tutup_masuk          │
            └──────────────────────────┘

            ┌──────────────────────┐
            │ kalender_akademik    │
            │──────────────────────│
            │ id_libur (PK)        │
            │ tanggal              │
            │ keterangan           │
            │ status_libur         │
            └──────────────────────┘
```

---

## Spesifikasi Tabel

### A. Tabel Inti (Autentikasi & Master)

#### 1. Tabel `users` — Portal SSO

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| username | VARCHAR(50) | UNIQUE, INDEXED | Login via NISN/NIP |
| email | VARCHAR(100) | UNIQUE, NULLABLE | — |
| password | VARCHAR(255) | NOT NULL | Bcrypt hashed |
| role | ENUM('admin','siswa','guru','walimurid') | NOT NULL | — |
| timestamps | — | — | created_at, updated_at |

#### 2. Tabel `wali_murid`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_wali_murid | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_user | BIGINT UNSIGNED | FK → users.id | — |
| nama | VARCHAR(100) | NOT NULL | — |
| no_hp | VARCHAR(20) | NULLABLE | — |
| alamat | TEXT | NULLABLE | — |

#### 3. Tabel `guru`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_guru | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_user | BIGINT UNSIGNED | FK → users.id | — |
| nama | VARCHAR(100) | NOT NULL | — |
| kode_guru | VARCHAR(20) | UNIQUE | Kode identitas guru |

#### 4. Tabel `kelas`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_kelas | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_guru | BIGINT UNSIGNED | FK → guru.id | Wali Kelas |
| nama_kelas | VARCHAR(50) | NOT NULL | Contoh: "X-A Reguler" |

#### 5. Tabel `siswa`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_siswa | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_user | BIGINT UNSIGNED | FK → users.id | — |
| id_kelas | BIGINT UNSIGNED | FK → kelas.id | — |
| id_wali_murid | BIGINT UNSIGNED | FK → wali_murid.id | — |
| nis | VARCHAR(30) | UNIQUE | Nomor Induk Siswa |
| nisn | VARCHAR(30) | UNIQUE, **INDEXED** | Nomor Induk Siswa Nasional |
| nama | VARCHAR(100) | **INDEXED** | — |
| tanggal_lahir | DATE | NOT NULL | — |
| no_hp | VARCHAR(20) | NULLABLE | — |
| alamat | TEXT | NULLABLE | — |
| tahun_angkatan | YEAR | NOT NULL | — |
| status | ENUM('Aktif','Non-Aktif') | **INDEXED** | — |

### B. Tabel Transaksional

#### 6. Tabel `presensi` — Tabel Paling Berat

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_presensi | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_siswa | BIGINT UNSIGNED | FK → siswa.id, **INDEXED** | — |
| tanggal | DATE | **INDEXED** | — |
| waktu_masuk | TIME | NOT NULL | — |
| latitude | VARCHAR(20) | NOT NULL | — |
| longitude | VARCHAR(20) | NOT NULL | — |
| foto_url | VARCHAR(500) | NOT NULL | URL ke Object Storage, contoh: `presensi/2026/06/absen_123.jpg` |
| status_kehadiran | ENUM('Hadir','Terlambat','Alpa') | **INDEXED** | — |

> **Peringatan:** Tabel ini akan menjadi tabel dengan baris terbanyak. **WAJIB** menggunakan composite index pada `(id_siswa, tanggal)`.

#### 7. Tabel `pengajuan_izin`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_izin | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_siswa | BIGINT UNSIGNED | FK → siswa.id, **INDEXED** | — |
| id_wali_murid | BIGINT UNSIGNED | FK → wali_murid.id | — |
| kategori | ENUM('Sakit','Acara','Lomba') | NOT NULL | — |
| tanggal_mulai | DATE | NOT NULL | — |
| tanggal_selesai | DATE | NOT NULL | — |
| bukti_dokumen_url | VARCHAR(500) | NULLABLE | URL ke Object Storage |
| status_approval | ENUM('Pending','Disetujui','Ditolak') | **INDEXED** | — |

#### 8. Tabel `jadwal_piket`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_jadwal | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| id_guru | BIGINT UNSIGNED | FK → guru.id, **INDEXED** | — |
| hari_tugas | ENUM('Senin','Selasa','Rabu','Kamis','Jumat') | **INDEXED** | — |

### C. Tabel Konfigurasi

#### 9. Tabel `pengaturan_jam_presensi`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_pengaturan | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| hari | ENUM('Senin','Selasa','Rabu','Kamis','Jumat','Sabtu') | NOT NULL | — |
| jam_buka_masuk | TIME | NOT NULL | Contoh: 06:00 — siswa mulai bisa presensi |
| batas_terlambat | TIME | NOT NULL | Contoh: 07:00 — lewat jam ini status 'Terlambat' |
| jam_tutup_masuk | TIME | NOT NULL | Contoh: 08:00 — presensi dikunci |

#### 10. Tabel `kalender_akademik`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id_libur | BIGINT UNSIGNED | PK, AUTO_INCREMENT | — |
| tanggal | DATE | NOT NULL | — |
| keterangan | VARCHAR(200) | NULLABLE | Contoh: "Idul Fitri", "Rapat Yayasan" |
| status_libur | BOOLEAN | NOT NULL | Aktif/Tidak |

---

## Strategi Index

```sql
-- Users
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_role ON users(role);

-- Siswa
CREATE INDEX idx_siswa_nisn ON siswa(nisn);
CREATE INDEX idx_siswa_nama ON siswa(nama);
CREATE INDEX idx_siswa_status ON siswa(status);

-- Presensi (tabel kritis)
CREATE INDEX idx_presensi_siswa_tanggal ON presensi(id_siswa, tanggal);
CREATE INDEX idx_presensi_tanggal ON presensi(tanggal);
CREATE INDEX idx_presensi_status ON presensi(status_kehadiran);

-- Pengajuan Izin
CREATE INDEX idx_izin_siswa ON pengajuan_izin(id_siswa);
CREATE INDEX idx_izin_status ON pengajuan_izin(status_approval);

-- Jadwal Piket
CREATE INDEX idx_piket_guru ON jadwal_piket(id_guru);
CREATE INDEX idx_piket_hari ON jadwal_piket(hari_tugas);
```

---

## Catatan Penting

1. **Tidak ada BLOB/Base64 di database** — foto dan dokumen hanya disimpan sebagai URL ke Object Storage (S3-compatible)
2. **Composite index** pada `presensi(id_siswa, tanggal)` sangat krusial untuk performa query rekap harian/bulanan
3. **Enum vs Integer** — menggunakan ENUM karena nilai terbatas dan lebih mudah dibaca langsung dari database
4. **Soft deletes** — pertimbangkan menambahkan kolom `deleted_at` untuk tabel master (siswa, guru, kelas)
