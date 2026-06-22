# Agent Rules — SMART Absen SMA UII

## Aturan Umum
- Gunakan bahasa Indonesia dalam percakapan dan dokumentasi (kecuali kode)
- Jawab dengan ringkas, langsung ke titik — tidak perlu preamble/penjelasan panjang
- Jika diminta membuat kode, ikuti pattern yang sudah ada di project
- Jangan menambahkan komentar di kode kecuali diminta
- Jangan hapus informasi atau kredit yang sudah ada
- Jika ragu tentang arsitektur, tanyakan dulu sebelum coding

## Konvensi Commit
```
feat:      Fitur baru
fix:       Perbaikan bug
docs:      Dokumentasi
chore:     Maintenance (dependensi, CI, config)
refactor:  Refaktor kode (tanpa perubahan fungsional)
style:     Perbaikan coding style (Pint otomatis)
test:      Penambahan test
```

## Branching
```
main → develop → feature/nama-fitur
```
- `main`: production — protected, hanya via PR
- `develop`: integration branch
- `feature/*`: untuk development harian

## Laravel Conventions
- Route: web.php untuk Blade, api.php untuk API Sanctum
- Controller: single action prefer method `__invoke` untuk simple case
- Validation: Form Request class
- Database: migration + seeder, jangan langsung edit SQL
- Eloquent: eager loading untuk mencegah N+1
- Config: semua environment variable via `.env`, bukan hardcode

## Keamanan
- Jangan commit `.env`, key, token, password
- Jangan expose debug info di production
- Gunakan prepared statements (via Eloquent)
- Input user harus divalidasi & di-sanitize
