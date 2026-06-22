# 📚 OpenKB — Open Knowledge Base

**OpenKB** adalah direktori knowledge base personal untuk komunikasi dengan AI Agent (opencode/Claude Code). Setiap anggota tim memiliki folder personal sendiri yang **tidak di-track git** agar bebas menulis catatan, instruksi personal, dan log sesi AI.

---

## Struktur Folder

```
.openkb/
├── TEMPLATE.md               ← Pedoman ini (di-track git)
├── SHARED/                    ← Referensi bersama (di-track git)
│   ├── glossary.md            ← Istilah-istilah teknis project
│   ├── references.md          ← Link & referensi cepat
│   └── decision-log.md        ← Catatan keputusan arsitektur
└── PERSONAL/                  ← ❌ GITIGNORE — isi masing-masing
    └── <github-username>/
        ├── profile.md         ← "Siapa saya" — role, skill, preferensi
        ├── instructions.md    ← Instruksi personal untuk AI Agent
        ├── sessions/          ← Log percakapan dengan AI
        │   └── YYYY-MM-DD.md
        └── quick-notes.md     ← Catatan cepat / coding snippets
```

## Cara Penggunaan

### 1. Setup Awal
```bash
# Clone repo
cd C:\laragon\www\smauii-aksesekolah

# Buat folder personal kamu sendiri
mkdir .openkb\PERSONAL\<github-username>
mkdir .openkb\PERSONAL\<github-username>\sessions

# Copy template profile & instructions
# (isi sesuai dengan peranmu di project)
```

### 2. Profile (`profile.md`)
Tulis tentang dirimu:
- Nama & Role
- Tech stack yang kamu kuasai
- Area fokus di project ini
- Preferensi coding style
- Hal yang perlu dihindari AI saat membantu kamu

### 3. Instructions (`instructions.md`)
Instruksi khusus untuk AI Agent:
- Bagaimana kamu ingin AI menjelaskan sesuatu
- Format output yang kamu suka
- Hal-hal yang AI harus ingat tentang caramu bekerja

### 4. Sessions (`sessions/YYYY-MM-DD.md`)
Setelah ngobrol dengan AI, catat:
- Apa yang dikerjakan
- Keputusan penting
- Hal yang perlu diingat untuk sesi berikutnya

### 5. Quick Notes (`quick-notes.md`)
Tempat menyimpan:
- Snippet kode yang sering dipakai
- Command terminal yang sering lupa
- Checklist task harian

---

## Integrasi dengan opencode

File di `.openkb/SHARED/` bisa dijadikan referensi di `opencode.json`:

```json
{
  "instructions": [
    ".opencode/SHARED/project-context.md",
    ".opencode/SHARED/agent-rules.md"
  ],
  "references": {
    "openkb": {
      "path": ".openkb",
      "description": "Knowledge base — glossary, referensi, keputusan"
    }
  }
}
```

Dengan begitu, AI Agent akan otomatis paham konteks project dan aturan main tim setiap kali dipanggil.

---

## Aturan

1. **Folder `PERSONAL/` tidak boleh di-commit** — sudah ada di `.gitignore`
2. File di `SHARED/` boleh diedit bersama via Pull Request
3. Gunakan markdown untuk semua file
4. Bahasa Indonesia untuk catatan, Inggris untuk kode
