# torchforge-viz

> Lightweight, well-maintained training visualizer with TensorBoard-compatible output for Rust ML pipelines.

Part of the [torchforge-rs](https://github.com/torchforge-rs) ecosystem.

[![Crates.io](https://img.shields.io/crates/v/torchforge-viz.svg)](https://crates.io/crates/torchforge-viz)
[![Docs.rs](https://docs.rs/torchforge-viz/badge.svg)](https://docs.rs/torchforge-viz)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Rust](https://img.shields.io/badge/rust-2024%20edition-orange.svg)](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0/)

---

## Why

Training a model without visibility is flying blind. For Rust ML pipelines, the options today are:

- `tensorboard-rs` — exists, but has ~164 downloads/month and is effectively unmaintained
- Route through Python — defeats the purpose of a Rust pipeline
- Roll your own — repeated work, no standard

TensorBoard itself ships a Rust backend (RustBoard, since TensorBoard 2.5.0) because the Python data loading path was too slow. The wire format is stable and well-understood. What's missing is a **well-maintained, first-class Rust writer** for that format.

`torchforge-viz` is that writer — plus a path toward on-device visualization for edge targets where no display server exists.

---

## Design Principles

1. **TensorBoard-compatible by default** — writes standard event files; no new tooling required to visualize
2. **No Python runtime required** — pure Rust writer, zero Python dependency
3. **RL-native metrics** — episode return, policy entropy, value loss, replay buffer statistics are first-class, not bolted on
4. **Edge-aware** — optional terminal UI (TUI) mode for devices without a display server
5. **Honest about unknowns** — where the design is unresolved, we say so explicitly

---

## Status

`v0.0.x` — **Pre-alpha. No stable API. Active design phase.**

See [ARCHITECTURE.md](ARCHITECTURE.md) for current design decisions and open questions.
See [TODO.md](TODO.md) for the implementation roadmap.

---

## Planned Usage

```rust
use torchforge_viz::SummaryWriter;

let mut writer = SummaryWriter::new("runs/experiment_01")?;

for step in 0..10_000 {
    // Scalars
    writer.add_scalar("train/episode_return", episode_return, step)?;
    writer.add_scalar("train/policy_loss", policy_loss, step)?;
    writer.add_scalar("train/value_loss", value_loss, step)?;

    // Histogram of policy action distribution
    writer.add_histogram("train/action_dist", &action_probs, step)?;
}

writer.flush()?;
```

> **Note:** This API is illustrative. It will change. Do not depend on it.

---

## Contributing

The most valuable contributions right now are:

- Verifying the current TensorBoard protobuf schema version compatibility
- Testing on edge targets without a display server
- Identifying RL-specific metrics not covered by the current design

Open an issue before submitting a PR.

---

## License

Apache-2.0. See [LICENSE](LICENSE).
