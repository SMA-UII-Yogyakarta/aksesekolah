# Glossary — Terms & Abbreviations

| Term | Abbreviation | Description |
|---|---|---|
| **SMART Absen** | — | SMA UII digital attendance system name |
| **SSO** | Single Sign-On | One account login for all SMA UII applications |
| **RBAC** | Role-Based Access Control | Access rights system based on roles (Admin, Student, Teacher, etc.) |
| **IdP** | Identity Provider | Identity provider for SSO (Laravel Sanctum) |
| **Live Attendance** | — | Real-time attendance feature with selfie + GPS |
| **Triple-Layer Validation** | — | 3-layer validation: academic calendar → active day → time range |
| **Object Storage** | — | File (photo) storage in S3-compatible, not in DB |
| **Guardian** | — | Parent/guardian of student who can submit leave requests |
| **Homeroom Teacher** | — | Teacher who verifies student leave in their class |
| **Duty Teacher** | — | Teacher who monitors real-time attendance per class |
| **Monorepo** | — | Single repo with submodules for multiple apps |
| **Laragon** | — | Windows development environment (PHP, MySQL, Apache) |

## Repository Names

| Short Name | GitHub Repo | Content |
|---|---|---|
| aksesekolah | `SMA-UII-Yogyakarta/aksesekolah` | Monorepo entrypoint + docs |
| core | `SMA-UII-Yogyakarta/core` | Laravel Backend |
| webapp | `SMA-UII-Yogyakarta/webapp` | Web frontend *(planned)* |
| flutter | `SMA-UII-Yogyakarta/flutter` | Mobile app *(planned)* |
| .github | `SMA-UII-Yogyakarta/.github` | Organization profile & templates |

## Domains

| Domain | Function |
|---|---|
| `smauii-core.test` | Local dev |
| `absen.smauii.sch.id` | Production *(planned)* |
| `SMA-UII-Yogyakarta.github.io/aksesekolah` | GitHub Pages (documentation) |
