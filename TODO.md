# TODO

> Roadmap aligned to SemVer. `v0.x` = pre-alpha/alpha, no stability guarantees. `v1.0.0` = first stable API.
>
> `[RESEARCH]` — requires verification or prototyping before implementation.
> `[DECISION]` — blocked on an architectural choice in [ARCHITECTURE.md](ARCHITECTURE.md).
> `[BLOCKED]` — depends on another item being completed first.
> `[LEGAL]` — requires licensing review before proceeding.
>
> **Changelog**:
> - `2026-03-26` — Added Phase 0 project foundation; added FDRL tag namespace decision to v0.3.0; added `## v1.x — FDRL Logging` section.
> - `2026-03-26 (v2)` — Reviewed against project knowledge section 13 (Rust ML ecosystem position). No substantive TODO changes required — viz crate has no NN backend dependency by design. Framework-agnostic `SummaryWriter` interface confirmed correct per ecosystem positioning.

---

## Phase 0 — Project Foundation

**Goal**: Establish the repository as a credible, contribution-ready OSS project before any functional code ships. These items are prerequisites for v0.1.0 — nothing is merged to `main` until Phase 0 is complete.

### Repository Structure
- [x] Initialize repository with standard layout:
  ```
  .github/
    workflows/
    ISSUE_TEMPLATE/
    PULL_REQUEST_TEMPLATE.md
    CODEOWNERS
  examples/
  proto/            <- vendored .proto files (do not add until [LEGAL] is resolved)
  src/
  tests/
    compat/         <- Python TensorBoard compatibility scripts (Tier 2 testing)
  ARCHITECTURE.md
  CHANGELOG.md
  CODE_OF_CONDUCT.md
  CONTRIBUTING.md
  LICENSE
  README.md
  SECURITY.md
  TODO.md
  ```
- [x] `Cargo.toml` with correct metadata: `name`, `version = "0.0.1"`, `edition = "2024"`, `rust-version = "1.85"`, `license = "Apache-2.0"`, `repository`, `homepage`, `description`, `keywords`, `categories`
- [x] `.gitignore` (standard Rust + editor artifacts + Python `__pycache__`, `.venv`)
- [x] `rust-toolchain.toml` pinning `stable` channel

### License
- [x] `LICENSE` — Apache-2.0 full text
- [x] SPDX identifier `Apache-2.0` in `Cargo.toml`
- [x] License header policy documented in `CONTRIBUTING.md`

### Governance Documents
- [x] `CODE_OF_CONDUCT.md` — Contributor Covenant v2.1
- [x] `CONTRIBUTING.md` — must cover:
  - Prerequisites: Rust 1.85+, `protoc` (required for `prost-build`), Python + `uv` (for Tier 2 compat testing only)
  - How to build locally (including `build.rs` proto compilation step)
  - Branching model and PR process
  - Commit message format (Conventional Commits recommended)
  - Code style: `cargo fmt`, `cargo clippy -- -D warnings`
  - What "ready to merge" means (CI green, docs on all public items, CHANGELOG entry)
  - Tier 1 vs Tier 2 testing distinction: Tier 1 runs on every PR; Tier 2 (Python compat) is manual or release-branch only
  - Issue templates for bug reports, feature requests, design questions
- [x] `SECURITY.md` — must cover:
  - Supported versions (latest `v0.x` only, no backports)
  - Private reporting via GitHub Security Advisories
  - Response SLA: acknowledge within 72 hours, triage within 7 days
  - In scope: soundness issues, incorrect CRC/framing producing silently corrupt event files, supply chain issues via `cargo audit`
  - Out of scope: TensorBoard's own bugs, theoretical-only issues

### GitHub Templates
- [x] `.github/PULL_REQUEST_TEMPLATE.md` — checklist: description, linked issue, tests added/updated, docs updated, CHANGELOG entry, Tier 2 compat test run if event format changed
- [x] `.github/ISSUE_TEMPLATE/bug_report.md` — Rust version, OS, TensorBoard version, reproduction steps
- [x] `.github/ISSUE_TEMPLATE/feature_request.md` — problem, proposed API sketch, alternatives considered
- [x] `.github/ISSUE_TEMPLATE/design_question.md` — for pre-implementation architecture discussion
- [x] `CODEOWNERS` — assign owners to `ARCHITECTURE.md`, `proto/`, `SECURITY.md`, `Cargo.toml`

### CI — GitHub Actions
- [x] `ci.yml` — runs on every push and PR to `main`:
  - `cargo fmt --check`
  - `cargo clippy -- -D warnings`
  - `cargo test`
  - `cargo doc --no-deps`
  - Matrix: `[stable, nightly]` x `[ubuntu-latest]`
- [x] `audit.yml` — push to `main` + daily schedule:
  - `cargo audit`
  - `cargo deny check` (license policy, bans, advisories)
