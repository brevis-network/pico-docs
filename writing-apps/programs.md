---
description: Writing app programs with Pico
---

# Programs

## Pico entrypoint

The program is executed and proved with the zkVM platform. Pico links the programs by the main function annotated with the `entrypoint` macro\_rules. The program needs to be declared with `no_main`.&#x20;

```rust
// declare no_main
#![no_main]

// point out the main function in your program.
pico_sdk::entrypoint!(main);

pub fn main() {
    //todo: write your program logic here
}

```

Raw system calls are available by using `pico_sdk::riscv_ecalls::*` but it is recommended to use the integrated patch libraries to avoid disrupting the standard development workflow. The program can then be compiled to RiscV without creating conditional compilation-related hoops aside from the entry point (unless your system assumes a word size of 64 bits, which is untrue in the zkVM). A few light wrappers for elliptic curve types can also be found in the `pico-patch-libs` crate.

Be very careful with using heap memory. The currently implemented allocator does not free any memory, so cloning a medium-sized `Vec` a few too many times will cause your program to go OOM. You must write your own allocator if a more managed memory solution is required.

## Inputs and outputs

`pico_sdk::io::read_as` Read a serialized input data type into the program and deserialize it into specific types, such as u32, u64, and bytes. Pico `prove` CLI provides two ways to pass the inputs: hex string and file path. When your specific input is a file path , the file content will be read as bytes in the program. &#x20;

```sh
cargo pico prove --input "" # hex or file path
```

`pico_sdk::io::commit` Commit serializable data to the public stream. The public inputs will be compressed using a SHA256 hash and exported to the on-chain.&#x20;

`pico_sdk::io:commit_bytes` Write the public values in a byte buffer.

## Setup programs

1. Create an instant program

```sh
cargo pico new --template basic basic-example
```

The project only contains a program module.  You can test and debug your RISC-V program quickly using the basic template.

2. Create a program with EVM

```sh
cargo pico new --template evm evm-example
```

The created project with the evm template will contain an extra `Contracts` folder for app and verification contracts .

* Proving with the evm option referring to [this page](proving.md) can generate the proof for on-chain EVM.
* Verification test requires to the install [foundryup and forge test](https://book.getfoundry.sh/getting-started/installation).&#x20;

Use the pre-prepared pico EVM proof and Groth16 verifier in the repo to run contract tests.

```sh
cd contract 
forge test
```



