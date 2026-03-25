# TODO

> Roadmap aligned to SemVer. `v0.x` = pre-alpha/alpha, no stability guarantees. `v1.0.0` = first stable API.
>
> `[RESEARCH]` — requires verification or prototyping before implementation.
> `[DECISION]` — blocked on an architectural choice in [ARCHITECTURE.md](ARCHITECTURE.md).
> `[BLOCKED]` — depends on another item being completed first.
> `[LEGAL]` — requires licensing review before proceeding.

---

## Pre-v0.1.0 — Prerequisite Research

These must be resolved before any implementation begins.

- [ ] `[RESEARCH]` Verify TensorBoard protobuf schema against current TensorBoard source (>= v2.15)
- [ ] `[LEGAL]` Confirm licensing of deriving protobuf definitions from TensorFlow `.proto` files (Apache-2.0)
- [ ] `[RESEARCH]` Verify masked CRC32C implementation matches TensorFlow's exact formula
- [ ] ~~`[DECISION]` Choose protobuf crate~~ **Decided**: `prost` + `prost-build` (see ARCHITECTURE.md)
- [ ] `[RESEARCH]` Verify `tensorboard-rs` correctness as prior art reference — do not copy, but learn from

---

## v0.1.0 — Scalar Writing

**Goal**: Write scalar events readable by TensorBoard. The minimum bar for a training loop to be observable.

### Project Setup
- [ ] Add `rust-version = "1.85"` to Cargo.toml (MSRV for edition 2024)
- [ ] Vendor required `.proto` files into `proto/` directory
- [ ] Set up `build.rs` with `prost-build` for proto compilation
- [ ] Add dependencies: `prost`, `thiserror`, `crc32c`
- [ ] Add build dependencies: `prost-build`

### Module Structure
- [ ] `src/lib.rs` — public API re-exports
- [ ] `src/error.rs` — `VizError` enum via `thiserror`
- [ ] `src/crc.rs` — masked CRC32C implementation
- [ ] `src/event.rs` — `EventFileWriter` (low-level record framing)
- [ ] `src/writer.rs` — `SummaryWriter` (high-level API)

### Event File Writer
- [ ] Implement `EventFileWriter` with correct record framing (length + CRC + data + CRC)
- [ ] Masked CRC32C implementation (verified against TensorBoard)
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
- [ ] Minimal event file reader (for roundtrip testing only, not public API)
- [ ] Malformed input tests: empty tags, NaN/Inf values, step edge cases
- [ ] `proptest` roundtrip: random scalar events write-then-read correctly

### Testing — Tier 2 (Python, separate CI job)
- [ ] `tests/compat/` Python script using `tensorboard.backend.event_processing.event_file_loader`
- [ ] Verify TensorBoard can read event files produced by `torchforge-viz`
- [ ] Not a gate for every PR — run on release branches or manually

### CI
- [ ] GitHub Actions: stable + nightly Rust
- [ ] `cargo clippy -- -D warnings`
- [ ] `cargo test`
- [ ] `cargo audit`

### Documentation
- [ ] Doc comments on all public items
- [ ] Example: minimal scalar logging loop in `examples/`

---

## v0.2.0 — Histograms, Images & Multi-Scalar

**Goal**: Full parity with `tensorboard-rs` feature set, correctly implemented. Plus `add_scalars` deferred from v0.1.0.

- [ ] `[RESEARCH]` Verify TensorBoard histogram bucket schema exactly
- [ ] `[DECISION]` Choose fixed vs. dynamic histogram buckets
- [ ] `add_histogram(tag, values, step)`
- [ ] `add_image(tag, image, step)` — PNG encoding via `image` crate
- [ ] `add_images(tag, images, step)` — grid of images
- [ ] `add_scalars(tag, map, step)` — multiple scalars in one plot (requires multi-writer with sub-tag directories, see ARCHITECTURE.md for file handle management strategy)

---

## v0.3.0 — RL-Native Metrics

**Goal**: Typed metric helpers for RL training loops. Reduces boilerplate and standardizes tag naming.

- [ ] `[DECISION]` Design typed metric API layer above `SummaryWriter`
- [ ] `RLLogger` struct with methods:
  - [ ] `log_episode(return, length, step)`
  - [ ] `log_policy(loss, entropy, kl_div, step)`
  - [ ] `log_value(loss, explained_variance, step)`
  - [ ] `log_buffer(size, capacity, samples_per_sec, step)`
  - [ ] `log_system(memory_bytes, step_time_ms, step)`
- [ ] Standardized tag naming convention (documented)

---

## v0.4.0 — Terminal UI (TUI) Mode

**Goal**: Live training visibility on edge devices without a display server.

- [ ] `[RESEARCH]` Verify `ratatui` works on target edge platforms (Raspberry Pi, Jetson)
- [ ] `[RESEARCH]` Verify TUI viability on serial-only terminals
- [ ] `[DECISION]` In-process vs. threaded TUI architecture
- [ ] `TuiWriter` implementing same interface as `SummaryWriter`
- [ ] Live scalar plot (sparkline)
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
- [ ] MSRV declared and tested
- [ ] `#![deny(missing_docs)]` enforced

---

## Ongoing

- [ ] Keep dependencies on latest stable versions
- [ ] `cargo audit` clean at all times
- [ ] `cargo clippy -- -D warnings` clean at all times
- [ ] CHANGELOG.md maintained per [Keep a Changelog](https://keepachangelog.com/)
