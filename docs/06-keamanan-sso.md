# Keamanan & Single Sign-On (SSO)

## Arsitektur SSO

SMART Absen SMA UII menggunakan pendekatan **Single Sign-On (SSO) mandiri** yang dibangun di atas Laravel. Sistem ini dirancang sebagai **Identity Provider (IdP)** pertama di lingkungan yayasan, yang kelak akan menjadi standar autentikasi bagi seluruh aplikasi SMA UII.

### Konsep Identity Provider

```
┌─────────────────────────────────────────────────────────┐
│                   Identity Provider                      │
│                      (SMART Absen)                       │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  User Store   │  │  Token       │  │  Role        │  │
│  │  (users tbl)  │  │  Management  │  │  Resolver    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            SSO Session (Sanctum)                   │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│ Aplikasi   │ │ Aplikasi   │ │ Aplikasi   │
│ e-Learning │ │ Keuangan   │ │ Perpus-    │
│ (future)   │ │ (future)   │ │ takaan     │
│            │ │            │ │ (future)   │
└────────────┘ └────────────┘ └────────────┘
```

### Teknologi yang Digunakan

**Laravel Sanctum + Spatie Laravel Permission**

| Alasan | Detail |
|---|---|
| **Sederhana** | API token berbasis database, cocok untuk SPA dan mobile |
| **Stateful** | Session-based auth untuk Inertia pages |
| **Stateless** | API token untuk aplikasi third-party di masa depan |
| **Built-in** | Sanctum dan Spatie terintegrasi native dengan Laravel |
| **RBAC** | Spatie menyediakan role, permission, team, dan blade directive |

---

## Role-Based Access Control (RBAC)

### Role Hierarchy

```
┌─────────────────────────────────────────┐
│                SUPER ADMIN               │
│  (Akses penuh seluruh sistem + bypass)   │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│                  ADMIN                   │
│  (CRUD master data, pengaturan, rekap)  │
└─────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌──────────────┐    ┌──────────────────────┐
│  Wali Kelas  │    │    Guru Piket        │
│ (verifikasi  │    │ (monitoring realtime)│
│  izin, rekap)│    └──────────────────────┘
└──────┬───────┘
       │
       ▼ (tidak ada hirarki, role terpisah)
┌──────────────┐    ┌──────────────────────┐
│    Siswa     │    │   Wali Murid         │
│  (presensi)  │    │ (pantau anak + izin) │
└──────────────┘    └──────────────────────┘
```

### Implementasi dengan Spatie Laravel Permission

```bash
# Instalasi
bun add composer spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

```php
// Definisi role & permission di seeder / Service Provider
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

// Buat permission
Permission::create(['name' => 'create-presensi']);
Permission::create(['name' => 'view-laporan']);
Permission::create(['name' => 'manage-user']);
Permission::create(['name' => 'manage-kelas']);
Permission::create(['name' => 'approve-izin']);

// Buat role + assign permission
$superAdmin = Role::create(['name' => 'super-admin']); // all permissions via gate
$admin = Role::create(['name' => 'admin']);
$admin->givePermissionTo(['create-presensi', 'view-laporan', 'manage-user', 'manage-kelas', 'approve-izin']);

$guru = Role::create(['name' => 'guru']);
$guru->givePermissionTo(['create-presensi', 'view-laporan', 'approve-izin']);

$siswa = Role::create(['name' => 'siswa']);
$siswa->givePermissionTo(['create-presensi']);

$waliMurid = Role::create(['name' => 'wali-murid']);
$waliMurid->givePermissionTo(['view-laporan', 'approve-izin']);
```

```php
// Penggunaan di Route
use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum', 'role:admin|super-admin'])->group(function () {
    Route::resource('users', UserController::class);
});

Route::middleware(['auth:sanctum', 'permission:create-presensi'])->group(function () {
    Route::post('/presensi', [PresensiController::class, 'store']);
});
```

```php
// Penggunaan di Blade/Inertia
@role('admin')
    <!-- tampilkan admin panel -->
@endrole

@can('view-laporan')
    <!-- tampilkan tombol laporan -->
@endcan
```

---

## Triple-Layer Validation Presensi

Sebelum sistem menyimpan data kehadiran, server WAJIB melakukan tiga lapis validasi:

### Layer 1: Validasi Kalender Akademik

```php
// Cek apakah tanggal hari ini ada di tabel kalender_akademik
$isHoliday = KalenderAkademik::where('tanggal', today())
    ->where('status_libur', true)
    ->exists();

