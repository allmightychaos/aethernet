# Aethernet — Claude Instructions (macOS & Swift Edition)

## Clarification

If a request is unclear or has ambiguous scope, ask questions before starting.

## Swift Package Manager & Dependency Operations

Any operation that modifies project dependencies — adding Swift Packages (SPM), changing Build Settings, altering Linker Flags, updating build dependencies, or fetching external third-party binaries — requires explicit user approval **before** execution. No exceptions.

### Protocol

1. **Ask first.** State exactly what framework or package will be integrated and why. Wait for explicit approval.
2. **Research before integrating.** After approval, verify that the dependency supports sandboxing, hardened runtime constraints, and matches the target macOS deployment version (macOS 13.0+). Check for open issues regarding memory leaks in background loops.
3. **Never bypass the App Sandbox.** If an implementation pattern triggers a macOS Sandbox violation or entitlement block, do **not** attempt to bypass or request unneeded broad entitlements (e.g., full disk access, network server entitlements). Stop, report the exact security restriction violation to the user, and present native, sandboxed alternatives.

## Planning

For any task larger than a trivial style/layout fix, maintain a step-by-step plan in `docs/current_task.md`:

- Group steps into **phases** ordered by implementation efficiency.
- Break phases into **tasks** and **subtasks**.
- Check off each step as it completes (`- [x]`).
- **After a phase is fully done**, collapse its detail block to a one-sentence tl;dr (keep the heading + version, mark it ✅). Goals, architecture notes, and subtask checklists have served their purpose — remove them. Version ladder and cross-cutting notes stay.
- Every phase checklist must include `- [ ] Collapse this phase block to tl;dr and commit` as its final item so the collapse is never skipped.

## Commits

Commit exactly one logical step / change per commit. A long, granular history is the goal — never batch files or bundle engine logic with UI adjustments.

**Never stage two architectural categories in one commit. No exceptions. No rationalizing.**

### Cadence — when to commit

- **After every subtask** — the moment a subtask is checked off in `docs/current_task.md`, commit everything it produced (split by structural category) before starting the next.
- **After every task** — even when its subtasks were already committed individually.
- **The plan file commits alone** (e.g., `docs: create current_task.md for Ethernet Monitor implementation`).
- **UI/Layout changes (SwiftUI Views, Assets) commit separately** from Core Logic and C system bridges.
- **State management & AppKit Lifecycles commit separately** from View hierarchies.
- **The version bump commits alone** — never bundled with feature code, entitlements, or docs.

Before moving to the next subtask, ask: "Did I commit everything I just changed, split by category?" If no → commit first.

### Worked example — splitting a mixed change

```
❌ One commit: "feat(MenuBar): add ethernet tracking, AppKit menu items, and settings panel SwiftUI view"
   — bundles low-level system events + AppKit menu integration + SwiftUI layout into one massive change.

✅ Four commits:
   1. feat(EthernetMonitor): implement event-driven NWPathMonitor tracking engine
   2. feat(StatusItemManager): configure AppKit NSStatusItem setup and conditional visibility toggles
   3. feat(MenuView): build native NSMenu custom view rows for basic and expert modes
   4. feat(SettingsWindow): implement standalone SwiftUI settings view and launch-at-login integration
```

### Message format

Short, conventional, present-tense imperative (e.g., `feat: implement getifaddrs ipv4 socket parsing`). No AI attribution lines.

## Versioning

Full rules, changelog, and stable-checkpoint policy: `docs/versioning.md`.

### Criteria (quick reference)

| Bump | When |
|---|---|
| **patch** (x.y.**Z**) | A discrete unit of work is complete and compilation-safe — low-level bug fix, completed layout Polish task, documentation update, or refactor with no behavioral changes. |
| **minor** (x.**Y**.0) | New user-facing feature, new data parameter exposed in Expert mode, or backward-compatible API adjustment (e.g., shifting from CoreLocation to alternative geolocation wrappers). |
| **major** (**X**.0.0) | Breaking architectural overhaul, targeting a newer macOS SDK version breaking older support, or fundamental changes to how the app interacts with system background agents. |

### Patch cadence — patch liberally

A patch marks something **done**. Bump **one per completed task** — not per subtask, not only per phase. Multiple patches per phase are normal and expected. Subtasks earn commits; tasks earn patches.

A multi-phase feature batch lands as **one minor bump** at its end — the task patches ladder up to it. Never collapse a batch into a single combined bump.

### Procedure

1. Run version bump sequence or update project configuration version files.
2. `chore: bump version to X.Y.Z` — standalone commit.
3. Add row to changelog in `docs/versioning.md`.
4. `docs(versioning): add X.Y.Z entry` — standalone commit.

### Stable checkpoints

A **stable checkpoint** is an empty commit marking a verified-safe compilation state to fall back to if a later native API rewrite hits an unresolvable code sign, sandboxing, or compilation error:

```bash
git commit --allow-empty -m "STABLE: vX.Y.Z — Local compilation verified, App Sandbox validated, memory consumption stable at 0% idle"
```

- Create one **only after local physical machine testing and explicit user confirmation** that everything works perfectly — AppKit layout behavior, memory footprint metrics, and background launch behavior. **Always ask the user first.** Never self-declare a state stable.
- The `STABLE:` keyword keeps the last known-good point easily greppable (`git log --grep STABLE`).