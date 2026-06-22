**1\. Alur Program (Skenario Pengguna)**  
Sistem ini menggunakan alur berbasis peran (*Role-Based Access Control*) dengan pintu masuk tunggal melalui SSO.

* **Pintu Masuk (SSO Login):** Semua pengguna (Admin, Guru, Siswa, Wali Murid) masuk melalui satu portal login institusi. Sistem akan otomatis mendeteksi peran pengguna dan mengarahkan mereka ke *dashboard* masing-masing.  
* **Alur Murid:** Login \=\> Buka menu Presensi \=\> Sistem meminta akses Kamera & Lokasi \=\> Ambil swafoto \=\> Sistem mencatat waktu & koordinat \=\> Selesai.  
* **Alur Wali Murid:** Login \=\> Buka Dashboard untuk melihat grafik kehadiran anak \=\> Jika anak berhalangan, buka menu Pengajuan Izin \=\> Isi form (Sakit/Acara/Lomba), durasi, dan unggah bukti/surat \=\> Kirim pengajuan.  
* **Alur Wali Kelas:** Login \=\> Buka notifikasi izin masuk \=\> Verifikasi dan Update status izin siswa \=\> Pantau rekap harian kelas yang diampunya.  
* **Alur Guru Piket:** Login \=\> Buka menu Monitoring Harian \=\> Filter berdasarkan kelas yang sedang dipantau \=\> Lihat daftar siswa yang belum presensi/absen.  
* **Alur Admin:** Login \=\> Mengelola data master (Siswa, Guru, Wali, Kelas) secara *bulk* maupun satuan \=\> Mengelola rekap kehadiran dan izin secara penuh \=\> Menetapkan jadwal buka/tutup gerbang presensi digital dan mendaftarkan kalender libur akademik.

**2\. Jumlah Total Interface (Antarmuka)**  
Aplikasi akan memiliki rancangan desain responsif (berjalan optimal di Mobile & Desktop) dengan total sekitar 13 Antarmuka Utama:

* **Halaman Login Universal:** Portal utama integrasi SSO.  
* **Dashboard Admin:** Ringkasan statistik sekolah (total presensi hari ini, grafik absensi).  
* **Manajemen Data Master (Admin):** Modul CRUD untuk data Siswa, Guru, dan Wali Murid (termasuk fitur *Upload/Download Excel* untuk *bulk data*).  
* **Manajemen Kelas & Enrolment (Admin):** Interface untuk memetakan Wali Kelas dan memasukkan siswa ke kelas masing-masing.  
* **Pengaturan Waktu & Libur (Admin):** Antarmuka khusus untuk mengatur jam buka, batas terlambat, jam tutup presensi harian, serta mendaftarkan tanggal merah/hari libur akademik.  
* **Dashboard Siswa:** Ringkasan kehadiran pribadi.  
* **Form Presensi Live (Siswa):** Halaman khusus memuat antarmuka kamera (*viewfinder*) dan peta geolokasi.  
* **Dashboard Wali Murid:** Laporan presensi spesifik untuk putra/putrinya.  
* **Form Pengajuan Izin (Wali Murid):** Form input data izin beserta fitur unggah *file*.  
* **Dashboard Wali Kelas:** Laporan absensi khusus untuk satu kelas.  
* **Panel Verifikasi Izin (Wali Kelas):** Antarmuka untuk menyetujui/menolak izin dari Wali Murid.  
* **Dashboard Guru Piket:** Layar monitoring presensi *real-time* dengan *filter* kelas/hari.  
* **Laporan & Export (Global):** Antarmuka khusus untuk mencetak rekap Harian/Bulanan/Semester ke format PDF/Excel (bisa diakses Admin, Wali Kelas, dan Guru Piket).

**3\. Tools di Setiap Interface**

