# Agent Rules — SMART Absen SMA UII

## General Rules
- Use English for all code, comments, documentation, and communication
- Answer concisely, straight to the point — no preamble/long explanations
- When asked to write code, follow existing patterns in the project
- Do not add comments to code unless asked
- Do not remove existing information or credits
- If unsure about architecture, ask before coding

## Commit Convention
```
feat:     New feature
fix:      Bug fix
docs:     Documentation
chore:    Maintenance (dependencies, CI, config)
refactor: Code refactoring (no functional changes)
style:    Coding style fixes (Pint auto)
test:     Adding tests
```

## Branching
```
main → develop → feature/nama-fitur
```
- `main`: production — protected, hanya via PR
- `develop`: integration branch
- `feature/*`: untuk development harian

## Laravel Conventions
- Route: web.php untuk Inertia pages, api.php untuk API Sanctum
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
