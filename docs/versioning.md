# Versioning Policy

Aethernet follows semantic versioning with a strict patch-liberal cadence and a clear phase-to-bump mapping. This document is the source of truth for both policy and changelog.

## Policy

### Bump criteria

| Bump | When |
|---|---|
| **patch** (x.y.**Z**) | A discrete unit of work is complete and compilation-safe — low-level bug fix, completed polish task, documentation update, or refactor with no behavioral changes. |
| **minor** (x.**Y**.0) | New user-facing feature, new data parameter exposed in Expert mode, or backward-compatible API adjustment. |
| **major** (**X**.0.0) | Breaking architectural overhaul, targeting a newer macOS SDK breaking older support, or fundamental changes to how the app interacts with system background agents. |

### Patch cadence

A patch marks something **done**. Bump **one per completed task** — not per subtask, not only per phase. Subtasks earn commits; tasks earn patches.

A multi-phase user-facing feature batch lands as **one minor bump** at the end of the batch — the task patches ladder up to it. Never collapse a batch into a single combined bump.

### Procedure

1. Update the canonical version source — `VERSION` file at repo root (and, once the Xcode project exists, the `MARKETING_VERSION` build setting must mirror it).
2. `chore: bump version to X.Y.Z` — standalone commit.
3. Add a row to the changelog table below.
4. `docs(versioning): add X.Y.Z entry` — standalone commit.

### Stable checkpoints

A **stable checkpoint** is an empty commit marking a verified-safe compilation state:

```bash
git commit --allow-empty -m "STABLE: vX.Y.Z — Local compilation verified, App Sandbox validated, memory consumption stable at 0% idle"
```

Stable checkpoints are created **only after local physical-machine verification and explicit user confirmation**. Greppable via `git log --grep STABLE`.

## Phase-to-version ladder

| Phase | Final bump | Significance |
|---|---|---|
| Phase 1 — Environment Baseline & Repo Synced Checkpoint | **v0.1.0** | First installable agent skeleton; menu bar entry visible. |
| Phase 2 — Core Event Engine Implementation | (patches only) | Headless detection layer; no user-facing change yet. |
| Phase 3 — Status Bar and Window Integration | **v0.2.0** | First useful user-facing state surface. Closes the Phase 2+3 feature batch. |
| Phase 4 — Dynamic SwiftUI Interface Tiers | **v0.3.0** | Expert tier, geocoding, full menu view. |
| Phase 5 — Native Polish & Production Release Pipeline | **v1.0.0** | Signed, notarized, login-item-capable production release. Major bump for the first stable public release. |

## Changelog

| Version | Date | Type | Summary |
|---|---|---|---|
| 0.0.1 | 2026-05-21 | patch | Repository baseline: `.gitignore`, MIT license, `Claude.md`, `README.md`, `docs/versioning.md`, `docs/current_task.md`, `VERSION` file. Phase 1 / Task 1.1 complete. |
| 0.0.2 | 2026-05-21 | patch | Dual-remote `origin` configured (GitHub + Forgejo, HTTPS, one fetch URL / two push URLs). Both forges synced at identical commit SHA. Phase 1 / Task 1.2 complete. |