* **Form Presensi Live:** Menggunakan HTML5 *Geolocation API* untuk mendapatkan titik koordinat (Latitude/Longitude) dan *WebRTC/MediaDevices API* untuk mengambil foto langsung dari kamera perangkat (menghindari unggah foto dari galeri).  
* **Manajemen Data Master:** Menggunakan *library DataTables* untuk fitur *search, sort*, dan *pagination* data yang besar, serta *tools parser* untuk *import/export bulk data*.  
* **Dashboard & Rekap:** Menggunakan pustaka visualisasi data (seperti *Chart.js* atau *ApexCharts*) untuk menampilkan grafik tren kehadiran.  
* **Form Izin:** Komponen *File Uploader* dengan validasi tipe dokumen (JPG, PNG, PDF) dan batasan ukuran *file*.  
* **Frontend Image Compression:** Implementasi fungsi kompresi gambar berbasis JavaScript (*Canvas API*) pada antarmuka *Form Presensi Live* untuk mengecilkan ukuran swafoto hingga maksimal 200 KB tepat sebelum dikirim ke *server*.  
* **External Object Storage (S3 Protocol):** Sistem akan menggunakan cloud storage terpisah yang kompatibel dengan protokol S3 khusus untuk menyimpan fail media statis. Ini mencakup fail foto absensi serta unggahan bukti surat pada Form Pengajuan Izin.

**4\. Spesifikasi Infrastruktur & Kinerja (Performance)**

* **Target Konkurensi (CCU):** Infrastruktur peladen (*server*) berbasis VPS (*Virtual Private Server*) HARUS MAMPU dan dioptimalkan untuk melayani antrean hingga 750 pengguna bersamaan (*Concurrent Users*) selama rentang waktu kritis presensi pagi (06:30 \- 07:00 WIB) tanpa mengalami *Gateway Timeout*.  
* **Optimasi Basis Data Berbasis Index:** Mesin basis data utama (MySQL/PostgreSQL) diwajibkan menggunakan struktur *Database Indexing* pada kolom-kolom relasional dan pencarian utama (seperti id\_siswa, tanggal, dan status\_kehadiran).  
* **Pemisahan Beban Basis Data (Storage Offloading):** Basis data dilarang keras menyimpan fail gambar dalam bentuk tipe data BLOB atau *string LongText* (Base64). Basis data murni difungsikan untuk menyimpan URL yang menunjuk ke lokasi fail di Object Storage eksternal.

**5\. Keamanan Logika & Pembatasan Waktu (Time Geofencing)**

* **Konfigurasi Dinamis (Anti-Hardcode):** Sistem tidak diperkenankan menggunakan logika waktu statis (*hardcode*) di dalam *source code*. Seluruh jam buka, jam tutup, dan batas keterlambatan harus ditarik dari tabel basis data pengaturan\_jam\_presensi agar fleksibel terhadap perubahan jadwal sekolah.  
* **Validasi Tiga Lapis (Triple-Layer Validation):** Sebelum menyimpan data kehadiran, *backend* wajib melakukan tiga lapis pengecekan:  
  1. *Validasi Kalender Akademik:* Menolak presensi jika tanggal sistem cocok dengan data hari libur di tabel kalender\_akademik.  
  2. *Validasi Hari Aktif:* Menolak presensi pada akhir pekan (Sabtu/Minggu), kecuali diatur aktif oleh Admin.  
  3. *Validasi Rentang Jam:* Menolak akses jika belum masuk jam buka, merekam 'Hadir' jika tepat waktu, merekam 'Terlambat' jika melewati batas toleransi, dan mengunci akses (*Disabled*) jika melewati jam tutup operasional.

**6\. Kelebihan & Kekurangan (Web-Based vs Native App)**

* **Kelebihan Web-Based (Mobile Responsive):**  
  * **Tanpa Instalasi:** Pengguna tidak perlu mengunduh aplikasi dari Play Store/App Store yang memakan memori internal *smartphone*. Cukup buka *browser* dan masuk via SSO.  
  * **Pembaruan Instan:** Setiap ada perbaikan atau fitur baru, pengguna langsung mendapatkan versi terbaru tanpa perlu melakukan *update* aplikasi.  
* **Kekurangan:**  
  * **Ketergantungan Browser & Internet:** Tidak memiliki mode *offline* yang sekuat aplikasi *native*. Jika sinyal internet terputus saat mengambil geolokasi, presensi bisa gagal submit.  
  * **Izin Perangkat (Permissions):** Terkadang pengguna awam secara tidak sengaja memblokir izin akses lokasi/kamera di *browser* mereka, sehingga membutuhkan edukasi tambahan.**6\. Rencana Anggaran Biaya**

