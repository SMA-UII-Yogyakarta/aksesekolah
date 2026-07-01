# Project Context — SMART Absen SMA UII

## Summary
Web-based attendance system for SMA UII Yogyakarta with geolocation, camera selfie, and SSO. Developed by PT Koneksi Jaringan Indonesia.

## Tech Stack
- **Backend**: PHP 8.4 / Laravel 13 — repository: `core.git`
- **Frontend**: InertiaJS 3 + React 19 + TypeScript 5.7 + Tailwind CSS 4 + Vite 8
- **Package Manager**: Bun (frontend), Composer (backend)
- **Database**: PostgreSQL 16 via NeonDB (production), PostgreSQL 16 (local), SQLite (testing)
- **Cache & Queue**: Redis / Database driver
- **Object Storage**: S3-compatible (Wasabi / MinIO)
- **Web Server**: Apache (dev) / Nginx (production)
- **Auth**: Laravel Sanctum + Spatie Laravel Permission (RBAC)

## Architecture
```
aksesekolah.git (monorepo entrypoint)
├── apps/backend/ → submodule → core.git
├── brief/        → planning documents
├── docs/         → technical documentation
├── .opencode/    → opencode configuration
└── .openkb/      → knowledge base
```

## Team
| Person | Role | Repo Focus |
|---|---|---|
| sandikodev | Project Manager | All repos |
| Ahmad Hanif Hasan | Product Analyst | brief/, docs/ |
| Fathan Mubina | Junior Frontend Developer | core.git (Inertia+React) |
| Ihsan | Junior Backend Developer | core.git (Laravel API) |
| Azis | Learning Mentor | core.git, playbook (mentoring) |

## Database (10 tables)
- **Master**: users, students, teachers, guardians, school_classes
- **Transaction**: attendances, leave_requests, duty_schedules
- **Configuration**: attendance_time_settings, academic_calendars

## Important Rules
1. All `main` changes must go through PR with at least 1 review
2. Commit using Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
3. Coding style: PSR-12 (Laravel Pint)
4. Tests must be green before merge
5. Credit belongs to PT Koneksi Jaringan Indonesia — selling or redistributing the source code is prohibited
