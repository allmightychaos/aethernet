# aethernet

A minimalist macOS menu bar agent that surfaces wired Ethernet status — and nothing else.

Lowercase by design. Invisible when there's no cable to talk about. Zero scheduled work, zero polling loops, zero idle CPU.

---

## design philosophy

A monitoring tool that costs nothing to keep around. Aethernet wakes only when the kernel tells it something has changed — never on a timer. When no wired interface is attached, the menu bar entry hides itself completely. When a cable lands, the icon appears, colored to match the link's actual state.

The name is lowercase. So is the binary. So is everything that doesn't need to shout.

## features

- **Zero-idle overhead engine.** Built on `NWPathMonitor` with a single dedicated utility-QoS dispatch queue. The process performs no work between actual link transitions.
- **Native AppKit transparency.** `NSStatusItem.isVisible` toggles cleanly off when no wired interface is plugged in. The agent leaves no trace in the menu bar when it has nothing to report.
- **BSD-accurate interface inspection.** A low-level C-bridge over `getifaddrs(3)` reads link-layer truth — MAC, MTU, IPv4 / IPv6 — directly from the kernel's interface table, filtered to the active physical Ethernet adapter only.
- **Two information tiers.** A basic view for at-a-glance status (link state, IPv4) and an Expert tier that adds physical layer details, MAC, MTU, full interface metadata, and `CLGeocoder` reverse-geocoded region context.
- **Sandbox-native.** Ships fully App-Sandboxed, signed, and notarized. No helper bundles, no broad entitlements, no privileged operations.
- **Modern boot registration.** Optional launch-at-login through `SMAppService.mainApp` — no `Library/LoginItems` helper, no `SMLoginItemSetEnabled` legacy path.

## architecture

```
                          ┌───────────────────────────┐
   kernel link event   →  │  NWPathMonitor (utility)  │
                          └─────────────┬─────────────┘
                                        │ pathUpdateHandler
                                        ▼
                          ┌───────────────────────────┐
                          │  EthernetEngine           │
                          │   • getifaddrs C-bridge   │
                          │   • SCNetworkInterface    │
                          │   • state machine         │
                          └─────────────┬─────────────┘
                                        │ @MainActor publish
                                        ▼
                          ┌───────────────────────────┐
                          │  StatusItemManager        │
                          │   • isVisible toggle      │
                          │   • icon + label colors   │
                          │   • NSMenu host           │
                          └─────────────┬─────────────┘
                                        │ on click
                                        ▼
                          ┌───────────────────────────┐
                          │  MenuView (SwiftUI)       │
                          │   • basic tier            │
                          │   • expert tier           │
                          │   • CLGeocoder lookup     │
                          └───────────────────────────┘
```

Every layer above is event-driven. No layer holds a timer. No layer schedules a recurring task.

## system footprint expectations

| State | CPU | RSS | Wake-ups |
|---|---|---|---|
| Idle, no cable plugged in, agent hidden | **0%** | < 14 MB | 0 / s |
| Idle, cable connected, basic tier rendered | **0%** | < 18 MB | 0 / s |
| Link transition (plug or unplug event) | brief spike, < 5 ms wall | < 20 MB | 1 per event |
| Expert tier visible, geo lookup running | ≤ 1% for ≤ 2 s, then 0% | < 22 MB | event-bound |

These targets are continuously verified through Instruments time-profiling and `powermetrics` wake-up sampling. Any deviation is treated as a regression.

## install

Aethernet is distributed as a signed, notarized macOS application bundle. The application registers itself as a login item only with explicit user consent through the in-app toggle. Releases are published synchronously to both [GitHub](https://github.com/allmightychaos/aethernet) and [code.skritt.net](https://code.skritt.net/allmightychaos/aethernet).

## requirements

- macOS 13.0 (Ventura) or newer
- Wired Ethernet adapter (USB-C, Thunderbolt, or built-in)
- Optional: Location Services permission (for the Expert tier's reverse-geocoding feature)

## development

The full engineering manual lives in [`Claude.md`](Claude.md). Phase-by-phase execution state is tracked in [`docs/current_task.md`](docs/current_task.md). Versioning policy and the full changelog are in [`docs/versioning.md`](docs/versioning.md).

## license

[MIT](LICENSE) — © 2026 allmightychaos.