if ($isHoliday) {
    return response()->json([
        'status' => 'rejected',
        'message' => 'Hari ini adalah hari libur akademik'
    ], 403);
}
```

### Layer 2: Validasi Hari Aktif

```php
// Cek apakah hari ini adalah hari aktif
$hariIni = today()->format('l'); // Monday, Tuesday, dll.

$pengaturan = PengaturanJamPresensi::where('hari', $hariIni)->first();

if (! $pengaturan) {
    return response()->json([
        'status' => 'rejected',
        'message' => 'Tidak ada jadwal presensi untuk hari ini'
    ], 403);
}
```

### Layer 3: Validasi Rentang Jam

```php
$waktuSekarang = now()->format('H:i:s');

if ($waktuSekarang < $pengaturan->jam_buka_masuk) {
    // Belum waktunya presensi
    return response()->json(['status' => 'too_early'], 403);
}

if ($waktuSekarang <= $pengaturan->batas_terlambat) {
    $status = 'Hadir';
} elseif ($waktuSekarang <= $pengaturan->jam_tutup_masuk) {
    $status = 'Terlambat';
} else {
    // Presensi ditutup
    return response()->json(['status' => 'closed'], 403);
}
```

### Diagram Validasi

```
[Request Presensi Masuk]
        │
        ▼
┌───────────────────┐   TIDAK   ┌──────────────────┐
│  Cek Kalender     │───────────│  Tolak: "Hari    │
│  Akademik         │           │  Libur Akademik" │
└────────┬──────────┘           └──────────────────┘
         │ YA
         ▼
┌───────────────────┐   TIDAK   ┌──────────────────┐
│  Cek Hari Aktif   │───────────│  Tolak: "Tidak   │
│  (ada di         │           │  Ada Jadwal"     │
│  pengaturan?)    │           └──────────────────┘
└────────┬──────────┘
         │ YA
         ▼
┌──────────────────────────────────────────────────┐
│              Cek Rentang Jam                      │
├──────────┬──────────┬──────────┬──────────────────┤
│ < Buka   │ Buka s/d │ Terlambat│ > Tutup          │
│          │ Terlambat│ s/d Tutup│                   │
├──────────┼──────────┼──────────┼──────────────────┤
│ "Terlalu │ "Hadir"  │ "Terlam- │ "Presensi        │
│  Dini"   │          │  bat"    │ Ditutup"         │
└──────────┴──────────┴──────────┴──────────────────┘
```

---

## Prinsip Keamanan Lain

### 1. Konfigurasi Dinamis (Anti-Hardcode)

```
❌ DILARANG:
if ($jamSekarang < 6 && $jamSekarang > 8) { ... }

✅ DIVAJIBKAN:
$pengaturan = PengaturanJamPresensi::where('hari', today()->dayName)->first();
if ($waktuSekarang < $pengaturan->jam_buka_masuk) { ... }
```

### 2. Tokenisasi & Session

| Aspek | Metode |
|---|---|
| Web (Browser) | Session-based via Laravel cookie |
| API (Mobile future) | Sanctum token |
| CSRF Protection | Laravel built-in CSRF token |
| Password hashing | Bcrypt (12 rounds) |

### 3. Proteksi URL & Data

- Semua route dibungkus middleware `auth:sanctum` + `role:xxx`
- Policy/Guard untuk operasi spesifik (misal: siswa hanya bisa presensi untuk dirinya sendiri)
- Validasi server-side untuk semua input
- Rate limiting pada endpoint presensi untuk mencegah spam

### 4. Object Storage Security

- File diupload langsung ke Object Storage (bukan via server Laravel)
- URL file bersifat signed/temporary (expired dalam waktu tertentu)
- Validasi tipe file di server sebelum menyimpan URL

### Kelebihan & Kekurangan Web-Based

| Aspek | Kelebihan | Kekurangan |
|---|---|---|
| Instalasi | Tanpa install, cukup browser | — |
| Update | Instant, tanpa download | — |
| Offline | — | Tidak bisa presensi tanpa internet |
| Izin perangkat | — | Pengguna awam bisa blokir kamera/lokasi |
