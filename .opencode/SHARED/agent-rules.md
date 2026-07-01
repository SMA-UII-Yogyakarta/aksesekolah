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
main → develop → feature/feature-name
```
- `main`: production — protected, via PR only
- `develop`: integration branch
- `feature/*`: for daily development

## Laravel Conventions
- Route: web.php for Inertia pages, api.php for API Sanctum
- Controller: single action prefer method `__invoke` for simple case
- Validation: Form Request class
- Database: migration + seeder, do not edit SQL directly
- Eloquent: eager loading to prevent N+1
- Config: all environment variables via `.env`, not hardcoded

## Security
- Do not commit `.env`, key, token, password
- Do not expose debug info in production
- Use prepared statements (via Eloquent)
- User input must be validated & sanitized
