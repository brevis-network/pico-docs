
# Quick start

This page shows you how to create and prove a Fibonacci program.

## Start with the Fibonacci template&#x20;

1. Create project&#x20;

```
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

3. Prove program with Pico

```sh
# Prove in prover folder
cd ../prover
RUST_LOG=info cargo run --release
```

The `prover` subdirectory contains a Rust program that will load an input for the ELF that was just compiled, execute it, and generate a proof. This project has the entire functionality of the Pico SDK at its disposal, and can be customized however you want.

If you simply wish to use the default provided proving clients and options, you can prove using the Pico CLI via

```sh
RUST_LOG=info cargo pico prove --input "0x0A000000" --fast --elf /path/to/elf # input n = 10
```

The input to the fibonacci program is a single u32 specifying which number to compute, so we can directly pass the input with the `--input` option, supplying little endian bytes. `--fast` simply tells the prover to skip any recursion steps and terminate after finishing the RISC-V proof.

## Project Layout

```
Fibonacci
|—— app
     |—— elf
          |—— riscv32im-pico-zkvm-elf
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

`app/elf`**:** contains ELF with RISC-V instructions.&#x20;

`lib`:  contains components or utilities shared in multiple modules.

`prover`: contains the scripts to prepare program input data and execute the proving process.

## Start with the EVM template&#x20;

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
    RUST_LOG=info cargo run --release
    ```

    This step will locally set up the Groth16 Verifier contract and generate the Pico Groth16 proof. The files will be outputted to the `root/contracts/test_data` folder.\
    The prover program will then attempt to launch a Docker container to generate the final EVM proof with `gnark`. This ingests `test_data/proof.json` and should produce `test_data/proof.data`. If this file is not produced, you may need to increase the amount of RAM available to the container.
3.  Test EVM Proof

    ```bash
    cd ../contracts
    mv -f ./test_data/Groth16Verifier.sol ./src/Groth16Verifier.sol
    forge test
    ```

    The foundry test script will parse the proof generated in the previous step and interact with the Groth16 Verifier contract. With all tests passing, the EVM quick start is successful.
