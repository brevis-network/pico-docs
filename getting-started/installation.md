---
description: Install Pico toolchains
---

# Installation

## Requirements:

* [Rust (Nightly)](https://www.rust-lang.org/tools/install)
* [Git ](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Docker (Optional, for EVM proof)](https://docs.docker.com/engine/install/)

## Install&#x20;

### Option 1:  Cargo install

1. Install pico-cli from the GitHub repository

```sh
cargo +nightly-2025-08-04 install --git https://github.com/brevis-network/pico pico-cli
```

2. Check the version

```sh
cargo pico --version
```

### Option 2: Local install

1. Git clone Pico-VM repository

```sh
git clone https://github.com/brevis-network/pico
```

2. cargo install from the local path

```sh
cd sdk/cli
cargo install --locked --force --path .
```

### Rust toolchain

Pico uses the rust-specific rust toolchain version(nightly-2025-08-04) to build the program. The specific toolchain version can be found in the `rust_toolchain` file in the repository root.

```sh
rustup install nightly-2025-08-04
```

```bash
rustup component add rust-src --toolchain nightly-2025-08-04
```

[\
](https://book.openvm.dev/introduction.html)
