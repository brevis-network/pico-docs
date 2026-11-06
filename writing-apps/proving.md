---
description: Proving with Pico CLI and SDK APIs
---

# Proving

## Overview

Pico provides CLI and SDK tools to recursively prove the program to the developers.

Pico CLI provides a complete toolchain for compiling the RISC-V program and using Pico to complete end-to-end proof. Refer to the [installation page](../getting-started/installation.md) to install the CLI toolchain. Pico 3.1 proves over the KoalaBear field, with 128 bits of provable soundness.

Like the CLI, the `pico-prover` crate includes lower-level APIs that can prove the program directly. The [prover package](https://github.com/brevis-network/pico-zkapp-template/tree/main/prover) of the template project repository provides an example of how to import and initialize the SDK and quickly generate a RISC-V proof using the Pico SDK. In the [Proving Steps Section](proving.md#proving-steps), you can read more about VM e2e proving and the Gnark EVM proof generation for On-chain verification

Let's quickly go through the Pico SDK usage and generate a Fibonacci RISC-V proof.

1. Import `pico-sdk` and `pico-prover`

```toml
# Cargo.toml
pico-sdk = { git = "https://github.com/brevis-network/pico", tag = "v3.1.1" }
pico-prover = { git = "https://github.com/brevis-network/pico", tag = "v3.1.1" }
```

2. Execute the proving process and generate RISC-V proof.

{% code lineNumbers="true" %}
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
{% endcode %}

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

Corresponding to the writer functions, there are `read_as` and `read_vec` tools for io reading the serializable object or bytes into the program.

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

For example, the input `n` in the Fibonacci is a `u64` read through `pico_sdk::io::read_as`, so it must be in little-endian format and filled to 8 bytes.

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

#### Dev mode and whitelist mode

Pico proves in one of two modes:

* **Dev mode**, the default: each recursion step uses a circuit fitted to the run. Its COMBINE proof verifies only in the process that made it, and no contract accepts it.
* **Whitelist mode**: every recursion step uses one of a fixed set of circuits, prepared ahead of time and shipped with Pico. Their verifying keys form the whitelist, identified by its root. A COMBINE proof then verifies in any process with the same whitelist, and the on-chain verifier has the root built in, so EVM proofs need this mode.

The two modes give the same program different program ids; a contract pins the whitelist-mode one. To prove in whitelist mode, set `VK_VERIFICATION=true`, `MLE_VK_MAP` and `MLE_VK_ROOT` for the CLI (see EVM-Phase), or build the SDK prover with `shipped_whitelist()`.



CLI:

```sh
RUST_LOG=info cargo pico prove --elf elf/riscv64im-pico-zkvm-elf --phase wrap
```

Proving with `--phase wrap` will execute the prover up to and including the EMBED-Phase and write the inputs of the `Gnark` proof verification circuit. A contract accepts them only from whitelist mode. Without `--phase`, `prove` stops after the COMBINE phase.

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

The Pico CLI provides an EVM option to generate the program Groth16 proof and verifier contracts. It proves in whitelist mode only and runs gnark in a [Docker](https://docs.docker.com/engine/install/ubuntu/) image. From the root of a project made with the `evm` template:

CLI:

```sh
# Once: build the gnark image. The tag is your CLI's version.
docker build -t pico-gnark -f gnark/Dockerfile https://github.com/brevis-network/pico.git#v3.1.1

# Whitelist mode, with the whitelist in the template's whitelist/.
export VK_VERIFICATION=true
export MLE_VK_MAP=whitelist/mle_vk_map_kb.bin
export MLE_VK_ROOT="$(sed -n 's/^MLE_VK_ROOT=//p' whitelist/mle_vk_root.txt | tr -d '"')"
export PICO_GNARK_IMAGE=pico-gnark

# Set up the Groth16 PK/VK the first time, or after a version update.
cargo pico prove --elf app/elf/riscv64im-pico-zkvm-elf --input u32:10 --evm --setup --keys keys
# Generate the Groth16 proof with the existing keys.
cargo pico prove --elf app/elf/riscv64im-pico-zkvm-elf --input u32:10 --evm --keys keys
```

The [EVM template](https://github.com/brevis-network/pico-zkapp-template/tree/evm#the-on-chain-proof) runs the full flow, including the contract tests, and notes the machine it needs.

SDK:

```rust
// Initialize the prover in whitelist mode, which EVM proofs require.
let mut prover = Prover::builder(&elf)
    .shipped_whitelist()
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
