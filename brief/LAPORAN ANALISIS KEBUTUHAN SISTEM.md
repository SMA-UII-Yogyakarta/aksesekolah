# **LAPORAN ANALISIS KEBUTUHAN SISTEM**

**Nama Proyek:** SMART ABSEN SMA UII

**Target Arsitektur:** Web-Based (Mobile Responsive & Desktop)

**Teknologi Utama:** PHP Laravel & RDBMS (PostgreSQL/MariaDB/MySQL)

**Proyeksi Masa Depan:** Gerbang Utama Single Sign-On (SSO) & Fondasi "UII Satu Data"

1. ## **DESKRIPSI UMUM SISTEM**

Sistem Informasi Presensi Siswa Terintegrasi adalah aplikasi berbasis web responsif yang dirancang untuk mengotomatisasi, mengamankan, dan mensentralisasi pencatatan kehadiran siswa di lingkungan instansi sekolah di bawah naungan yayasan.

Sistem ini mengintegrasikan teknologi pelacakan lokasi (*Geolocation API*) dan pengambilan gambar biometrik (*Webcam*) di sisi klien yang dioptimalkan untuk menangani beban lalu lintas tinggi (*high concurrency traffic*) hingga 760+ siswa secara bersamaan pada jam masuk sekolah. Sistem ini menggunakan pintu masuk tunggal (*Single Sign-On*) sebagai fondasi awal standarisasi identitas digital terpusat bagi seluruh ekosistem aplikasi yayasan di masa mendatang.

2. ## **KEBUTUHAN FUNGSIONAL** 

Kebutuhan fungsional diatur berdasarkan peran pengguna (*Role-Based Access Control*) dan modul inti sistem:

1. ### **Modul Pintu Masuk (Universal SSO Portal)**

* Sistem harus menyediakan satu portal autentikasi (*login*) tunggal untuk seluruh kategori pengguna.  
* Sistem harus secara otomatis mendeteksi peran akun (*role*) setelah proses login berhasil, kemudian mengarahkan pengguna ke *dashboard* masing-masing (*Admin, Guru, Siswa,* atau *Wali Murid*).  
* Arsitektur modul *login* ini harus dirancang sebagai *Identity Provider* (IdP) menggunakan standar Laravel (Passport/Sanctum) agar dapat digunakan oleh aplikasi yayasan lainnya di masa depan.

2. ### **Modul Hak Akses: Admin (Pengelola Pusat)**

* **Manajemen Data Master (CRUD Satuan & Bulk):** Sistem harus mampu melakukan manajemen data (Buat, Baca, Ubah, Hapus) untuk entitas Siswa, Guru, dan Wali Murid, baik secara satuan maupun massal melalui fitur *Upload/Download* template file Excel.  
* **Enrolment Kelas & Pemetaan Akademik:** Sistem harus menyediakan fungsi untuk membuat Kelas, menunjuk 1 Guru sebagai Wali Kelas (*One-to-Many* terhadap siswa), dan memasukkan daftar siswa ke dalam kelas tersebut secara *bulk*.  
* **Manajemen Presensi & Izin (*Bypass System*):** Admin memiliki hak akses penuh untuk melakukan CRUD pada data kehadiran dan izin harian, bulanan, maupun semester sebagai fungsi koreksi/override jika terjadi kendala teknis.  
* **Dashboard Analitik Lanjut (*Drill-Down Analytics*):** Dashboard Admin harus menampilkan statistik kehadiran *real-time* yang bersifat interaktif dengan hirarki:  
  * *Level 1:* Grafik Statistik Keseluruhan Sekolah.  
  * *Level 2:* Grafik Statistik per Kelas.  
  * *Level 3:* Statistik per Siswa di dalam kelas tersebut.  
  * *Level 4:* Detail Kehadiran *by Name* (Harian, Bulanan, Semester).

3. ### **Modul Hak Akses: Siswa**

* **Presensi Live (Kamera & GPS):** Sistem harus menyediakan antarmuka presensi yang mewajibkan siswa mengambil foto diri (*selfie*) dan mengunci titik koordinat geolokasi (Latitude & Longitude) secara *real-time*.  
* **Laporan Kehadiran Pribadi:** Menyediakan grafik dan rekapitulasi kehadiran mandiri yang dapat difilter berdasarkan rentang Harian, Bulanan, dan Semester.

4. ### **Modul Hak Akses: Wali Murid**

