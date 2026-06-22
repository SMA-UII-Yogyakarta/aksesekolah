### **A. Tabel Inti (Autentikasi & Master)**

**1\. Tabel users (Portal SSO)**

* id (PK)  
* username (VARCHAR, **INDEXED**, unik, untuk login via NISN/NIP)  
* email (VARCHAR, unik, *Nullable*)  
* password (VARCHAR, hashed)  
* role (ENUM: 'admin', 'siswa', 'guru', 'walimurid')

**2\. Tabel wali\_murid**

* id\_wali\_murid (PK)  
* id\_user (FK ke users)  
* nama (VARCHAR)  
* no\_hp (VARCHAR, *Nullable*)  
* alamat (TEXT, *Nullable*)

**3\. Tabel guru**

* id\_guru (PK)  
* id\_user (FK ke users)  
* nama (VARCHAR)  
* kode\_guru (VARCHAR, unik)

**4\. Tabel kelas**

* id\_kelas (PK)  
* id\_guru (FK ke guru \=\> *Sebagai Wali Kelas*)  
* nama\_kelas (VARCHAR, misal: "X-A Reguler")

**5\. Tabel siswa**

* id\_siswa (PK)  
* id\_user (FK ke users)  
* id\_kelas (FK ke kelas)  
* id\_wali\_murid (FK ke wali\_murid)  
* nis (VARCHAR, unik)  
* nisn (VARCHAR, **INDEXED**, unik)  
* nama (VARCHAR, **INDEXED**)  
* tanggal\_lahir (DATE)  
* no\_hp (VARCHAR, *Nullable*)  
* alamat (TEXT, *Nullable*)  
* tahun\_angkatan (YEAR)  
* status (ENUM: 'Aktif', 'Non-Aktif', **INDEXED**)

  ### 

  ### **B. Tabel Transaksional (Aktivitas Harian)**

**6\. Tabel presensi (Tabel Paling Berat)**

* id\_presensi (PK)  
* id\_siswa (FK ke siswa, **INDEXED**)  
* tanggal (DATE, **INDEXED**)  
* waktu\_masuk (TIME)  
* latitude (VARCHAR)  
* longitude (VARCHAR)  
* foto\_url (VARCHAR \=\> *Hanya menyimpan link URL file di Object Storage, misal: 'presensi/2026/06/absen\_123.jpg'*)  
* status\_kehadiran (ENUM: 'Hadir', 'Terlambat', 'Alpa', **INDEXED**)

**7\. Tabel pengajuan\_izin**

* id\_izin (PK)  
* id\_siswa (FK ke siswa, **INDEXED**)  
* id\_wali\_murid (FK ke wali\_murid)  
* kategori (ENUM: 'Sakit', 'Acara', 'Lomba')  
* tanggal\_mulai (DATE)  
* tanggal\_selesai (DATE)  
* bukti\_dokumen\_url (VARCHAR \=\> *Link URL file nota dokter/surat di Object Storage*)  
* status\_approval (ENUM: 'Pending', 'Disetujui', 'Ditolak', **INDEXED**)

**8\. Tabel jadwal\_piket**

* id\_jadwal (PK)  
* id\_guru (FK ke guru, **INDEXED**)  
* hari\_tugas (ENUM: 'Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat', **INDEXED**)

  ### **C. Tabel Konfigurasi (Kalender)**

**9\. Tabel pengaturan\_jam\_presensi**

* id\_pengaturan (PK)  
* hari (Enum: 'Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat', 'Sabtu')  
* jam\_buka\_masuk (TIME \=\> Misal: 06:00 WIB. Siswa baru bisa menekan tombol absen setelah jam ini).  
* batas\_terlambat (TIME \=\> Misal: 07:00 WIB. Jika absen pukul 07:01, otomatis statusnya 'Terlambat').  
* jam\_tutup\_masuk (TIME \=\> Misal: 08:00 WIB. Setelah jam ini, tombol absen masuk otomatis terkunci/hilang).

**10\. Tabel kalender\_akademik**

* id\_libur (PK)  
* tanggal (DATE)  
* keterangan (VARCHAR, *Nullable* \=\> Misal: "Idul Fitri", "Rapat Yayasan")  
* status\_libur (Boolean/Enum \=\> Aktif/Tidak)