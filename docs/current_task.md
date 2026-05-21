# Aethernet — Current Task

Master implementation plan from cold workspace to v1.0.0 production release. Each phase is a nested checklist; check items as they complete (`- [x]`). When a phase finishes, collapse its detail block to a single-sentence tl;dr with its closing version and a ✅ mark, then commit the collapse.

---

## Cross-cutting reference

- **Bundle identifier:** `net.skritt.aethernet`
- **Display name:** Aethernet (binary / repo / target name stays lowercase `aethernet`)
- **Deployment target:** macOS 13.0 (Ventura)
- **Architecture:** App-sandboxed `LSUIElement = true` menu bar agent
- **Entitlements (planned):**
  - `com.apple.security.app-sandbox` — mandatory
  - `com.apple.security.network.client` — for `NWPathMonitor` and `CLGeocoder`
  - `com.apple.security.personal-information.location` — Expert tier only
- **Primary forge:** `github.com/allmightychaos/aethernet`
- **Mirror forge:** `code.skritt.net/allmightychaos/aethernet`
- **Sync verification:** `git remote -v` must show one fetch URL and two distinct push URLs under `origin`.

## Version ladder (target)

- Phase 1 close → **v0.1.0**
- Phase 2 internal → patch series, no minor
- Phase 3 close (Phase 2+3 feature batch) → **v0.2.0**
- Phase 4 close → **v0.3.0**
- Phase 5 close → **v1.0.0**

Task-level patches between minors are expected and encouraged.

---

## Phase 1 — Environment Baseline & Repo Synced Checkpoint

**Goal:** Take the workspace from a single `Claude.md` to a fully synchronized public dual-remote repository hosting a compilable, sandboxed, login-item-capable `LSUIElement` agent stub with a placeholder menu bar item. End state: anyone can `git clone` from either forge and build a working (empty-functionality) `.app`.

**Architecture notes:**
- Repository tracks `Claude.md` and the entire `docs/` tree publicly — deliberate transparency choice.
- Dual-push under a single `origin` remote: one fetch URL (GitHub), two push URLs (GitHub + Forgejo), HTTPS transport using `gh`'s credential helper and `tea`'s token respectively.
- Xcode project is checked in; `xcuserdata/` and `DerivedData/` gitignored.
- `Info.plist` carries `LSUIElement = YES` so no Dock icon ever appears.

### Task 1.1 — Local workspace & baseline files

- [x] `git init -b main`
- [x] `.gitignore` (macOS, Xcode, SwiftPM, fastlane, signing artifacts, editors)
- [x] `LICENSE` (MIT, attributed to allmightychaos, 2026)
- [x] `Claude.md` (operational manual; preserved verbatim from user)
- [x] `docs/versioning.md` (policy + phase-to-version ladder + empty changelog seed)
- [x] `README.md` (teaser, features, architecture diagram, footprint targets)
- [x] `docs/current_task.md` (this file)
- [x] `VERSION` file at repo root (`0.0.1`)
- [x] Initial micro-commit sequence — one architecture category per commit
- [x] Bump to **v0.0.1** (patch — baseline docs + infra complete)

### Task 1.2 — Remote forges & dual-push origin

- [ ] `gh repo create allmightychaos/aethernet --public --description "..."`
- [ ] `tea repo create --name aethernet --description "..."`
- [ ] `git remote add origin https://github.com/allmightychaos/aethernet.git`
- [ ] `git remote set-url --add --push origin https://github.com/allmightychaos/aethernet.git`
- [ ] `git remote set-url --add --push origin https://code.skritt.net/allmightychaos/aethernet.git`
- [ ] Verify `git remote -v` shows the expected fetch + dual push topology
- [ ] `git push -u origin main`; verify both forges received the same commit SHA
- [ ] Bump to **v0.0.2** (patch — dual-remote sync verified)

### Task 1.3 — Xcode project skeleton

