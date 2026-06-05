# Zenith Kernel

## Vision and Purpose

**Zenith Kernel** is the flagship kernel of the **Deepcomet AI** ecosystem. It unites the original Deepcomet civilization framing ambitions with pragmatic, modern kernel engineering. Zenith is designed to be the foundational runtime for AI native systems, distributed compute fabrics, and long lived infrastructure that advances human capability while prioritizing safety, verifiability, and modular evolution.

**Primary goals**
- **Civilization framing** — provide a resilient, auditable, and extensible kernel that can scale from edge devices to datacenter fabrics and support long term societal infrastructure.
- **Post Unix foundation** — replace legacy assumptions with capability handles, object semantics, and explicit resource contracts.
- **AI native runtime** — first class scheduling and resource management for NPU and GPU workloads.
- **Practical adoption** — interoperate with existing ecosystems through optional compatibility layers while keeping the core clean and future ready.

---

## Combined Design Principles

### Core Principles
- **Microkernel minimalism** — kernel responsibilities limited to scheduling, memory protection, capability enforcement, and secure IPC.
- **Capability security** — unforgeable handles for all resources, least privilege by default, delegation and revocation primitives.
- **Memory safety** — Rust for core subsystems and drivers, C only where hardware constraints demand it and isolated behind verified boundaries.
- **Object oriented drivers** — lifecycle managed, isolated drivers with explicit capability interfaces.
- **Async first IPC** — channels and message semantics inspired by modern kernels for low latency and composability.
- **Optional compatibility** — POSIX and Linux ABI provided as user space services, not kernel primitives.
- **AI native scheduling** — workload classification, NPU and GPU affinity, QoS policies, and predictive scheduling hooks.
- **Formal assurance** — incremental verification for IPC, capability enforcement, and memory isolation.

### Design Tradeoffs and Rationale
- **Safety over legacy convenience** — prefer safe abstractions even if it increases short term porting cost.
- **Modularity over monolith** — subsystems run as isolated agents to reduce blast radius and enable independent verification.
- **Pragmatism over purity** — maintain compatibility shims to accelerate adoption while keeping them clearly separated from core semantics.

---

## Architecture Overview and Primitives

### High Level Architecture
- **Core microkernel** in `core/`  
  - **Scheduler** — multi policy, pluggable, AI aware.  
  - **Memory manager** — capability backed regions, zero copy grants.  
  - **IPC subsystem** — asynchronous channels, synchronous RPC adapters.  
  - **Capability manager** — creation, delegation, revocation, auditing.

- **User space services**  
  - **Agent manager** — lifecycle for agents and service agents.  
  - **Driver host** — Rust driver runtime with object model and sandboxing.  
  - **Networking service** — FreeBSD derived stack wrapped in capabilities.  
  - **Storage service** — pluggable filesystem backends and object stores.  
  - **Compatibility services** — POSIX server, Linux ABI shim, container runtime.

- **Submodule federation**  
  - Each upstream fork lives as a Git submodule and is adapted via a thin adapter layer to Zenith primitives.

### Key Primitives
- **Agent** — execution unit replacing the traditional process model. Agents own capabilities and run in isolated address spaces.
- **Channel** — typed asynchronous IPC primitive for message passing and capability transfer.
- **Object** — kernel visible resource abstraction for devices, memory regions, and services.
- **Capability** — unforgeable token granting specific rights to an object, supports delegation and revocation.

### Example Interaction Flow
1. Agent A requests a device capability from a device manager service via a channel.  
2. Device manager grants a capability token to Agent A.  
3. Agent A uses the capability to map device memory or send commands through an object interface.  
4. Capability revocation is enforced by the kernel and audited.

---

## Integration Strategy and Submodules

### Submodule Palette
| **Subsystem** | **Role** | **Source** |
|---|---:|---|
| **Mono‑Zenith** | Linux drivers and tooling | `DeepcometAI/mono-zenith` |
| **seL4 / Perception** | Capability model and verification experiments | `seL4` or `DeepcometAI/perception` |
| **Zenith‑XNU** | Hybrid Mach and I/O Kit ideas | `DeepcometAI/zenith-xnu` |
| **FreeBSD** | Networking and filesystem subsystems | `freebsd/freebsd-src` |
| **Fuchsia (Zircon)** | Handle model and async IPC primitives | `fuchsia` with `zircon/` subtree |