| Komponen Pengerjaan | Deskripsi Singkat | Estimasi Biaya |
| :---- | :---- | :---- |
| **Arsitektur Database & SSO** | Perancangan ERD, relasi multi-user, dan pembuatan fondasi login SSO mandiri. | Rp 1.800.000 |
| **Modul Presensi Utama** | Pengembangan fitur akses kamera (swafoto) dan penguncian titik koordinat (GPS) siswa. | Rp 1.500.000 |
| **Modul Admin & Master Data** | Pembuatan antarmuka CRUD (termasuk fitur import/export Excel) untuk data Siswa, Guru, dan Kelas. | Rp 1.000.000 |
| **Modul Guru Piket & Wali Kelas** | Pembuatan filter monitoring real-time, sistem verifikasi izin, dan rekap kelas. | Rp 1.000.000 |
| **Modul UI/UX Siswa & Wali Murid** | Pembuatan dashboard responsif, grafik kehadiran, dan form pengajuan izin. | Rp 1.150.000 |
| **Sewa *Object Storage Eksternal*** | Sewa Object Storage Eksternal (Kapasitas 50 GB \- 250 GB) Tahunan | Rp 850.000 |
| **Setup & Deployment Server Internal** | Konfigurasi, instalasi database, dan deployment aplikasi ke server mandiri milik sekolah. | Rp 1.200.000 |
| **TOTAL ESTIMASI** |  | **Rp 8.500.000** |

### **7\. Timeline Pengerjaan (Target 8 Minggu / 2 Bulan)**

|                                                                                                                    Minggu Ke Pekerjaan | Minggu Ke 1 | Minggu Ke 2 | Minggu Ke 3 | Minggu Ke 4 | Minggu Ke 5 | Minggu Ke 6 | Minggu Ke 7 | Minggu Ke 8 |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **Analisis Kebutuhan** |   |   |   |   |   |   |   |   |
| **Wireframing (Desain UI/UX).** |   |   |   |   |   |   |   |   |
| **Desain ERD Database** |   |   |   |   |   |   |   |   |
| **Integrasi SSO** |   |   |   |   |   |   |   |   |
| **Pengaturan Server/Environment** |   |   |   |   |   |   |   |   |
| **Pembuatan Dashboard Admin (CRUD Master Data).** |   |   |   |   |   |   |   |   |
| **Pengembangan Modul Kelas & Enrolment (Relasi Siswa, Guru, Wali Kelas)** |   |   |   |   |   |   |   |   |
| **Presensi Siswa (Geolokasi & Kamera)** |  |  |  |  |  |  |  |  |
| **Pengembangan Modul Guru Piket dan Verifikasi Wali Kelas.** |   |   |   |   |   |   |   |   |
| **Penyatuan Sistem** |   |   |   |   |   |   |   |   |
| **Pembuatan Fitur Laporan/Export (Harian/Bulanan/Semester).** |   |   |   |   |   |   |   |   |
| **Internal Testing** |   |   |   |   |   |   |   |   |
| **Optimalisasi Tampilan Mobile.** |   |   |   |   |   |   |   |   |
| **Bug Fixing** |   |   |   |   |   |   |   |   |
| **User Acceptance Testing (UAT)** |   |   |   |   |   |   |   |   |
| **Deployment ke server produksi** |   |   |   |   |   |   |   |   |
| **Pelatihan/Panduan Pengguna.** |   |   |   |   |   |   |   |   |

### 

### **8\. Kemungkinan Update di Masa Depan (*Future Roadmap*)**

Sistem presensi ini diproyeksikan sebagai fondasi awal transformasi digital terpusat bagi yayasan, dengan peta jalan pengembangan sebagai berikut:

* **Fase 1: Pembentukan SSO Yayasan Mandiri** Sistem *login* dari aplikasi ini akan dijadikan portal identitas utama (*Identity Provider*). Kedepannya, semua aplikasi sekolah (e-Learning, Keuangan, Perpustakaan) cukup menggunakan satu akun SSO dari sistem ini.  
* **Fase 2: Tonggak "UII Satu Data"** Mewujudkan sentralisasi data untuk mencegah data ganda antar unit. Seluruh rekam jejak siswa dan guru (presensi, akademik, administrasi) akan terintegrasi pada satu ID.  
* **Fase 3: Automasi & Teknologi Cerdas** *Geofencing* yang lebih ketat (pembatasan radius lokasi otomatis), dan *WhatsApp Gateway* untuk notifikasi absensi *real-time* ke wali murid.

