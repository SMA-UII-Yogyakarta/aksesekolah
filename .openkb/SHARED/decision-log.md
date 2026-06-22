# Decision Log — Catatan Keputusan Arsitektur

Format setiap entri:
```
## YYYY-MM-DD — [Judul Keputusan]

**Status**: [Proposed / Accepted / Deprecated]
**Pemutus**: [Nama]
**Konteks**: [Latar belakang / masalah]
**Keputusan**: [Apa yang diputuskan]
**Konsekuensi**: [Dampak positif & negatif]
```

---

## 2026-06-22 — Arsitektur Monorepo dengan Git Submodules

**Status**: Accepted
**Pemutus**: sandikodev
**Konteks**: Perlu memisahkan backend, frontend, dan mobile dalam satu organisasi GitHub tanpa mengikat mereka dalam satu repo monolitik.
**Keputusan**: `aksesekolah.git` sebagai entrypoint monorepo, `core.git` sebagai submodule di `apps/backend`. Frontend (`webapp.git`) dan mobile (`flutter.git`) akan menyusul.
**Konsekuensi**:
- (+) Pemisahan concern yang jelas
- (+) Setiap tim bisa kerja di repo masing-masing
- (-) Submodule perlu disinkron manual (git submodule update)
- (-) Developer baru perlu paham konsep submodule

---