- [ ] Create `aethernet.xcodeproj` with a single macOS App target named `aethernet`
- [ ] Bundle ID `net.skritt.aethernet`; display name `Aethernet`; deployment target macOS 13.0
- [ ] Info.plist: `LSUIElement = YES`, `LSApplicationCategoryType = public.app-category.utilities`
- [ ] Entitlements file: `com.apple.security.app-sandbox`, `com.apple.security.network.client`
- [ ] `aethernet/Engine/` and `aethernet/UI/` source folders staged
- [ ] Asset catalog with placeholder template symbol
- [ ] Confirm cold debug build succeeds with no warnings; launches with no UI
- [ ] Bump to **v0.0.3** (patch — Xcode skeleton compilable)

### Task 1.4 — Placeholder `NSStatusItem`

- [ ] `AppDelegate.swift` claiming `NSApplicationDelegate`
- [ ] `StatusItemManager` stub class: instantiates an `NSStatusItem` with a system symbol icon
- [ ] No menu yet; click does nothing
- [ ] Confirm the item appears in the menu bar on launch, disappears on quit
- [ ] Bump to **v0.0.4** (patch — placeholder UI present)

### Task 1.5 — Phase 1 closure

- [ ] Manual verification on physical machine: build runs, agent appears, no Dock icon, no crash logs
- [ ] Bump to **v0.1.0** (minor — first installable user-facing milestone)
- [ ] Push and confirm both remotes synced
- [ ] *Request user confirmation for a STABLE checkpoint commit*
- [ ] Collapse this phase block to tl;dr and commit

---

## Phase 2 — Core Event Engine Implementation

**Goal:** Replace the placeholder with a fully event-driven detection engine that knows exactly when a wired Ethernet adapter transitions between attached/detached/up/down — with zero polling, zero idle work, and BSD-accurate inspection of the active interface.

**Architecture notes:**
- `EthernetEngine` is an `actor` for thread-safety; `NWPathMonitor.pathUpdateHandler` is its sole input edge.
- Dedicated `DispatchQueue(label: "net.skritt.aethernet.engine", qos: .utility)` services the monitor.
- BSD inspection sits behind an `InterfaceInspector` value type wrapping a pure C bridge over `getifaddrs(3)`, `freeifaddrs(3)`, `inet_ntop(3)`, and the `sa_family_t` casting dance. Memory is freed in a `defer` block — no leaks across the linked list.
- Interface-type discrimination uses `SCNetworkInterface` (from `SystemConfiguration`) cross-checked against `ifa_flags` to ensure we don't confuse Wi-Fi `en0` for wired Ethernet on laptops.
- Engine emits a single immutable `EthernetSnapshot` value type on each transition. Downstream consumers subscribe via `AsyncStream`.

### Task 2.1 — `NWPathMonitor` wrapper on dedicated queue

- [ ] `PathMonitorService` class on a `.utility` QoS queue
- [ ] `pathUpdateHandler` forwards into the engine actor
- [ ] Verified zero idle work between path updates via Instruments

### Task 2.2 — `getifaddrs` C-bridge & `InterfaceInspector`

- [ ] `InterfaceInspector.snapshot()` walks the `ifaddrs` linked list
- [ ] `sockaddr_in` and `sockaddr_in6` parsing via `inet_ntop`
- [ ] `AF_LINK` parsing for MAC, MTU
- [ ] Virtual / loopback / tunnel exclusion (`utun*`, `lo0`, `awdl*`, `llw*`, `bridge*`, `gif*`, `stf*`, `anpi*`, `ap*`, `pktap*`)
- [ ] `defer { freeifaddrs(ptr) }` cleanup verified leak-free under repeated invocation

### Task 2.3 — `SCNetworkInterface` cross-check

- [ ] Query `SCNetworkInterfaceCopyAll()` and match by BSD name
- [ ] Filter to `kSCNetworkInterfaceTypeEthernet`
- [ ] Combined with flag check (`IFF_UP | IFF_RUNNING`) and absence of `IFF_LOOPBACK`, `IFF_POINTOPOINT`

### Task 2.4 — Engine state machine & `EthernetSnapshot` publication

