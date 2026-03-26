# Architecture

> Living document. Decisions marked `[OPEN]` are not yet settled. No assumption is papered over.
>
> **Changelog**:
> - `2026-03-26` — Added FDRL metric groups to `## RL-Native Metrics`; new `## FDRL Logging Considerations` section; updated `## Out of Scope` to reflect v1.x federation logging target.
> - `2026-03-26 (v2)` — Added `## Rust ML Ecosystem Context` note from project knowledge section 13; confirmed viz approach is independent of NN backend choice.

---

## Problem Statement

A Rust ML training loop — specifically an on-device RL loop — needs to emit observable signals:

1. Scalar metrics over time (episode return, loss, entropy)
2. Distributions (action histograms, weight distributions)
3. Images (optional, for vision-based RL)
4. On-device terminal output for edge targets with no display server

The output must be consumable by existing tooling (TensorBoard) without requiring Python in the training process.

---

## TensorBoard Event File Format

TensorBoard reads from event files — binary files written as a sequence of length-prefixed, CRC-checked protobuf records.

**Known**: The format is stable and documented. TensorBoard itself ships a Rust reader (RustBoard, `tensorboard/data/server`) that reverse-confirms the format is parseable from Rust.

**Known**: The relevant protobuf schemas are in the TensorFlow repository under `tensorflow/core/util/event.proto` and `tensorflow/core/framework/summary.proto`.

**[OPEN]**: Whether the schema has changed in recent TensorBoard versions (>2.15) in a breaking way — needs verification against current TensorBoard source before implementation.

**[OPEN]**: Licensing implications of deriving protobuf definitions from TensorFlow's `.proto` files (Apache-2.0 licensed) — needs legal review before publishing generated code.

---

## Protobuf Strategy

**Decided**: Use `prost` + `prost-build` for protobuf serialization. `prost` generates idiomatic Rust structs, is the community standard (used by `tonic`/tokio ecosystem), and is actively maintained.

**Decided**: Vendor `.proto` files in a `proto/` directory. `build.rs` compiles them at build time via `prost-build`. This requires users to have `protoc` installed (standard for protobuf projects) and avoids the "generated code drifted from source" class of bugs.

Proto files to vendor (minimal set for event file writing):
- `tensorflow/core/util/event.proto`
- `tensorflow/core/framework/summary.proto`
- `tensorflow/core/framework/tensor.proto`
- `tensorflow/core/framework/tensor_shape.proto`
- `tensorflow/core/framework/types.proto`
- `tensorflow/core/framework/resource_handle.proto`

---

## Core Abstractions

### `SummaryWriter`

The primary interface. Wraps an event file writer and exposes typed methods for each supported summary type.

```rust
pub struct SummaryWriter {
    writer: EventFileWriter,
    log_dir: PathBuf,
}

impl SummaryWriter {
    pub fn new(log_dir: impl AsRef<Path>) -> Result<Self, VizError>;
    pub fn add_scalar(&mut self, tag: &str, value: f32, step: u64) -> Result<(), VizError>;
    pub fn add_histogram(&mut self, tag: &str, values: &[f32], step: u64) -> Result<(), VizError>;
    pub fn add_image(&mut self, tag: &str, image: &ImageData, step: u64) -> Result<(), VizError>;
    pub fn flush(&mut self) -> Result<(), VizError>;
}
```

**Known**: This interface mirrors the Python `SummaryWriter` API closely — intentional, reduces cognitive overhead for practitioners migrating from Python.

**Decided**: `add_scalars` (multiple scalars in one plot) is deferred to v0.2.0. The PyTorch-compatible implementation requires multiple event file writers (one per sub-tag directory), adding significant complexity. `add_scalar` covers the primary use case; `add_scalars` will be implemented correctly rather than rushed.

---

### `EventFileWriter`

Low-level writer. Serializes events to the TensorBoard binary format.

Record format per event:
```
[ uint64 data_length ][ uint32 masked_crc32_of_length ][ bytes data ][ uint32 masked_crc32_of_data ]
```

**Known**: The masked CRC32 uses a specific masking function from TensorFlow's `crc32c`:
```
masked_crc = ((crc >> 15) | (crc << 17)) + 0xa282ead8
```
This is verified against the TensorFlow source.

#### Buffering Strategy

