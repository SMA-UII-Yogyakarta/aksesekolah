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
│  │            SSO Session (Sanctum/Passport)         │   │
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

**Rekomendasi: Laravel Sanctum**

| Alasan | Detail |
|---|---|
| **Sederhana** | API token berbasis database, cocok untuk SPA dan mobile |
| **Stateful** | Bisa pakai session-based auth untuk web tradisional |
| **Stateless** | API token untuk aplikasi third-party di masa depan |
| **Built-in** | Sudah termasuk di Laravel, tidak perlu package tambahan |

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

### Implementasi Middleware

```php
// Di Laravel, buat middleware untuk tiap role:

class RoleMiddleware
{
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user() || $request->user()->role !== $role) {
            abort(403, 'Unauthorized');
        }
        return $next($request);
    }
}

// Registrasi di Kernel / Route:
// Route::middleware('role:admin')->group(...);
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
| API (Mobile future) | Sanctum token / Passport |
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
