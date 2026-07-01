# Decision Log — Architecture Decision Record

Format for each entry:
```
## YYYY-MM-DD — [Decision Title]

**Status**: [Proposed / Accepted / Deprecated]
**Decision Maker**: [Name]
**Context**: [Background / problem]
**Decision**: [What was decided]
**Consequences**: [Positive & negative impacts]
```

---

## 2026-06-22 — Monorepo Architecture with Git Submodules

**Status**: Accepted
**Decision Maker**: sandikodev
**Context**: Need to separate backend, frontend, and mobile within one GitHub organization without binding them in a single monolithic repository.
**Decision**: `aksesekolah.git` as the monorepo entrypoint, `core.git` as a submodule in `apps/backend`. Frontend (`webapp.git`) and mobile (`flutter.git`) will follow.
**Consequences**:
- (+) Clear separation of concerns
- (+) Each team can work in their own repository
- (-) Submodules require manual synchronization (git submodule update)
- (-) New developers need to understand the submodule concept

---