* **Aturan Bisnis Relasi (*One-to-Many*):** Satu akun Wali Murid dapat mengampu lebih dari satu siswa (kakak-beradik) dalam instansi yang sama.  
* **Multi-Profil Anak:** Dashboard Wali Murid wajib menyediakan fitur *switch-profile* (pilihan anak) untuk melihat rekap presensi masing-masing anak secara terpisah (Harian, Bulanan, Semester).  
* **Pengajuan Izin Digital:** Menyediakan formulir pengajuan izin tidak masuk sekolah (Kategori: Sakit, Acara, Lomba) dengan kewajiban memilih profil anak yang dimaksud, menentukan durasi hari, serta mengunggah dokumen bukti fisik (seperti surat dokter/surat tugas) dalam format gambar/PDF.

5. ### **Modul Hak Akses: Wali Kelas**

* **Aturan Bisnis Relasi (*One-to-Many*):** Satu Wali Kelas bertanggung jawab penuh atas rekapitulasi data banyak siswa di dalam satu ruang lingkup kelas yang diampunya.  
* **Panel Verifikasi Izin:** Sistem harus menampilkan *kotak masuk* notifikasi pengajuan izin dari Wali Murid yang berada di bawah perwaliannya, untuk kemudian disetujui (*Approve*) atau ditolak (*Reject*).  
* **Monitoring & Rekap Kelas:** Wali Kelas dapat membaca dan mengeksplorasi laporan kehadiran seluruh siswa di kelasnya secara Harian, Bulanan, maupun Semester.

6. ### **Modul Hak Akses: Guru Piket**

* **Live Monitoring Filtered:** Guru piket dapat melihat rekapitulasi kehadiran seluruh siswa sekolah pada hari berjalan, yang dilengkapi dengan fitur *filter cepat* berdasarkan kelas untuk mempermudah pengecekan fisik di lapangan.

3. ## **KEBUTUHAN NON-FUNGSIONAL** 

1. ### **Performa & Manajemen Konkurensi Jaringan (*Performance*)**

* **Optimasi Payload Gambar (Frontend Compression):** Antarmuka presensi siswa wajib mengintegrasikan skrip kompresi di sisi klien (*Client-Side Rendering*) menggunakan JavaScript (HTML5 Canvas/WebcamJS) untuk membatasi resolusi tangkapan kamera pada ukuran 320x240 piksel dengan kualitas JPEG 90%.  
* **Batasan Ukuran File:** Hasil konversi foto harus berbentuk teks Base64 dengan ukuran file **maksimal \< 20 KB** per siswa. Hal ini bertujuan agar total data yang masuk saat 760 siswa melakukan presensi serentak tidak melebihi **\~15,2 MB**, sehingga menghemat *bandwidth* internal sekolah.  
* **Tuning Database:** Pengaturan *web server* dan database relasional pada *server* internal instansi wajib menaikkan parameter koneksi maksimal (`max_connections`) minimal di angka **1000 koneksi serentak** untuk menghindari kegagalan sistem (*connection timeout* / *Too Many Connections*) pada jam sibuk (06:30 \- 07:00 WIB).

2. ### **Desain Antarmuka (*Usability*)**

* **Responsive Web Design:** Tampilan aplikasi harus menyesuaikan ukuran layar perangkat secara fleksibel. Penggunaan oleh *Siswa* dan *Wali Murid* dioptimalkan untuk tampilan mobile (*Smartphone*), sedangkan untuk *Admin, Wali Kelas,* dan *Guru Piket* dioptimalkan untuk tampilan layar Desktop/Tablet.

3. ### **Keamanan & Autentikasi (*Security*)**

* **Sesi Akses Aman:** Autentikasi SSO didukung oleh sistem tokenisasi yang aman untuk mencegah manipulasi sesi login.  
* **Restriksi Hak Akses:** Sistem harus memastikan bahwa pengguna tidak dapat mengakses URL atau memanipulasi data di luar hak akses peran (*role*) mereka melalui proteksi *middleware* di tingkat kode Laravel.

4. ### **Keandalan & Ekspor Data (*Reliability & Output*)**

* **Kompatibilitas Ekspor Laporan:** Seluruh laporan rekapitulasi (Harian, Bulanan, Semester) pada level Admin, Wali Kelas, dan Guru Piket harus dapat diunduh secara akurat ke dalam format dokumen universal, yaitu **PDF** (untuk dokumen cetak resmi) dan **Excel** (untuk pengolahan data lanjutan).  
* **Kesiapan Server Internal:** Sistem harus dapat berjalan stabil pada infrastruktur *server* lokal/mandiri milik instansi dengan sistem penyimpanan berbasis SSD/NVMe untuk menjamin kecepatan tulis data (*I/O operations*) dokumen Base64 gambar presensi.