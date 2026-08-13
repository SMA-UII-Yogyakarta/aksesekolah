# 10. Design System, Storybook & Porting Multi-Platform

> **Panduan Desain Terpadu, Aksesibilitas WCAG 2.1 AA, dan Peta Jalan Porting Aplikasi Multi-Platform SMA UII Yogyakarta**

---

## 🎯 1. Filosofi & Kekayaan Intelektual (Intellectual Property)

Design System SMA UII dirancang sebagai **aset kekayaan intelektual terpadu** yang menjaga konsistensi identitas visual sekolah pada seluruh kanal digital (*web portal, mobile application, anjungan presensi fisik, hingga e-learning Moodle*).

### Nilai Utama:
1. **Single Source of Truth:** Token desain di Figma ditransformasikan menjadi CSS/Theme variables dan komponen React yang terdokumentasi di **Storybook**.
2. **Kepatuhan Aksesibilitas (WCAG 2.1 AA):** Setiap komponen diuji kontras warnanya, navigasi keyboard (tabbing/focus), dan semantic HTML.
3. **Kesiapan Pemisahan Arsitektur (*Decoupled Architecture*):** `aksesekolah` bertindak sebagai *master entrypoint* yang mengoordinasikan modul:
   * **`apps/backend` (`core.git`):** Headless API, Central IdP/Dapodik, Database & S3 Storage.
   * **`apps/frontend` (`webapp.git`):** Frontend Web terpisah (Next.js / Vite React).
   * **`apps/mobile` (`flutter.git` / React Native):** Mobile App Siswa & Wali Murid.

---

## 📖 2. Storybook Component Showcase

Komponen UI diisolasi dan didokumentasikan menggunakan **Storybook** di dalam repositori `core` (dan nantinya dipaketkan sebagai package library bersama di `aksesekolah/packages/ui`):

```bash
# Menjalankan Storybook interaktif (Port 6006)
bun run storybook

# Membangun bundle statis Storybook
bun run build-storybook
```

### Katalog Komponen Utama:
* **Atoms:** `Button`, `Badge`, `StatusBadge`, `Input`, `Label`, `Toggle`, `Checkbox`, `Radio`, `FormError`.
* **Molecules:** `StatCard`, `SelectInput`, `SearchBar`, `Pagination`, `PageHeader`.
* **Organisms:** `Drawer` (Form Interaktif Tambah/Edit), `Modal` (Dialog Konfirmasi), `Table` (Tabel Data dengan Sorting & Filter), `CommandPalette` (Shortcut Ctrl+K).
* **Features:** `AttendanceCamera` (WebRTC Selfie Capture + Geofence GPS SMA UII Sorowajan).

---

## 📱 3. Strategi Porting ke Next.js, React Native & Flutter

```
                     [ TOKENS & DESIGN SYSTEM SMA UII ]
                                     │
       ┌─────────────────────────────┼─────────────────────────────┐
       ▼                             ▼                             ▼
 [ Web (Next.js / Inertia) ]  [ React Native App ]         [ Flutter App ]
  • Tailwind CSS 4             • NativeWind / StyleSheet     • Flutter ThemeData
  • React UI Components        • Shared Zod Schemas          • Dart Model Generators
  • Storybook Testing          • Playwright/Appium E2E       • Flutter Integration Test
```

1. **Shared Schemas & DTO:** Skema validasi Zod (`resources/js/schemas/`) di `core` dapat langsung digunakan di aplikasi Next.js maupun React Native.
2. **Resilient Selectors:** Atribut `dusk="..."` dan `data-testid="..."` dipasang pada semua elemen interaktif untuk memastikan skrip pengujian E2E (Playwright / Dusk / Appium) tidak rapuh terhadap perubahan visual.
