# 📚 OpenKB — Open Knowledge Base

**OpenKB** is a personal knowledge base directory for communication with AI Agents (Opencode, Claude Code, Claw Code, Cursor). Each team member has their own personal folder that is **not git-tracked** so they are free to write notes, personal instructions, and AI session logs.

Role-specific instructions are loaded from **playbook/ROLES/** — see `../smauii-playbook/ROLES/` for available roles.

---

## Folder Structure

```
~/.openkb/                       ← In HOME directory, not in repo
├── PERSONAL/
│   └── <github-username>/
│       ├── profile.md           ← "Who I am" — role, skills, preferences
│       ├── sessions/            ← Conversation logs with AI
│       │   └── YYYY-MM-DD.md
│       └── quick-notes.md       ← Quick notes / coding snippets
│
├── ROLES/                       ← Copy from playbook.git/ROLES/
│   ├── product-analyst.md       ← Role-specific AI instructions
│   ├── junior-backend.md        ← Role-specific AI instructions
│   ├── junior-frontend.md       ← Role-specific AI instructions
│   ├── senior-developer.md      ← Role-specific AI instructions
│   ├── project-manager.md       ← Role-specific AI instructions
│   └── stakeholder.md           ← Role-specific AI instructions
│
└── PROJECTS/                    ← Per-project notes
    └── smart-absen/
        ├── notes.md
        └── decisions.md
```

## Usage Guide

### 1. Initial Setup
```bash
# Create personal folder
mkdir ~\.openkb\PERSONAL\<github-username>
mkdir ~\.openkb\PERSONAL\<github-username>\sessions
mkdir ~\.openkb\ROLES
mkdir ~\.openkb\PROJECTS\smart-absen

# Copy role instructions from playbook
# (choose according to your current role)
copy ..\smauii-playbook\ROLES\product-analyst.md ~\.openkb\ROLES\
copy ..\smauii-playbook\ROLES\junior-backend.md ~\.openkb\ROLES\
copy ..\smauii-playbook\ROLES\junior-frontend.md ~\.openkb\ROLES\
```

### 2. Profile (`profile.md`)
Write about yourself:
- Name & Current Role
- Tech stack you master
- Focus areas in this project
- Coding style preferences
- Things AI should avoid when helping you

### 3. Role (`ROLES/<role>.md`)
Role-specific instructions are copied from `playbook.git/ROLES/`:
- **ROLES content** is already defined in the playbook by Sandikodev
- Just copy the appropriate role file
- If your role changes (e.g., from junior to mid), just replace the role file
- Example: Hanif starts as Product Analyst, later can switch to UI/UX Designer

### 4. Setup opencode.json (User-Level)
Create file `~\.config\opencode\opencode.json`:
```json
{
  "username": "your-github-username",
  "instructions": [
    "~/.openkb/PERSONAL/<user>/profile.md",
    "~/.openkb/ROLES/<current-role>.md"
  ],
  "references": {
    "personal-notes": {
      "path": "~/.openkb/PROJECTS/smart-absen",
      "description": "Personal notes about the SMART Absen project"
    }
  }
}
```

### 5. Sessions (`sessions/YYYY-MM-DD.md`)
After chatting with AI, note down:
- What was done
- Important decisions
- Things to remember for the next session

### 6. Quick Notes (`quick-notes.md`)
Place to store:
- Frequently used code snippets
- Terminal commands you often forget
- Daily task checklist

---

## Integration with AI Agent Tools

### Opencode
Files in `~/.openkb/` are referenced in `~/.config/opencode/opencode.json` (user-level).

### Claude Code
File `.claude/settings.json` in the repo root reads the same references.

### Cursor IDE
File `.cursorrules` in the repo root contains project context.

### All tools read from the same sources:
- **Project context**: `.opencode/SHARED/` in each repo
- **Personal profile**: `~/.openkb/PERSONAL/<user>/profile.md`
- **Role instructions**: `~/.openkb/ROLES/<role>.md`
- **Knowledge base**: `playbook.git/`, `kaede-powerup.git/`

With this architecture, the AI Agent will automatically understand the project context, your role, and team rules — regardless of which tool you choose.

---

## Rules

1. **`PERSONAL/` folder must not be committed** — already in `.gitignore`
2. Files in `SHARED/` can be edited together via Pull Request
3. **Role instructions** in `~/.openkb/ROLES/` — copy from playbook.git, update yourself according to current role
4. **Project notes** in `~/.openkb/PROJECTS/<project>/` — for context that does not need to be shared
5. Use markdown for all files
6. Indonesian for notes, English for code

## References

- **Role definitions**: `../smauii-playbook/ROLES/`
- **AI workflow**: `../smauii-playbook/WORKFLOWS/ai-agent-workflow.md`
- **Access policy**: `../smauii-playbook/ACCESS/outsourced-policy.md`
- **KAEDE Power-Up**: `../kaede-powerup/`
