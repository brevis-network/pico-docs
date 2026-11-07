
# Features

## Logging

Pico leverages Rust’s standard logging utilities to provide detailed runtime information, particularly about performance and program statistics. You can control the verbosity of the logs via the `RUST_LOG` environment variable:

* **Info Level:** Set `RUST_LOG=info` to output overall performance data and high-level progress information.
* **Debug Level:** Set `RUST_LOG=debug` to display detailed logs, including statistics of chunks and records as they are generated and processed.

For scenarios where you want to save logs to a file without color codes (which may be embedded by default), you can pipe the output through a tool like `ansi2txt`. This conversion ensures that the log file remains clean and free of terminal-specific formatting, as the tracing framework does not automatically adjust colors based on environment variables.

## Debugging

In the rare event that proving fails on a correctly executing binary, Pico provides additional debug capabilities to assist in pinpointing issues:

* **Enhanced Debugging Features:** Enable the `debug` and `debug-lookups` features when running the prover. These features provide extra context by outputting detailed information on individual constraints and lookup operations within each processing batch.
* **Minimal Memory Impact:** Since debug information is generated from data already in memory for the current batch of proofs, enabling debugging does not incur a significant additional memory cost. The debug data can be discarded once the batch is processed and debugged.
* **Accessing Debug Data:** Combine the debugging features with `RUST_LOG=debug` to capture detailed logs.

## Guest VM Cycle Tracking

Please see [VM Cycle Tracking](advanced/vm-cycle-tracking.md) for a detailed explanation of this feature.

## Proving Options

Pico offers several configurable parameters to optimize the proving process for your system’s resources and performance requirements. `cargo pico` reads them from environment variables; with `pico-prover`, set them on `ProverBuilder`:

* **`CHUNK_SIZE`** (`chunk_size`): the number of instructions executed per chunk. Defaults to 2^23.
* **`PROVE_CONCURRENCY`** (`concurrency`): the number of RISC-V chunks proved at once. Defaults to 4. Each one holds a full proving working set, so lower it on a machine with less memory.

These options allow you to balance performance and resource utilization, making it possible to optimize Pico for a wide range of environments—from resource-constrained setups to high-performance systems.

## Features

`pico-vm` comes enabled with several features by default. These being `strict`, `rayon`, and `multi-field-transcript`. `strict` is a compile time option for `#![deny(warnings)]` on the entire pico-vm module. `rayon` enables the usage of rayon's `ParallelIterator` and related traits to use multithreading to speed up the proving process. This feature should be left on unless you wish to compile a single threaded prover for profiling reasons, as `rayon` tends to pollute the stack trace when running `flamegraph`. `multi-field-transcript` enables the EMBED and WRAP phases, which the EVM path needs.

To build `pico-vm` without default features, simply set default-features = false in your Cargo.toml or run `cargo build -p pico-vm --no-default-features` for your local build environment, maybe adding `--example` if you want to build a specific example in addition to the Rust library.

## Single threaded profiling

As mentioned in the previous section, we support single threaded builds in order to generate neater flamegraphs for profiling purposes. For example, to build `test_mle_e2e` for profiling, run

```
cargo build -p pico-vm --no-default-features --profile profiling --example test_mle_e2e
```

to build the binary and then run

```
sudo flamegraph -o flamegraph.svg -- ./target/profiling/examples/test_mle_e2e --elf fib --n 10 --step riscv
```

with [`cargo flamegraph`](https://github.com/flamegraph-rs/flamegraph) to produce a flamegraph that you can use to explore cost centers.
