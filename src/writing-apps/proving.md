
# Proving

## Overview

Pico provides CLI and SDK tools to recursively prove the program to the developers.

Pico CLI provides a complete toolchain for compiling the RISC-V program and using Pico to complete end-to-end proof. Refer to the [installation page](../getting-started/installation.md) to install the CLI toolchain. Pico 3.0 proves over the KoalaBear field.

Like the CLI, the `pico-prover` crate includes lower-level APIs that can prove the program directly. The [prover package](https://github.com/brevis-network/pico-zkapp-template/tree/main/prover) of the template project repository provides an example of how to import and initialize the SDK and quickly generate a RISC-V proof using the Pico SDK. In the [Proving Steps Section](proving.md#proving-steps), you can read more about VM e2e proving and the Gnark EVM proof generation for On-chain verification

Let's quickly go through the Pico SDK usage and generate a Fibonacci RISC-V proof.

1. Import `pico-sdk` and `pico-prover`

```toml
# Cargo.toml
pico-sdk = { git = "https://github.com/brevis-network/pico" }
pico-prover = { git = "https://github.com/brevis-network/pico" }
```

2. Execute the proving process and generate RISC-V proof.


```rust
// prover/src/main.rs
fn main() {
    // Initialize logger
    init_logger();

    // Load the ELF file
    let elf = load_elf("../app/elf/riscv64im-pico-zkvm-elf");

    // Initialize the prover
    let mut prover = Prover::new(&elf).expect("Failed to initialize the prover");
    // Initialize new stdin
    let mut stdin = Stdin::new();

    // Set up input and generate proof
    let n = 100u64;
    stdin.write(&n);

    // Generate proof
    let proof = prover.prove_to(stdin, Phase::Riscv).expect("Failed to generate proof");

    // Decodes public values from the proof's committed values.
    let public_values: FibonacciData = proof.committed().read().expect("Failed to decode public values");

    // Verify the public values
    verify_public_values(n, &public_values);
}

/// Verifies that the computed Fibonacci values match the public values.
fn verify_public_values(n: u64, public_values: &FibonacciData) {
    println!(
        "Public value n: {:?}, a: {:?}, b: {:?}",
        public_values.n, public_values.a, public_values.b
    );

    // Compute Fibonacci values locally
    let (result_a, result_b) = fibonacci(0, 1, n);

    // Assert that the computed values match the public values
    assert_eq!(result_a, public_values.a, "Mismatch in value 'a'");
    assert_eq!(result_b, public_values.b, "Mismatch in value 'b'");
}
```


## Pico EmulatorStdin

### Stdin Writer

1. **Pico SDK supports writing the serializable object and bytes to Pico.**

```rust
/// Write a serializable struct to the buffer.
pub fn write<T: Serialize>(&mut self, data: &T);
/// Write a slice of bytes to the buffer.
pub fn write_bytes(&mut self, bytes: &[u8]);
```

Examples:

```rust
use std::vec;
use pico_prover::Stdin;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct FibonacciInputs {
    pub a: u64,
    pub b: u64,
    pub n: u64,
}

fn main() {
    // Initialize new stdin
    let mut stdin = Stdin::new();
    // example 1: write a u64 to the VM
    let n = 100u64;
    stdin.write(&n);

    // example 2: write a struct
    let inputs = FibonacciInputs { a: 0, b: 1, n };
    stdin.write(&inputs);

    // example 3: write a byte array
    let bytes = vec![1, 2, 3, 4];
    stdin.write_bytes(&bytes);
}
```

2. **CLI input option**

The `prove` command <mark style="color:orange;">`--input`</mark> option can take a hex string or a file path. the hex string must be match the length of the read type. For example, the input `n = 100u64`; the <mark style="color:orange;">hex string should be</mark> `0x6400000000000000` <mark style="color:orange;">in little-endian format</mark>.

```sh
RUST_LOG=info cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --input "0x6400000000000000" --phase riscv
```

### Read

Corresponding to the writer functions, there are read\_as and read\_slice tools for io reading the serializable object or bytes into the program.

SDK examples:

<pre class="language-rust"><code class="lang-rust">
use pico_sdk::io::{read_as, read_vec};

<strong>#[derive(Serialize, Deserialize)]
</strong>pub struct FibonacciInputs {
    pub a: u64,
    pub b: u64,
    pub n: u64,
}

fn main() {
    // example 1: read the u64 input `n`
<strong>    let n: u64 = read_as();
</strong><strong>    
</strong>    // example 2: read FibonacciInputs struct  
    let inputs = read_as::&#x3C;FibonacciInputs>();
    
    // example 3: read a byte array
    let bytes: Vec&#x3C;u8> = read_vec();
}
</code></pre>

## End-to-end Proving

This section introduces more advanced CLI options and SDK APIs to complete the end-to-end proving process. The Proving process consists of multiple stages, including [RISCV, RECURSION, and EVM Phases](advanced/proverchain.md).

### RISCV-Phase

Prove RISC-V programs and generate an uncompressed proof with the `--phase riscv` option. The command is mainly used to test and debug the program.

CLI:

```sh
RUST_LOG=info cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --phase riscv
```

For example, when executing the fast proving with inputs in the Fibonacci, the input `n` is a `u64` data received through `pico::sdk::read_as`, and it must be in little-endian format and filled to 8 bytes.

```sh
RUST_LOG=info cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --input "0x6400000000000000" --phase riscv
```

SDK:

```rust
// Initialize the prover.
let mut prover = Prover::new(&elf).expect("Failed to initialize the prover");

// Initialize new stdin and write the inputs.
let mut stdin = Stdin::new();

// Set up input
let n = 100u64;
stdin.write(&n);

let riscv_proof = prover.prove_to(stdin, Phase::Riscv).expect("Failed to generate proof");
```

### RECURSION-Phase

CLI:

```sh
RUST_LOG=info cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --phase wrap
```

Proving with `--phase wrap` will execute the prover up to and including the EMBED-Phase and write the inputs of the `Gnark` proof verification circuit, which can then be verified on-chain via contract. Without `--phase`, `prove` stops after the COMBINE phase.

options:

`--output`

You can specify the output path to generate the files prepared for the `Gnark` verification and default is `pico_out` in the current directory

```sh
RUST_LOG=info cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --phase wrap --output outputs
```

SDK:

```rust
// Initialize the prover.
let mut prover = Prover::new(&elf).expect("Failed to initialize the prover");
// ... write to stdin as previously described
let proof = prover.prove_to(stdin, Phase::Wrap).expect("Failed to generate proof");
let output_dir = PathBuf::from_str(&"./outputs").expect("the output dir is invalid");

proof
    .write_wrap_artifacts(
        output_dir.join("wrap_constraints.json"),
        output_dir.join("wrap_groth16_witness.json"),
    )
    .expect("Failed to write the outputs");
```

Outputs

* `wrap_constraints.json`: The schema of the stark proof constraints is used to transform to Gnark circuit constraints.
* `wrap_groth16_witness.json`: input witnessness of Gnark circuit.

### EVM-Phase

The Pico CLI provides an EVM option to generate the program Groth16 proof and verifier Contracts. The evm option runs in whitelist mode only (`VK_VERIFICATION=true`, `MLE_VK_MAP`, `MLE_VK_ROOT`), with gnark from `$PICO_GNARK_IMAGE` (a [Docker](https://docs.docker.com/engine/install/ubuntu/) image built from `gnark/Dockerfile`) or `$PICO_GNARK_BIN`.

CLI:

```sh
# Setup groth16 PK/VK if its never generated or a new version update.
cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --input "0x6400000000000000" --evm --setup --keys keys
# generate groth16 proof
cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --input "0x6400000000000000" --evm --keys keys
```

SDK:

```rust
// Initialize the prover in whitelist mode, which EVM proofs require.
let mut prover = Prover::builder(&elf)
    .verify_keys(true)
    .vk_map("mle_vk_map_kb.bin")
    .trusted_vk_root(vk_root)
    .build()
    .expect("Failed to initialize the prover");
let output_dir = PathBuf::from_str(&"./outputs").expect("the output dir is invalid");
// Writes the inputs of the Gnark circuit; the Groth16 proof itself is made by `cargo pico prove --evm`.
let bundle = prover.prove_evm(stdin).expect("Failed to generate evm proof");
bundle.write_to(output_dir).expect("Failed to write the outputs");
```

The outputs:

`proof.data`: `Groth16` proof generated by the Gnark Verifier Circuit.

`inputs.json`: what the verifier contract is called with, including the public values.

With `--setup`, the `Gnark` `Groth16` ProvingKey/VerificationKey is also generated at this step, into the `--keys` directory. The `--setup` only needs to be executed once; its keys are for testing only, since whoever runs the setup knows its randomness.

### EVM Verification

The generated `inputs.json` format is as follows:

```
{
  "riscvVKey": "bytes32",
  "publicValues": "bytes",
  "committedValuesDigest": "bytes32",
  "proof": "uint256[12]"
}
```

After parsing the input data, you can call the `PicoVerifier.sol` as shown below:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/// @title Pico Verifier Interface
/// @author Brevis Network
/// @notice This contract is the interface for the Pico Verifier.
interface IPicoVerifier {
    /// @notice Verifies a proof with given public values and riscv verification key.
    /// @param riscvVkey The verification key for the RISC-V program.
    /// @param publicValues The public values encoded as bytes.
    /// @param proof The proof of the riscv program execution in the Pico.
    function verifyPicoProof(
        bytes32 riscvVkey,
        bytes calldata publicValues,
        uint256[12] calldata proof
    ) external view;
}
```

The `verifyPicoProof` function in `PicoVerifier.sol` takes a RISC-V verification key, public values, and a Pico proof, using the Groth16 verifier to validate the proof and public inputs via the pairing algorithm. For the full implementation of the PicoVerifier, please refer to the repository [here](https://github.com/brevis-network/pico-zkapp-template/blob/evm/contracts/src/PicoVerifier.sol).

In production, you need to verify riscvVKey and parse the public values verified by PicoVerifier. You can refer to the Fibonacci.sol example in the repository [here](https://github.com/brevis-network/pico-zkapp-template/blob/evm/contracts/src/Fibonacci.sol).