### Integration Patterns
- **Adapter layer** — small, well documented adapters translate upstream APIs into Zenith primitives.
- **Pin and audit** — pin submodules to specific commits and maintain a changelog for each pin update.
- **Isolation boundary** — run upstream subsystems as user space services where possible to limit kernel complexity.
- **Gradual replacement** — replace unsafe components with Rust equivalents incrementally while keeping functional parity.

### Submodule Management Policy
- Keep submodule paths under `submodules/` with one commit pinned per release.  
- Maintain a `submodule-pins/` manifest that records commit, tag, license, and upstream URL for auditing.  
- Use semantic tags for cross subsystem releases, for example `zenith-core-0.1` and `zenith-freebsd-0.1`.

---

## Roadmap, Releases, and Milestones

### Release Strategy
- **Alpha** `zenith-0.1` — core microkernel primitives, basic agent lifecycle, simple channel IPC, CI for x86_64.
- **Beta** `zenith-0.5` — capability manager, Rust driver host prototype, FreeBSD networking service integration.
- **Stable** `zenith-1.0` — verified IPC invariants, POSIX compatibility service, multi arch CI including ARM64.
- **Ecosystem** `zenith-2.x` — AI scheduling primitives, NPU and GPU resource manager, verified subsystems.

### Phased Roadmap
- **Phase 0 Foundation** — define primitives, bootstrap Rust toolchain, minimal scheduler and IPC.
- **Phase 1 Subsystems** — integrate FreeBSD networking, add Rust driver host, add Mono‑Zenith drivers.
- **Phase 2 Compatibility** — POSIX server, Linux ABI shim, container support.
- **Phase 3 Assurance** — formal proofs for IPC and capability enforcement, fuzzing harnesses.
- **Phase 4 AI Integration** — workload classification, predictive scheduling, hardware offload orchestration.
- **Phase 5 Governance** — contributor governance, long term maintenance model, ecosystem partnerships.

### Tagging and Release Checklist
- **Tagging convention** — `zenith-core-X.Y`, `zenith-<subsystem>-X.Y`.  
- **Release checklist** — tests green, security audit, documentation updated, submodule pins recorded, license audit complete.

---

## Development Workflow, CI, Testing, and Onboarding

### CI CD Principles
- **Per submodule pipelines** — build and test each submodule independently.
- **Integration pipeline** — link adapters and run end to end smoke tests.
- **Matrix builds** — x86_64 and ARM64 baseline, optional RISC V later.
- **Fail fast** — isolate failing submodule builds from core pipeline until adapter compatibility is validated.

### Example GitHub Actions Stages
1. **Checkout** with `--recurse-submodules`.  
2. **Build** core microkernel in Rust with strict lints and MIRI where applicable.  
3. **Build** each submodule using pinned commits.  
4. **Integration** run QEMU based smoke tests for agent lifecycle and IPC.  
5. **Fuzzing** run libFuzzer jobs against IPC and capability interfaces.  
6. **Verification** run formal checks for verified components.

### Testing Matrix
- **Unit tests** for primitives and adapters.  
- **Integration tests** for agent to service flows.  
- **System tests** in QEMU for boot, agent lifecycle, and basic networking.  
- **Fuzz tests** for IPC channels and capability serialization.  
- **Formal proofs** for critical invariants where feasible.

### Developer Onboarding
```bash
# clone with submodules
git clone --recurse-submodules https://github.com/DeepcometAI/zenith.git

# initialize missing submodules
git submodule update --init --recursive

# build core locally
cd core
cargo build --release

# run QEMU dev image
make qemu
```
- **Local dev** target `make dev-image` builds a QEMU image with a minimal agent and test harness.  
- **Debugging** includes integrated logging, capability audit traces, and kernel live debug hooks.  
- **Docs** include step by step contributor guide, architecture reference, and API docs for primitives.

---

## Security, Verification, Governance, and Licensing

### Security Model
- **Least privilege by default** — agents start with minimal capabilities.  
- **Capability revocation** — immediate and auditable revocation paths.  
- **Audit trails** — immutable logs for capability grants and revocations.  
- **Runtime monitors** — immune system inspired watchdogs for anomaly detection.

