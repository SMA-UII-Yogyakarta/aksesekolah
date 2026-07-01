# Git Workflow & Submodule Management

## Branching Philosophy

This project uses a simplified **Git Flow** with three main branches:

```
main           ───●────────────────────●───────────►
                  │                    │
develop          ──●──●──●──●──●──●──●──●──●──●──►
                  │  │  │  │  │  │  │  │  │  │
feature/*        ──┘  └──┘  └──┘  └──┘  └──┘
```

| Branch | Function | Deploy |
|---|---|---|
| `main` | Stable code, production-ready | ✅ Yes |
| `develop` | Feature integration, daily development | ❌ No |
| `feature/*` | Specific feature, branch from `develop` | ❌ No |
| `hotfix/*` | Emergency fix, branch from `main` | ✅ Yes (after merge) |

---

## Daily Developer Workflow

### 1. Starting a New Feature

```bash
# Make sure develop is up to date
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/feature-name

# Work as usual...
git add .
git commit -m "feat: adding feature X"
```

### 2. Completing a Feature

```bash
# Update feature branch with latest develop
git checkout develop
git pull origin develop
git checkout feature/feature-name
git rebase develop

# Resolve conflicts if any
# Force push (careful, only for your own branch!)
git push origin feature/feature-name --force-with-lease

# Create Pull Request to develop
gh pr create --base develop --head feature/feature-name
```

### 3. Code Review & Merge

```bash
# After PR is reviewed and approved:
gh pr merge feature/feature-name --squash

# Delete feature branch
git branch -d feature/feature-name
git push origin --delete feature/feature-name
```

### 4. Release to Production

```bash
# Merge develop to main
git checkout main
git pull origin main
git merge develop
git push origin main
```

---

## Submodule Management

### Concept

A submodule is a git reference to another repository within the parent repository. The `aksesekolah.git` monorepo uses submodules to reference separate application repositories.

```
smauii-aksesekolah/           ← parent repository
├── apps/
│   ├── backend/              ← submodule → core.git (SHA pointer)
│   ├── frontend/             ← submodule → webapp.git (SHA pointer) [future]
│   └── mobile/               ← submodule → flutter.git (SHA pointer) [future]
├── .gitmodules               ← submodule list
└── .git/config               ← submodule remote URLs
```

### Adding a New Submodule

```bash
# From the parent repository root
cd C:\laragon\www\smauii-aksesekolah

# Add backend submodule
git submodule add git@github.com:SMA-UII-Yogyakarta/core.git apps/backend

# Commit .gitmodules changes
git add .gitmodules apps/backend
git commit -m "chore: add backend submodule"
git push
```

### Cloning a Repository with Submodules

```bash
# Method 1: Clone directly with submodules
git clone --recurse-submodules git@github.com:SMA-UII-Yogyakarta/aksesekolah.git

# Method 2: Clone first, then init submodules
git clone git@github.com:SMA-UII-Yogyakarta/aksesekolah.git
cd aksesekolah
git submodule update --init --recursive
```

### Updating Submodule to Latest Version

```bash
# Method 1: Update all submodules
git submodule update --remote --recursive

# Method 2: Update specific submodule
git submodule update --remote apps/backend

# After update, commit pointer changes
git add apps/backend
git commit -m "chore: update backend submodule"
git push
```

### Working Inside a Submodule

A submodule is essentially a regular git repository. Developers can enter and work inside it:

```bash
# Enter submodule
cd apps/backend

# Create your own branch
git checkout -b feature/attendance

# Work as usual
git add .
git commit -m "feat: attendance implementation"
git push origin feature/attendance

# Create PR to core.git (not to aksesekolah.git!)
```

> **Important Note:** Submodules are in *detached HEAD* state by default.
> Always create a branch before working inside a submodule!

### Submodule Tips

| Situation | Command |
|---|---|
| Clone new project | `git clone --recurse-submodules <url>` |
| Pull + update submodule | `git pull --recurse-submodules` |
| Update all submodules | `git submodule update --remote` |
| Submodule on specific branch | `git submodule foreach 'git checkout main'` |
| View submodule status | `git submodule status` |

---

## Commit Message Convention

Use [Conventional Commits](https://www.conventionalcommits.org/) convention:

```
<type>: <short description>

Examples:
feat: add attendance feature with geolocation
fix: fix holiday date validation
chore: update laravel/framework dependency
docs: update monorepo architecture documentation
style: fix blade template indentation
refactor: extract validation to service class
test: add unit test for triple-layer validation
```

| Type | Description |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `chore` | Routine tasks (dependency update, config) |
| `docs` | Documentation |
| `style` | Format changes (no logic changes) |
| `refactor` | Code refactoring (no functionality changes) |
| `test` | Adding tests |

---

## GitHub Actions (Continuous Integration)

Recommended CI pipeline for each repository:

### core.git (Backend)

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  laravel:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction
      - run: php artisan key:generate
      - run: php artisan test
      - run: ./vendor/bin/phpstan analyse
```

---

## Monorepo to Submodule Update Flow

```
[Backend Developer completes feature in core.git]
        │
        ▼
[Push to core.git → main branch]
        │
        ▼
[Monorepo maintainer updates submodule]
  cd smauii-aksesekolah
  git submodule update --remote apps/backend
  git add apps/backend
  git commit -m "chore: update backend submodule to latest"
  git push
        │
        ▼
[Other team members do git pull in aksesekolah.git]
  git pull --recurse-submodules
  git submodule update --init --recursive
```

---

## Monorepo .gitignore

```gitignore
# Node
node_modules/
npm-debug.log
yarn-error.log

# OS
.DS_Store
Thumbs.db
*.swp
*.swo

# IDE
.idea/
.vscode/
*.sublime-project
*.sublime-workspace
.zed/

# Env
.env
.env.local

# Log
*.log

# Temporary
tmp/
temp/
```
