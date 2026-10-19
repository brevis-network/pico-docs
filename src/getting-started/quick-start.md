
# Quick start

This page shows you how to create and prove a Fibonacci program.

## Start with the Fibonacci template

1. Create project

```sh
cargo pico new --template basic Fibonacci
```

This creates a directory `Fibonacci` with the `basic` template, which contains a fibonacci program.

2. Build program

```sh
# Build program in app folder
cd app
cargo pico build    
```

This will use the Pico compiler to generate a RISC-V ELF that can be executed by the Pico ZKVM.

3. Prove program with Pico (Default Path)

```sh
# Prove in prover folder
cd ../prover
RUST_LOG=info cargo run --release -- prove fib.pico
```

The `prover` subdirectory contains a Rust program that will load an input for the ELF that was just compiled, execute it, and generate a proof. This project has the entire functionality of the Pico SDK at its disposal, and can be customized however you want.

If you simply wish to use the default provided proving clients and options, you can prove using the Pico CLI via

```sh
RUST_LOG=info cargo pico prove --input "0x6400000000000000" --phase riscv --elf /path/to/elf # input n = 100
```

The input to the fibonacci program is a single u64 specifying which number to compute, so we can directly pass the input with the `--input` option, supplying little endian bytes. `--fast` simply tells the prover to skip any recursion steps and terminate after finishing the RISC-V proof.

4. Prove program with AOT (Experimental)

AOT is an alternative execution path: the emulator thread can be driven by AOT-compiled chunks generated from the specific guest ELF you are proving, instead of the interpreter. This trades a one-time codegen step for reduced per-proof execution cost.

* Generate AOT chunks

The generated chunks are tied to the PC ranges of a specific ELF. You must re-generate whenever `app/src/` changes.

From the project root, after `cargo pico build`:

```sh
cargo pico aot --elf app/elf/riscv64im-pico-zkvm-elf --out aot-generated
```

`cargo pico aot` wires `pico-aot-runtime` as a git dep in the generated chunk crates, pinned to the CLI's release tag, so `aot-generated/` works at the template root without requiring a local Pico checkout. Set `PICO_SOURCE` to a Pico checkout to build against it instead.

* Prove with `--aot`

Enable the template's `aot` feature:

```sh
cd prover
RUST_LOG=info cargo run --release --features aot -- prove fib.pico
```

## Project Layout

```
Fibonacci
|—— app
     |—— elf
          |—— riscv64im-pico-zkvm-elf
     |—— src
          |—— main.rs
     |—— Cargo.toml
|—— lib
     |—— src
          |—— lib.rs
     |—— Cargo.toml
|—— prover
     |—— src
           |—— main.rs
     |—— Cargo.toml
|—— Cargo.toml
|—— Cargo.lock
|—— rust-toolchain     
```

The template project includes 3 workspace members: `app`, `lib` and `prover`

`app`: contains the program source code, which will be compiled to `RiscV`

`app/elf`**:** contains ELF with RISC-V instructions.

`lib`: contains components or utilities shared in multiple modules.

`prover`: contains the scripts to prepare program input data and execute the proving process.

## Start with the EVM template

**Minimum memory requirement**: 32GB

1.  Create and build the EVM Example Project

    ```bash
    cargo pico new evm-example --template evm
    cd evm-example/app/
    cargo pico build
    ```

    This uses the `evm` template, which will set up a proving script that will generate a Groth16 proof suitable for verification via smart contract on an ETH compatible chain.
2.  Prove program to EVM

    ```bash
    cd ../prover
    RUST_LOG=info cargo run --release -- prove fib.pico
    ```

    This proves the program in dev mode and writes `fib.pico`, a RISC-V proof that verifies from the ELF but is not one a contract can check. The EVM proof comes from `cargo pico prove --evm` in whitelist mode, with a gnark prover and a Groth16 key set; the template's README, "The on-chain proof", lists the exact commands.
3.  Test EVM Proof

    ```bash
    cd ../contracts
    forge test
    ```

    The Foundry tests verify the proof in `test_data/inputs.json` with the Groth16 verifier in `test/dev`. To test your own EVM proof, copy its `inputs.json` and key files in, as the template's README shows. With all tests passing, the EVM quick start is successful.