### Verification Strategy
- **Formal verification** for IPC and capability enforcement using seL4 techniques where practical.  
- **Incremental proofs** — start with small, critical components and expand.  
- **Automated proofs in CI** — run verification jobs for tagged releases.  
- **Third party audits** — invite external security reviews for major releases.

### Governance Model
- **Core maintainers** — small trusted team for kernel core and primitives.  
- **Subsystem maintainers** — owners for each submodule fork.  
- **Contribution process** — PRs, code review, CI green, security signoff for changes touching primitives.  
- **Roadmap council** — cross project group that approves major design changes and release milestones.

### Licensing and Legal
- **Zenith core** — permissive license to be chosen and documented in `LICENSE`.  
- **Upstream submodules** — retain original licenses; maintain `THIRD_PARTY_LICENSES.md` with SPDX identifiers.  
- **Contributor License Agreement** — optional CLA for corporate contributors to clarify IP.  
- **Compliance** — license audit on each release and export control documentation for cryptography.

---

## Aurelia Programming Language and Future Runtimes

### Aurelia Overview
**Aurelia** is the planned systems programming language and runtime for future Zenith versions. Aurelia is designed to be a safe, expressive language for writing agents, system services, and verified subsystems. Aurelia focuses on:
- **Memory safety with deterministic control** — ownership and capability aware types that map naturally to kernel primitives.
- **Capability first types** — language level constructs for capability creation, transfer, and revocation.
- **Formal friendly semantics** — language features chosen to simplify formal reasoning and automated verification.
- **Async native concurrency** — first class async primitives that map directly to Zenith channels and scheduling hooks.
- **Interoperability** — seamless FFI with Rust and controlled interop with C for hardware boundaries.

### Aurelia Roadmap
- **Phase A** — language design, core type system, and compiler prototype that emits LLVM IR or MLIR.
- **Phase B** — runtime integration with Zenith agent model, channel bindings, and capability types.
- **Phase C** — toolchain for verification, model extraction for formal proofs, and standard library for system services.
- **Phase D** — migration path for critical subsystems from Rust to Aurelia where verification and expressiveness provide clear benefits.

### How Aurelia Fits Zenith
- **Primary language for future agents and services** — Aurelia becomes the recommended language for writing high assurance services and AI native schedulers.
- **Coexists with Rust** — Rust remains the primary language for many subsystems during transition. Interop layers and adapters enable mixed language stacks.
- **Verification synergy** — Aurelia’s semantics are designed to simplify formal proofs for capability and IPC invariants.

---

## Contribution, Community, and Next Steps

### How to Contribute
- Fork the repo and submit pull requests.  
- Follow coding guidelines in `CONTRIBUTING.md` with Rust preferred and C only at hardware boundaries.  
- Respect subsystem boundaries and upstream submodule policies.  
- Sign the CLA if required for corporate contributions.

### Maintainer Roles
- **Core maintainers** — review and merge changes to `core/`.  
- **Subsystem maintainers** — manage submodule forks and adapter layers.  
- **Security maintainers** — triage and patch security issues.

### Immediate Next Steps
- Fork and pin chosen upstream subsystems into `submodules/`.  
- Bootstrap `core/` with Rust toolchain, scheduler prototype, and channel IPC.  
- Create CI pipelines that build core and run QEMU smoke tests.  
- Publish contributor guidelines and the `THIRD_PARTY_LICENSES.md` file.  
- Start Aurelia language design repository and link it as a future submodule.

---

## Glossary

- **Agent** — execution unit owning capabilities.  
- **Channel** — async IPC primitive.  
- **Object** — kernel resource abstraction.  
- **Capability** — unforgeable token granting rights.  
- **Adapter** — translation layer between an upstream subsystem and Zenith primitives.  
- **Submodule pin** — recorded commit used for reproducible builds.

---

## Contact and Governance Links

- **Repository** — `https://github.com/DeepcometAI/zenith`  
- **Issue tracker** — use repository Issues for bugs and feature requests.  
- **Mailing list** — see `docs/COMMUNITY.md` for governance and council contact points.

---