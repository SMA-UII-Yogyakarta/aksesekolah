---
layout: default
title: Home
---

# SMART Absen SMA UII

**Integrated Digital Attendance System** based on web with geolocation, camera selfie, and Single Sign-On.

SMART Absen is a digital attendance application that allows students to check in via selfie and geolocation verification, guardians to submit leave requests digitally, and teachers to monitor attendance in **real-time**.

---

## Technical Documentation

All documentation is available in this repository. Navigate based on your role:

### 📘 For Administrative Team & School Management

| Document | Description |
|---|---|
| [Requirement Analysis](03-requirement-analisis.md) | Complete list of functional (14 features) and non-functional (12 items) requirements |
| [ERD & Database Structure](04-erd-database.md) | Entity Relationship Diagram and explanation of 10 database tables |
| [Module & Flow](05-modul-alur-flow.md) | Complete usage scenarios for each user role |
| [Budget & Timeline](08-budget-timeline-roadmap.md) | Budget plan of Rp 8,500,000 and 8-week timeline |

### ⚙️ For Technical Team & Developers

| Document | Description |
|---|---|
| [Monorepo Architecture](01-arsitektur-monorepo.md) | Monorepo structure, git submodules, directory layout |
| [Development Environment](02-lingkungan-development.md) | Laragon setup, PHP 8.4, PostgreSQL 16, troubleshooting |
| [Security & SSO](06-keamanan-sso.md) | Laravel Sanctum, RBAC, Triple-Layer Validation |
| [Git Workflow](07-git-workflow-submodule.md) | Branching strategy, submodule management, CI/CD |
| [Deployment & Infrastructure](09-deployment-infrastruktur.md) | VPS, Nginx, S3 object storage, performance tuning |

---

## Quick Start

### Prerequisites

| Tool | Description |
|---|---|
| [Laragon](https://laragon.org) 6.0+ | Full-stack web environment (PHP, Apache) |
| PHP 8.3+ | Included with Laragon |
| PostgreSQL 16 | Database (via NeonDB or Laragon add-on) |
| Composer | PHP dependency manager |
| Bun 1.2+ | Frontend package manager & build tooling |

### Clone & Setup

```bash
# Clone backend to Laragon
cd C:\laragon\www
git clone git@github.com:SMA-UII-Yogyakarta/core.git smauii-core

# Install dependencies
composer install

# Environment & key
cp .env.example .env
php artisan key:generate

# Database
php artisan migrate --seed
```

Open **http://smauii-core.test** — application ready to use.

---

## Repository Ecosystem

```
SMA-UII-Yogyakarta/
├── aksesekolah/   → Monorepo entrypoint (this repository)
│   ├── apps/backend/ → core.git (Laravel 13)
│   ├── apps/frontend/ → webapp.git [planned]
│   ├── apps/mobile/ → flutter.git [planned]
│   ├── brief/     → Initial planning documents
│   └── docs/      → Technical documentation ← You are here
├── core/          → Backend API Laravel 13
├── webapp/        → Web frontend [planned]
├── flutter/       → Mobile app [planned]
└── .github/       → GitHub organization profile
```

---

## Developed By

**PT Koneksi Jaringan Indonesia** — *Software House Agency Koneksi Digital*

Official information technology development partner for SMA UII Yogyakarta.

> **Copyright & License** — All source code in this project is developed by PT Koneksi Jaringan Indonesia for SMA UII Yogyakarta. All rights reserved. It is prohibited to sell or redistribute without written permission from both parties.

---

<p align="center">
  <strong>SMA UII Yogyakarta</strong><br />
  Jl. Taman Siswa No.158, Wirogunan, Mergangsan, Kota Yogyakarta, DIY 55151<br />
  Telp: (0274) 489693<br />
  <a href="https://www.instagram.com/smauiiofficial/">📸 Instagram</a> ·
  <a href="https://www.youtube.com/channel/UCaLhqaoGXpLHK-KlTwiS8aw">▶️ YouTube</a> ·
  <a href="https://www.tiktok.com/@smauiiofficial">🎵 TikTok</a><br />
  <em>Smart School · Bright Future</em>
</p>