- [ ] `EthernetSnapshot` value type (link state, IPv4, IPv6, MAC, MTU, interface name, link speed if exposed)
- [ ] State enum: `.noAdapter`, `.adapterPresentNoLink`, `.adapterPresentLinkUp(snapshot:)`
- [ ] `AsyncStream<EthernetSnapshot>` exposed via `EthernetEngine.events`
- [ ] Snapshot deduplication so identical consecutive paths don't re-emit

### Task 2.5 — Phase 2 closure

- [ ] Unit tests for snapshot deltas across simulated path updates
- [ ] Memory profile confirms 0% idle CPU between events
- [ ] Bump to **v0.1.x final patch** (engine complete; minor deferred to Phase 3 close)
- [ ] Collapse this phase block to tl;dr and commit

---

## Phase 3 — Status Bar and Window Integration

**Goal:** Wire the engine's `EthernetSnapshot` stream to a fully reactive `NSStatusItem`. State drives icon, color, and visibility. When no wired interface exists, the menu bar entry hides itself completely. When a cable arrives, the icon appears with the correct color for the link state.

**Architecture notes:**
- `StatusItemManager` is `@MainActor`; subscribes to the engine via `for await snapshot in stream`.
- Visibility toggle uses `NSStatusItem.isVisible` — set only from the main actor; AppKit handles bar layout with no glitches.
- Icon: SF Symbol `cable.connector` (up) / `cable.connector.slash` (down). Tint follows `NSColor.systemGreen` / `.systemRed` / `.tertiaryLabelColor`.
- Reopen interception: `applicationShouldHandleReopen(_:hasVisibleWindows:)` opens the menu instead of presenting a window.

### Task 3.1 — Reactive icon & color binding

- [ ] Subscribe `StatusItemManager` to `EthernetEngine.events`
- [ ] Symbol swap on link state transition, animated via `NSImage` template tint
- [ ] Color states: connected (green), no link (red), no adapter (hidden — see 3.2)

### Task 3.2 — Visibility auto-hide policy

- [ ] `isVisible = false` when state is `.noAdapter`
- [ ] `isVisible = true` on first transition into `.adapterPresent*`
- [ ] Verify no layout jitter under repeated plug/unplug at speed

### Task 3.3 — Reopen interception

- [ ] `NSApplicationDelegate.applicationShouldHandleReopen` opens the status menu
- [ ] No alternate window is ever presented

### Task 3.4 — Phase 3 closure

- [ ] Manual verification: cable in → icon appears within 200 ms; cable out → icon hides within 200 ms
- [ ] Bump to **v0.2.0** (minor — closes the Phase 2+3 user-facing feature batch)
- [ ] Collapse this phase block to tl;dr and commit

---

## Phase 4 — Dynamic SwiftUI Interface Tiers

**Goal:** Build the click-down menu UI in two tiers — basic and expert — with the expert tier surfacing physical-layer detail, MAC, MTU, and a CoreLocation-driven geocoded ISP / region descriptor. Settings window allows tier selection and login-item registration toggle.

**Architecture notes:**
- `NSMenu` hosts a single custom `NSView` row that wraps a SwiftUI `MenuView` inside `NSHostingView`.
- Typography: SF Pro Text 12pt for primary labels, 11pt monospaced for IP / MAC. Color tokens derive from `NSColor.labelColor` family for full Light / Dark / Increased-Contrast support.
- `LocationCoordinator` is a `@MainActor`-isolated `CLLocationManager` host. First Expert-tier reveal triggers `requestWhenInUseAuthorization()`; subsequent reveals reuse cached coordinate + reverse-geocoded `CLPlacemark` for up to one link-transition cycle.
- Denied or disabled location: Expert tier shows "Location disabled" copy with a one-click "Open System Settings" deep link to `x-apple.systempreferences:com.apple.preference.security?Privacy_LocationServices`.

### Task 4.1 — `MenuView` skeleton & basic tier

