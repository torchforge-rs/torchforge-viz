# torchforge-viz

> Lightweight, well-maintained training visualizer with TensorBoard-compatible output for Rust ML pipelines.

Part of the [torchforge-rs](https://github.com/torchforge-rs) ecosystem.

[![Crates.io](https://img.shields.io/crates/v/torchforge-viz.svg)](https://crates.io/crates/torchforge-viz)
[![Docs.rs](https://docs.rs/torchforge-viz/badge.svg)](https://docs.rs/torchforge-viz)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Rust](https://img.shields.io/badge/rust-2024%20edition-orange.svg)](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0/)
[![CI](https://github.com/torchforge-rs/torchforge-viz/actions/workflows/ci.yml/badge.svg)](https://github.com/torchforge-rs/torchforge-viz/actions/workflows/ci.yml)

## Prerequisites

- **Rust 1.85+** — Install from [rustup.rs](https://rustup.rs/)
- **protoc** — Protocol Buffers compiler (required for building)
  ```bash
  # Ubuntu/Debian
  sudo apt install protobuf-compiler
  
  # macOS
  brew install protobuf
  
  # Or download from https://github.com/protocolbuffers/protobuf/releases
  ```

---

## Why

Training a model without visibility is flying blind. For Rust ML pipelines, the options today are:

- `tensorboard-rs` — exists, but has ~164 downloads/month and is effectively unmaintained
- Route metrics through Python — defeats the purpose of a Rust pipeline
- Roll your own — repeated work, no standard

TensorBoard itself ships a Rust backend ([RustBoard](https://github.com/tensorflow/tensorboard/blob/master/docs/design/rustboard.md), since TensorBoard 2.5.0) because the Python data loading path was too slow. The wire format is stable and well-understood. What's missing is a **well-maintained, first-class Rust writer** for that format.

`torchforge-viz` is that writer — plus a path toward on-device visualization for edge targets where no display server exists.

---

## Design Principles

1. **TensorBoard-compatible by default** — writes standard event files readable by existing tooling; no new standard required
2. **No Python runtime** — pure Rust writer, zero Python dependency in the training process
3. **RL-native metrics** — episode return, policy entropy, value loss, replay buffer statistics are first-class, not bolted on
4. **Edge-aware** — optional terminal UI (TUI) mode for devices without a display server
5. **Framework-agnostic** — `SummaryWriter` is independent of neural network backend; works with `burn`, `candle`, or anything else
6. **Honest about unknowns** — two hard prerequisites (schema verification, license review) are documented and must be resolved before v0.1.0 implementation begins

---

## Status

**`v0.0.1` — Pre-alpha. No stable API. Two hard prerequisites block v0.1.0 implementation.**

The repository structure, CI, governance documents, and OSS foundation are complete. Before any implementation begins:

1. **`[LEGAL]`** — Licensing of deriving protobuf definitions from TensorFlow's Apache-2.0 `.proto` files must be confirmed
2. **`[RESEARCH]`** — TensorBoard protobuf schema compatibility with versions >= 2.15 must be verified

Neither of these is expected to be blocking long-term, but neither will be assumed away. See [ARCHITECTURE.md](ARCHITECTURE.md) for full detail.

---

## Roadmap

| Version | Goal |
|---|---|
| **Pre-v0.1.0** | Schema verification, license review (hard blockers) |
| **v0.1.0** | Scalar writing — correct TensorBoard event files readable by `tensorboard` |
| **v0.2.0** | Histograms, images, `add_scalars` |
| **v0.3.0** | `RLLogger` — typed metric helpers for RL training loops, FDRL-aware tag namespace |
| **v0.4.0** | Terminal UI (TUI) mode for edge devices without a display server |
| **v0.5.0** | `MultiWriter`, tag filtering, step-based decimation |
| **v1.0.0** | Stable API, verified on TensorBoard >= 2.15, TUI on edge hardware |

---

## Planned API

> **⚠️ Illustrative only — will change before v0.1.0 is released.**

```rust
use torchforge_viz::SummaryWriter;

let mut writer = SummaryWriter::new("runs/experiment_01")?;

for step in 0..500_000_u64 {
    writer.add_scalar("train/episode_return", episode_return, step)?;
    writer.add_scalar("train/policy_loss", policy_loss, step)?;
    writer.add_scalar("train/value_loss", value_loss, step)?;
    writer.add_histogram("train/action_dist", &action_probs, step)?;
}

writer.flush()?;
// Open tensorboard --logdir runs/ to visualize
```

```rust
// v0.3.0+ — typed RL metrics
use torchforge_viz::RLLogger;

let mut logger = RLLogger::new("runs/dqn_cartpole")?;
logger.log_episode(return_, length, step)?;
logger.log_policy(policy_loss, entropy, kl_div, step)?;
logger.log_buffer(buffer_size, capacity, samples_per_sec, step)?;
```

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide including:

- Prerequisites (including `protoc` installation)
- Branching model and PR process  
- Code style and testing requirements
- Tier 1 / Tier 2 testing distinction

The most valuable contributions right now are:

- Verifying TensorBoard protobuf schema compatibility (>= v2.15) — this unblocks v0.1.0
- Confirming the Apache-2.0 licensing position on derived `.proto` definitions — this unblocks v0.1.0
- Testing on edge targets without a display server
- Identifying RL-specific metrics not covered by the current `RLLogger` design

**Open an issue before submitting a PR.**

Please read our [Code of Conduct](CODE_OF_CONDUCT.md) before participating.
To report a security issue, see [SECURITY.md](SECURITY.md).

---

## License

Apache-2.0. See [LICENSE](LICENSE).

Part of the [torchforge-rs](https://github.com/torchforge-rs) ecosystem — also see [torchforge-data](https://github.com/torchforge-rs/torchforge-data) and [torchforge-bench](https://github.com/torchforge-rs/torchforge-bench).