**Decided**: Wrap the underlying `File` in `BufWriter` (default 8KB buffer). Scalar events are ~50-100 bytes, so ~80-160 events buffer before an automatic OS write. `flush()` calls `BufWriter::flush()` then `File::sync_all()` for explicit fsync.

#### Serialization Buffer Reuse

**Decided**: Store a reusable `Vec<u8>` scratch buffer in `EventFileWriter`. Clear it on each `write_event` call and encode into it. The vec grows to max event size and stays — no per-event allocation after warmup. This matters for high-frequency RL logging loops.

---

### Error Handling

**Decided**: Use `thiserror` for the `VizError` enum. Provides matchable error variants with minimal boilerplate — the standard approach for Rust libraries.

```rust
#[derive(Debug, thiserror::Error)]
pub enum VizError {
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),

    #[error("protobuf encode error: {0}")]
    Encode(#[from] prost::EncodeError),

    #[error("invalid tag: {0}")]
    InvalidTag(String),
}
```

---

### Module Structure

**Decided**: Flat module layout for v0.x scope.

```
src/
  lib.rs          — re-exports public API (SummaryWriter, VizError)
  writer.rs       — SummaryWriter implementation
  event.rs        — EventFileWriter (low-level record framing)
  crc.rs          — masked CRC32C
  error.rs        — VizError enum
proto/            — vendored .proto files (compiled by build.rs)
```

Public API surface: `SummaryWriter`, `VizError`, and the `add_*` / `flush` methods. Everything else is `pub(crate)`. Nest into subdirectories if module count exceeds ~8 files.

---

### Histogram Approximation

TensorBoard histograms use a specific bucket schema. Two options:

1. **TensorBoard default buckets** — fixed bucket boundaries matching TensorBoard's Python implementation. Ensures identical rendering.
2. **Dynamic buckets** — computed from data range. Simpler but may render differently in TensorBoard.

**[OPEN]**: Which approach to use. Option 1 is preferred for compatibility but requires implementing TensorBoard's exact bucket schema. Needs verification.

---

### Terminal UI (TUI) Mode

For edge devices without a display server, a terminal-based live view of training metrics.

```
Episode:  1,240 | Return:  -12.3 | Loss: 0.043 | Buffer: 18,432/50,000
█████████████████████░░░░░░░░░░░░░░░░░░░░  Return (last 100 episodes)
```

**[OPEN]**: Crate selection. Candidates:
- `ratatui` — actively maintained, successor to `tui-rs`
- `crossterm` — lower level, more control, less abstraction

**[OPEN]**: Whether TUI mode is in-process (same thread as training) or a separate thread reading from a channel. In-process risks blocking the training loop. Separate thread adds complexity. Decision deferred.

**[OPEN]**: Whether TUI mode is viable on all target edge platforms (no-std targets, serial-only terminals). Needs hardware verification.

---

## RL-Native Metrics

Standard ML visualizers treat all scalars equally. RL training has specific metric groups that benefit from structured logging:

| Group | Metrics |
|---|---|
| Environment | episode return, episode length, success rate |
| Policy | policy loss, entropy, KL divergence |
| Value | value loss, explained variance |
| Data | replay buffer size, samples per second |
| System | memory usage, step time |

`torchforge-viz` will provide typed helpers for these groups rather than requiring users to manually construct tag strings.

**[OPEN]**: Whether typed metric groups should be a separate higher-level API layer above `SummaryWriter`, or integrated directly. Design deferred.

<!-- v2: 2026-03-26 — FDRL metric groups added -->
### FDRL Metric Groups *(added 2026-03-26)*

When the training loop is federated (torchforge-federated, v1.x), additional metric groups become meaningful. These are **not v0.x implementation targets** — they are captured here so the v0.3.0 `RLLogger` API is designed to accommodate them without a breaking change.

| Group | Metrics | Notes |
|---|---|---|
| Federation | federation round, devices participating, devices dropped | One event per aggregation round |
| Local vs. global policy | local episode return, global policy return, policy divergence (KL) | Per-device logging; global is post-aggregation |
| Gradient statistics | gradient norm (pre-aggregation), gradient norm (post-aggregation), gradient staleness | Critical for diagnosing FedAvg convergence issues |
| Communication | bytes sent per round, aggregation latency | Edge-relevant: communication cost is a hard constraint |