- [ ] `NSHostingView` wrapper inside a custom `NSMenuItem` view
- [ ] Basic tier: link state row, primary IPv4 row, "Show details" disclosure
- [ ] Typography & spacing tokens defined in a single `MenuStyle` value type

### Task 4.2 — Expert tier reveal & data binding

- [ ] Disclosure expands to show MAC, MTU, IPv6, link speed, interface name
- [ ] Monospaced font for address-shaped values
- [ ] Smooth height-animation on disclosure toggle

### Task 4.3 — `LocationCoordinator` & permission flow

- [ ] `CLLocationManager` instantiated on main actor
- [ ] `requestWhenInUseAuthorization()` on first Expert-tier reveal
- [ ] Three handled states: `.authorizedWhenInUse`, `.denied`, `.notDetermined`
- [ ] Deep link to System Settings for the denied path

### Task 4.4 — `CLGeocoder` reverse-geocoding integration

- [ ] `reverseGeocodeLocation(_:)` async/throws path
- [ ] `CLPlacemark` → human-readable region descriptor
- [ ] Single in-memory cache invalidated on link-transition

### Task 4.5 — Settings window (SwiftUI)

- [ ] Standalone window — opened from menu, not from Dock
- [ ] Toggle: tier selection (basic / expert default)
- [ ] Toggle: launch at login (wires to Phase 5 `SMAppService`)
- [ ] About row (version, license link, GitHub link, Forgejo link)

### Task 4.6 — Phase 4 closure

- [ ] Visual QA on Light / Dark / Increased-Contrast modes
- [ ] Bump to **v0.3.0** (minor — Expert tier shipped)
- [ ] Collapse this phase block to tl;dr and commit

---

## Phase 5 — Native Polish & Production Release Pipeline

**Goal:** Take the v0.3.x build to a signed, notarized, login-item-capable, dual-forge-published v1.0.0 release.

**Architecture notes:**
- `SMAppService.mainApp.register()` for launch-at-login — sandbox-compatible, no helper bundle. App must be located in `/Applications` or `~/Applications` for registration to succeed.
- Build is signed with Developer ID Application; hardened runtime enabled; library validation on.
- Notarization via `notarytool submit ... --wait`.
- DMG packaged via `create-dmg` (host tooling; not a project dependency).
- Release artifacts (`.dmg`, `.app.zip`, dSYM zip) published synchronously to both forges via `gh release create` and `tea releases create`.

### Task 5.1 — `SMAppService` registration UI

- [ ] Settings window toggle wired to `SMAppService.mainApp`
- [ ] Status reflected from `.status` enum (enabled / requires-approval / not-registered)
- [ ] User-friendly copy for the `.requiresApproval` system-settings nudge

### Task 5.2 — Instruments profiling against footprint targets

- [ ] Time Profiler: 0% idle CPU between events confirmed
- [ ] Allocations: RSS within stated bounds in all four states from the README table
- [ ] `powermetrics` wake-ups: 0/s idle confirmed

### Task 5.3 — Developer ID signing & hardened runtime

- [ ] Code-sign with Developer ID Application certificate
- [ ] Hardened runtime enabled; library validation on
- [ ] Entitlements verified via `codesign -d --entitlements -`

### Task 5.4 — Notarization automation

- [ ] `notarytool submit` script using app-specific password
- [ ] Stapling via `xcrun stapler staple`

### Task 5.5 — DMG packaging

- [ ] `create-dmg` invocation produces a signed, stapled DMG
- [ ] Drag-to-Applications layout

### Task 5.6 — Dual-forge release publication

- [ ] `gh release create v1.0.0 --notes-file …` with artifacts attached
- [ ] `tea releases create --tag v1.0.0 …` with the same artifacts
- [ ] Verify both forges show identical artifact set and same release notes

### Task 5.7 — Phase 5 closure

- [ ] User-confirmed STABLE checkpoint at v1.0.0
- [ ] Bump to **v1.0.0** (major — first stable public release)
- [ ] Collapse this phase block to tl;dr and commit
