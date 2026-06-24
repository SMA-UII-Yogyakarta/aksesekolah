# 📚 OpenKB — Open Knowledge Base

**OpenKB** adalah direktori knowledge base personal untuk komunikasi dengan AI Agent (Opencode, Claude Code, Claw Code, Cursor). Setiap anggota tim memiliki folder personal sendiri yang **tidak di-track git** agar bebas menulis catatan, instruksi personal, dan log sesi AI.

Role-specific instructions di-load dari **playbook/ROLES/** — lihat `../smauii-playbook/ROLES/` untuk daftar role yang tersedia.

---

## Struktur Folder

```
~/.openkb/                       ← Di HOME directory, bukan di repo
├── PERSONAL/
│   └── <github-username>/
│       ├── profile.md           ← "Siapa saya" — role, skill, preferensi
│       ├── sessions/            ← Log percakapan dengan AI
│       │   └── YYYY-MM-DD.md
│       └── quick-notes.md       ← Catatan cepat / coding snippets
│
├── ROLES/                       ← Copy dari playbook.git/ROLES/
│   ├── product-analyst.md       ← Role-specific AI instructions
│   ├── junior-backend.md        ← Role-specific AI instructions
│   ├── junior-frontend.md       ← Role-specific AI instructions
│   ├── senior-developer.md      ← Role-specific AI instructions
│   ├── project-manager.md       ← Role-specific AI instructions
│   └── stakeholder.md           ← Role-specific AI instructions
│
└── PROJECTS/                    ← Catatan per project
    └── smart-absen/
        ├── notes.md
        └── decisions.md
```

## Cara Penggunaan

### 1. Setup Awal
```bash
# Buat folder personal
mkdir ~\.openkb\PERSONAL\<github-username>
mkdir ~\.openkb\PERSONAL\<github-username>\sessions
mkdir ~\.openkb\ROLES
mkdir ~\.openkb\PROJECTS\smart-absen

# Copy role instructions dari playbook
# (pilih sesuai role kamu saat ini)
copy ..\smauii-playbook\ROLES\product-analyst.md ~\.openkb\ROLES\
copy ..\smauii-playbook\ROLES\junior-backend.md ~\.openkb\ROLES\
copy ..\smauii-playbook\ROLES\junior-frontend.md ~\.openkb\ROLES\
```

### 2. Profile (`profile.md`)
Tulis tentang dirimu:
- Nama & Role saat ini
- Tech stack yang kamu kuasai
- Area fokus di project ini
- Preferensi coding style
- Hal yang perlu dihindari AI saat membantu kamu

### 3. Role (`ROLES/<role>.md`)
Role-specific instructions dicopy dari `playbook.git/ROLES/`:
- **Isi ROLES** sudah didefinisikan di playbook oleh Sandikodev
- Kamu tinggal copy file role yang sesuai
- Kalau role-mu berubah (misal: dari junior ke mid), tinggal ganti file role
- Contoh: Hanif mulai sebagai Product Analyst, nanti bisa ganti ke UI/UX Designer

### 4. Setup opencode.json (User-Level)
Buat file `~\.config\opencode\opencode.json`:
```json
{
  "username": "github-username-anda",
  "instructions": [
    "~/.openkb/PERSONAL/<user>/profile.md",
    "~/.openkb/ROLES/<role-saat-ini>.md"
  ],
  "references": {
    "personal-notes": {
      "path": "~/.openkb/PROJECTS/smart-absen",
      "description": "Catatan pribadi tentang project SMART Absen"
    }
  }
}
```

### 5. Sessions (`sessions/YYYY-MM-DD.md`)
Setelah ngobrol dengan AI, catat:
- Apa yang dikerjakan
- Keputusan penting
- Hal yang perlu diingat untuk sesi berikutnya

### 6. Quick Notes (`quick-notes.md`)
Tempat menyimpan:
- Snippet kode yang sering dipakai
- Command terminal yang sering lupa
- Checklist task harian

---

## Integrasi dengan AI Agent Tools

### Opencode
File di `~/.openkb/` direferensikan di `~/.config/opencode/opencode.json` (user-level).

### Claude Code
File `.claude/settings.json` di root repo membaca referensi yang sama.

### Cursor IDE
File `.cursorrules` di root repo berisi konteks project.

### Semua tool membaca dari sumber yang sama:
- **Project context**: `.opencode/SHARED/` di masing-masing repo
- **Personal profile**: `~/.openkb/PERSONAL/<user>/profile.md`
- **Role instructions**: `~/.openkb/ROLES/<role>.md`
- **Knowledge base**: `playbook.git/`, `kaede-powerup.git/`

Dengan arsitektur ini, AI Agent akan otomatis paham konteks project, role kamu, dan aturan main tim — apapun tool yang kamu pilih.

---

## Aturan

1. **Folder `PERSONAL/` tidak boleh di-commit** — sudah ada di `.gitignore`
2. File di `SHARED/` boleh diedit bersama via Pull Request
3. **Role instructions** di `~/.openkb/ROLES/` — copy dari playbook.git, update sendiri sesuai role saat ini
4. **Catatan project** di `~/.openkb/PROJECTS/<project>/` — untuk konteks yang tidak perlu dishare
5. Gunakan markdown untuk semua file
6. Bahasa Indonesia untuk catatan, Inggris untuk kode

## Referensi

- **Role definitions**: `../smauii-playbook/ROLES/`
- **AI workflow**: `../smauii-playbook/WORKFLOWS/ai-agent-workflow.md`
- **Access policy**: `../smauii-playbook/ACCESS/outsourced-policy.md`
- **KAEDE Power-Up**: `../kaede-powerup/`