**Design implication for v0.3.0 `RLLogger`**: The typed metric API must use a tag namespace scheme that naturally extends to federation. Proposed convention: `local/<metric>` for per-device metrics, `global/<metric>` for aggregated metrics. This must be decided and documented at v0.3.0, before federation exists, so that v1.x logging is additive rather than a rename.

**[OPEN]**: Tag namespace convention for local vs. global metrics — decision deferred to v0.3.0 design, but must account for FDRL before finalizing.

---

## Performance Considerations

### Wall-Clock Timestamps

**Decided**: Use `SystemTime::now()` directly. On Linux, `clock_gettime` is a vDSO call (~20ns). At 10K events/sec, that's 0.2ms — negligible. No need for optional user-provided timestamps.

### Multi-Writer File Handle Management (v0.2.0+)

When `add_scalars` is implemented, the PyTorch-compatible approach requires multiple event file writers (one per sub-tag directory). This could exhaust file descriptors for many sub-tags. Preferred approach: lazy writer creation + LRU eviction of file handles. Design details deferred to v0.2.0.

---

## Dependency Decisions

| Crate | Version | Purpose | Status |
|---|---|---|---|
| `prost` | latest stable | Protobuf serialization | **Decided** |
| `prost-build` | latest stable | Build-time proto compilation | **Decided** |
| `thiserror` | latest stable | Error type derive macros | **Decided** |
| `crc32c` | latest stable | CRC32C checksums | Confirmed correct algorithm |
| `ratatui` | latest stable | TUI mode | Tentative — pending hardware verification |

---

## What We Explicitly Do Not Know

1. **TensorBoard protobuf schema stability** across recent versions — needs verification
2. **Licensing of derived protobuf definitions** from TensorFlow — needs legal review
3. **TUI viability on constrained edge targets** — needs hardware testing
4. **Histogram bucket schema** — needs exact verification against TensorBoard source
5. **In-process vs. threaded TUI** — design decision not yet made
6. **Tag namespace convention for FDRL local vs. global metrics** *(added 2026-03-26)* — must be decided at v0.3.0 before federation logging is needed

---

<!-- v2: 2026-03-26 — FDRL logging considerations section added -->
## FDRL Logging Considerations *(added 2026-03-26)*

In the federated RL scenario, each edge device runs its own `SummaryWriter` logging local training progress. A federation coordinator additionally needs to log global policy metrics after each aggregation round. These are two distinct log streams, not one.

**Implication for v0.x design:**
- The `SummaryWriter` interface as designed (one writer per log directory) is already compatible with this model — each device writes to its own `runs/device_<id>/` directory; the coordinator writes to `runs/global/`.
- No API changes are required at v0.x to support FDRL logging. The constraint is namespace discipline in the tag convention (see `### FDRL Metric Groups` above).
- The `MultiWriter` planned for v0.5.0 (fan-out to file + TUI simultaneously) is a precursor capability for the federation case where a single training step must log to both a local file and a federation-bound gradient buffer.

**What FDRL logging does NOT require from v0.x:**
- No network-aware writer
- No shared-state between device writers
- No aggregation logic in this crate

Federation logging infrastructure belongs in torchforge-federated (v1.x). This crate provides the per-device primitive it builds on.

---

## Rust ML Ecosystem Context *(added 2026-03-26)*

Project knowledge section 13 establishes where torchforge sits in the Rust ML stack. Reproduced here for orientation — the viz crate has no neural network backend dependency, but the training loops it observes will use `burn`.

- `burn` — Native Rust training framework. The RL training loops this crate monitors will be burn-backed.
- `candle` — Best for inference + LLMs. Not the training target.
- `torchforge` — **builds ON TOP of burn/candle** for the edge RL training use case.

**Implication for this crate**: No dependency on burn or candle is needed or wanted here. The `SummaryWriter` interface is intentionally framework-agnostic — it writes scalars, histograms, and images regardless of what computed them. This is the correct design. The ecosystem context confirms it: a visualizer that couples to a specific NN backend would be less useful across the stack, not more.

---

## Out of Scope (v0.x)

- Weights & Biases (wandb) integration — REST API wrappable in future, deferred
- Custom web dashboard — TensorBoard compatibility is sufficient for v1.0
- Audio summaries
- Text summaries
- Embedding projector
- Federation-aware logging (multi-device aggregation, gradient statistics, communication metrics) — *deferred to v1.x as torchforge-federated; per-device logging is fully supported in v0.x*
