# Contributing to torchforge-viz

Thank you for your interest in contributing to torchforge-viz! This document provides guidelines and information for contributors.

## Prerequisites

### Required Tools

- **Rust 1.85+**: This project requires Rust 1.85 or newer
- **protoc**: Protocol Buffers compiler (required for `prost-build`)
  - Install via package manager (e.g., `apt install protobuf-compiler` on Ubuntu)
  - Or download from [Protocol Buffers releases](https://github.com/protocolbuffers/protobuf/releases)
- **Python + uv**: Only required for Tier 2 compatibility testing
  - Install [uv](https://docs.astral.sh/uv/getting-started/installation/)
  - Used to run Python TensorBoard compatibility scripts

### Development Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/torchforge-rs/torchforge-viz.git
   cd torchforge-viz
   ```

2. Install Rust toolchain (automatically handled by `rust-toolchain.toml`):
   ```bash
   rustup update
   ```

3. Build the project:
   ```bash
   cargo build
   ```

4. Run tests:
   ```bash
   cargo test
   ```

## Building Locally

The project uses a `build.rs` script to compile Protocol Buffer definitions. The build process:

1. Compiles `.proto` files in the `proto/` directory using `prost-build`
2. Generates Rust code in `target/build/...`
3. Compiles the main crate

To build with debug information:
```bash
cargo build
```

To build optimized release:
```bash
cargo build --release
```

## Branching Model and PR Process

### Branch Organization

- `main`: Stable development branch, always passing CI
- `feature/*`: Feature branches for new functionality
- `fix/*`: Bug fix branches
- `docs/*`: Documentation updates

### Pull Request Process

1. Fork the repository
2. Create a feature branch from `main`
3. Make your changes
4. Ensure all tests pass and CI is green
5. Submit a pull request to `main`

### PR Requirements

- All CI checks must pass
- Documentation must be updated for any public API changes
- A CHANGELOG entry is required for behavior changes
- At least one review from a code owner is required

## Commit Message Format

We recommend using [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Build process or auxiliary tool changes

Examples:
```
feat(writer): add histogram support
fix(parser): handle malformed event files
docs(api): update scalar documentation
```

## Code Style

### Formatting

Use `cargo fmt` to format code:
```bash
cargo fmt --check  # Check formatting
cargo fmt          # Apply formatting
```

### Linting

Use `cargo clippy` for linting:
```bash
cargo clippy -- -D warnings  # Fail on warnings
```

All code must pass clippy checks without warnings.

### Documentation

- All public items must have documentation comments
- Use `///` for item documentation
- Include examples for complex APIs
- Document error conditions and panics

## Testing

### Tier 1 Testing (Required)

Runs on every PR:
- Unit tests (`cargo test`)
- Integration tests
- Documentation tests (`cargo doc --no-deps`)

### Tier 2 Testing (Manual/Release)

Python TensorBoard compatibility testing:
- Located in `tests/compat/`
- Requires Python and `uv`
- Runs manually or before releases
- Verifies TensorBoard can read generated event files

To run Tier 2 tests:
```bash
cd tests/compat
uv sync
uv run python test_tensorboard_compatibility.py
```

## What "Ready to Merge" Means

A PR is ready to merge when:

- [ ] All CI checks pass (formatting, clippy, tests, docs)
- [ ] Documentation is updated for all public API changes
- [ ] CHANGELOG entry is added for behavior changes
- [ ] At least one code owner has approved
- [ ] Tier 2 compatibility test run if event format changed
- [ ] PR description clearly explains the change
- [ ] Linked issue is referenced (if applicable)

## Issue Templates

### Bug Reports

When filing bug reports, please include:
- Rust version (`rustc --version`)
- Operating system
- TensorBoard version (if applicable)
- Minimal reproduction steps
- Expected vs actual behavior
- Any error messages or logs

### Feature Requests

Feature requests should include:
- Problem statement
- Proposed API sketch
- Alternatives considered
- Use cases and motivation

### Design Questions

For pre-implementation architecture discussion:
- Problem description
- Current limitations
- Proposed approach
- Trade-offs and alternatives

## License Headers

All source files should include the Apache-2.0 license header:

```rust
// Copyright 2025 torchforge-rs
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
```

## Getting Help

- Open an issue for bug reports or feature requests
- Use design questions for architectural discussions
- Join our discussions for general questions
- Check the [ARCHITECTURE.md](ARCHITECTURE.md) for design decisions

Thank you for contributing to torchforge-viz!