- [x] `compat.yml` — manual trigger (`workflow_dispatch`) only:
  - Sets up Python via `uv`, installs `tensorboard`
  - Runs `tests/compat/` Python scripts to verify TensorBoard can read event files produced by the crate
  - Not a gate for every PR — run before releases or when event format changes
- [x] Cache `~/.cargo/registry` and `target/` across workflow runs
- [ ] Branch protection on `main`: all `ci.yml` checks required, at least one review required

### Changelog
- [x] `CHANGELOG.md` initialized per [Keep a Changelog](https://keepachangelog.com/) format
- [x] Policy: every PR changing behavior requires a CHANGELOG entry under `[Unreleased]`

### Supply Chain
- [x] `deny.toml` for `cargo deny`:
  - Licenses: allowlist `Apache-2.0`, `MIT`, `MIT-0`, `BSD-2-Clause`, `BSD-3-Clause`, `ISC`, `Unicode-DFS-2016`
  - Bans: deny duplicate crate versions where avoidable
  - Advisories: deny all known vulnerabilities
- [x] Note: the `proto/` vendoring decision (below) has supply chain implications — `.proto` files are not Rust crates and not covered by `cargo deny`; document the provenance explicitly in `proto/README.md`

### README Polish
- [x] Badges rendering correctly: crates.io, docs.rs, license, CI status
- [x] `protoc` installation noted as a build prerequisite
- [x] Links to `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md`

---

## Pre-v0.1.0 — Prerequisite Research

**These two items are hard blockers. No implementation begins until both are resolved.**

- [x] `[LEGAL]` Confirm licensing of deriving protobuf definitions from TensorFlow `.proto` files (Apache-2.0). The `.proto` files are Apache-2.0 licensed; the question is whether vendoring them and compiling derived Rust structs via `prost-build` creates any license obligation beyond attribution. Needs explicit review — do not assume it is fine.
- [x] `[RESEARCH]` Verify TensorBoard protobuf schema against current TensorBoard source (>= v2.15). Confirm the `event.proto` / `summary.proto` schema has not changed in a breaking way. Document the exact TensorBoard version range this crate targets.
- [x] `[RESEARCH]` Verify masked CRC32C implementation matches TensorFlow's exact formula against published test vectors
- [x] `[RESEARCH]` Verify `tensorboard-rs` as prior art reference — do not copy, but understand what it got right and wrong

---

## v0.1.0 — Scalar Writing

**Goal**: Write scalar events readable by TensorBoard. The minimum bar for a training loop to be observable.

### Project Setup
- [ ] Vendor required `.proto` files into `proto/` directory
- [ ] `proto/README.md` documenting source URLs, upstream commit hash, and license attribution
- [ ] Set up `build.rs` with `prost-build` for proto compilation
- [ ] Add dependencies: `prost`, `thiserror`, `crc32c`
- [ ] Add build dependencies: `prost-build`
- [ ] `#![deny(missing_docs)]` enforced from v0.1.0

### Module Structure
- [ ] `src/lib.rs` — public API re-exports
- [ ] `src/error.rs` — `VizError` enum via `thiserror`
- [ ] `src/crc.rs` — masked CRC32C implementation
- [ ] `src/event.rs` — `EventFileWriter` (low-level record framing)
- [ ] `src/writer.rs` — `SummaryWriter` (high-level API)

### Event File Writer
- [ ] Implement `EventFileWriter` with correct record framing (length + CRC + data + CRC)
- [ ] Masked CRC32C implementation (verified against TensorBoard test vectors)
- [ ] Protobuf event serialization via `prost`
- [ ] `BufWriter` wrapping for buffered I/O
- [ ] Reusable scratch buffer for protobuf encoding (avoid per-event allocation)
- [ ] File rotation: new event file per `SummaryWriter` instantiation
- [ ] `flush()` with `BufWriter::flush()` + `File::sync_all()`

### SummaryWriter
- [ ] `SummaryWriter::new(log_dir)` — creates log directory if not exists
- [ ] `add_scalar(tag, value, step)`
- [ ] `flush()`
- [ ] Wall-clock timestamp on each event via `SystemTime::now()`
- [ ] Input validation: empty tags, special characters, NaN/Inf values, step bounds

### Testing — Tier 1 (Rust-only, CI gate)
- [ ] CRC32C unit tests against TensorFlow's known test vectors
- [ ] CRC32C edge cases: empty input, single byte, large input, masking step
- [ ] Protobuf serialization unit tests
- [ ] Event record framing: write then read-back verification
- [ ] Minimal event file reader (roundtrip testing only, not public API)
- [ ] Malformed input tests: empty tags, NaN/Inf values, step edge cases
- [ ] `proptest` roundtrip: random scalar events write-then-read correctly

### Testing — Tier 2 (Python, `compat.yml`, manual trigger)
- [ ] `tests/compat/` Python script using `tensorboard.backend.event_processing.event_file_loader`
- [ ] Verify TensorBoard can read event files produced by `torchforge-viz`
- [ ] Script managed via `uv` with a pinned `pyproject.toml` — no loose pip installs

### Documentation
- [ ] Doc comments on all public items
- [ ] Example: minimal scalar logging loop in `examples/`

---

## v0.2.0 — Histograms, Images & Multi-Scalar

**Goal**: Full parity with `tensorboard-rs` feature set, correctly implemented.

- [ ] `[RESEARCH]` Verify TensorBoard histogram bucket schema exactly against TensorBoard source
- [ ] `[DECISION]` Choose fixed (TensorBoard-matching) vs. dynamic histogram buckets
- [ ] `add_histogram(tag, values, step)`
- [ ] `add_image(tag, image, step)` — PNG encoding via `image` crate
- [ ] `add_images(tag, images, step)` — grid layout
- [ ] `add_scalars(tag, map, step)` — requires multi-writer with sub-tag directories + LRU file handle management (see ARCHITECTURE.md)

---

## v0.3.0 — RL-Native Metrics

**Goal**: Typed metric helpers for RL training loops.

- [ ] `[DECISION]` Design typed metric API layer — above `SummaryWriter` or integrated?
- [ ] `[DECISION]` *(added 2026-03-26)* Tag namespace convention for FDRL: decide `local/<metric>` vs. `global/<metric>` convention before finalizing `RLLogger` tag strings. This must be settled at v0.3.0 — before torchforge-federated exists — so that v1.x logging is additive not a rename. See ARCHITECTURE.md `## FDRL Logging Considerations`.
- [ ] `RLLogger` struct with methods:
  - [ ] `log_episode(return, length, step)`
  - [ ] `log_policy(loss, entropy, kl_div, step)`
  - [ ] `log_value(loss, explained_variance, step)`
  - [ ] `log_buffer(size, capacity, samples_per_sec, step)`
  - [ ] `log_system(memory_bytes, step_time_ms, step)`
- [ ] Standardized tag naming convention documented — must include `local/` prefix decision for FDRL forward compatibility

---

## v0.4.0 — Terminal UI (TUI) Mode

**Goal**: Live training visibility on edge devices without a display server.

- [ ] `[RESEARCH]` Verify `ratatui` works on target edge platforms (Raspberry Pi, Jetson)
- [ ] `[RESEARCH]` Verify TUI viability on serial-only terminals
- [ ] `[DECISION]` In-process vs. threaded TUI architecture
- [ ] `TuiWriter` implementing same interface as `SummaryWriter`
- [ ] Live scalar sparkline
- [ ] Episode return rolling average display
- [ ] Replay buffer fill indicator
- [ ] Graceful degradation when terminal capabilities are limited

---

## v0.5.0 — Multi-Writer & Filtering

**Goal**: Production ergonomics for longer training runs.

- [ ] `MultiWriter` — fan-out to multiple backends simultaneously (file + TUI)
- [ ] Tag filtering — log only specified tags to reduce I/O on constrained hardware
- [ ] Step-based decimation — log every N steps to reduce file size

---

## v1.0.0 — Stable API

**Gate criteria** (all must be met):
- [ ] Scalar, histogram, image writing verified against TensorBoard >= v2.15
- [ ] At least one real RL training run using `torchforge-viz` with published output
- [ ] TUI mode verified on at least one edge target
- [ ] Zero known correctness issues in event file format
- [ ] MSRV declared and tested in CI
- [ ] `#![deny(missing_docs)]` enforced
- [ ] *(added 2026-03-26)* `local/` vs. `global/` tag namespace convention finalized and documented — FDRL forward compatibility confirmed

---

## v1.x — FDRL Logging *(added 2026-03-26)*

**Goal**: Support federated training metric groups — per-device local policy metrics and global aggregated policy metrics — as first-class `RLLogger` extensions in torchforge-federated.

No code for this ships in torchforge-viz. The contribution of this crate to the FDRL story is:
- A stable, per-device `SummaryWriter` that each federated agent logs to independently
- A `MultiWriter` (v0.5.0) that the federation coordinator can use as a precursor for fan-out logging
- A tag namespace convention that makes `local/` and `global/` log streams unambiguously distinguishable in TensorBoard

The FDRL-specific metric groups (federation round, gradient statistics, communication cost, global policy return) will be implemented in torchforge-federated as an extension of the `RLLogger` API. They depend on this crate's stable `SummaryWriter` interface.

### What this crate must provide before v1.x FDRL work begins
- [ ] `SummaryWriter` stable API (v1.0.0)
- [ ] `RLLogger` with `local/` tag namespace convention documented (v0.3.0)
- [ ] `MultiWriter` available (v0.5.0)

### What lives in torchforge-federated, not here
- `FederationLogger` extending `RLLogger` with federation metric groups
- Gradient statistics logging
- Per-round communication cost logging
- Global policy evaluation logging

---

## Ongoing

- [ ] Keep dependencies on latest stable versions
- [ ] `cargo audit` clean at all times
- [ ] `cargo clippy -- -D warnings` clean at all times
- [ ] CHANGELOG.md maintained per [Keep a Changelog](https://keepachangelog.com/)
