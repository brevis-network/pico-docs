---
description: Proving with Pico CLI and SDK APIs
---

# Proving

## Overview&#x20;

Pico provides CLI and SDK tools to recursively prove the program to the developers. &#x20;

Pico CLI provides a complete toolchain for compiling the RISC-V program and using Pico to complete end-to-end proof. Refer to the [installation page](../getting-started/installation.md) to install the CLI toolchain. CLI default use the KoalaBear field for the backend proving, if you want to switch to other fields, read more details in [Proving Backends Page](advanced/proving-backends.md).&#x20;

Like the CLI, the Pico-SDK includes lower-level APIs that can prove the program directly. The [prover package](https://github.com/brevis-network/pico-zkapp-template/tree/main/prover) of the template project repository provides an example of how to import and initialize the SDK and quickly generate a RISC-V proof using the Pico SDK. In the [Proving Steps Section](proving.md#proving-steps), you can read more about VM e2e proving and the Gnark EVM proof generation for On-chain verification&#x20;

Let's quickly go through the Pico SDK usage and generate a Fibonacci RISC-V proof.

1. Import `pico-sdk`

```toml
# Cargo.toml
pico-sdk = { git = "https://github.com/brevis-network/pico" }
```

2. Execute the proving process and generate RISC-V proof.

{% code lineNumbers="true" %}
```rust
// prover/src/main.rs
fn main() {
    // Initialize logger
    init_logger();

    // Load the ELF file
    let elf = load_elf("../elf/riscv32im-pico-zkvm-elf");

    // Initialize the prover client
    let client = DefaultProverClient::new(&elf);
    // Initialize new stdin
    let mut stdin_builder = client.new_stdin_builder();

    // Set up input and generate proof
    let n = 100u32;
    stdin_builder.write(&n);

    // Generate proof
    let proof = client.prove_fast(stdin_builder).expect("Failed to generate proof");

    // Decodes public values from the proof's public value stream.
    let public_buffer = proof.pv_stream.unwrap();
    let public_values = PublicValuesStruct::abi_decode(&public_buffer, true).unwrap();

    // Verify the public values
    verify_public_values(n, &public_values);
}

/// Verifies that the computed Fibonacci values match the public values.
fn verify_public_values(n: u32, public_values: &PublicValuesStruct) {
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
pub fn write_slice(&mut self, slice: &[u8]);
```

Examples:

<pre class="language-rust"><code class="lang-rust">use std::vec;
use pico_sdk::client::SDKProverClient;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct FibonacciInputs {
    pub a: u32,
    pub b: u32,
    pub n: u32,
}

fn main() {
    // Initialize the prover client
    let client = SDKProverClient::new(&#x26;elf, false);
    // Initialize new stdin
    let mut stdin_builder = client.new_stdin_builder();
    // example 1: write a u32 to the VM
    let n = 100u32;
    stdin_builder.write(&#x26;n);
    
<strong>    // example 2: write a struct 
</strong>    let inputs = FibonacciInputs { a: 0, b: 1, n };
    stdin_builder.write(&#x26;inputs);
    
    // example 3: write a byte array
    let bytes = vec![1, 2, 3, 4];
    stdin_builder.write_slice(&#x26;bytes);
}
</code></pre>

2. **CLI input option**

The `prove` command <mark style="color:orange;">`--input`</mark> option can take a hex string or a file path. the hex string must be match the length of the read type. For example, the input `n = 100u32`; the <mark style="color:orange;">hex string should be</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`0x0A000000`</mark> <mark style="color:orange;"></mark><mark style="color:orange;">in little-endian format</mark>.

```sh
RUST_LOG=info cargo pico prove --input "0x0A000000" --fast
```

### Read

Corresponding to the writer functions, there are read\_as and read\_slice tools for io reading the serializable object or bytes into the program.

SDK examples:

<pre class="language-rust"><code class="lang-rust">
use pico_sdk::io::{read_as, read_vec};

<strong>#[derive(Serialize, Deserialize)]
</strong>pub struct FibonacciInputs {
    pub a: u32,
    pub b: u32,
    pub n: u32,
}

fn main() {
    // example 1: read the u32 input `n`
<strong>    let n: u32 = read_as();
</strong><strong>    
</strong>    // example 2: read FibonacciInputs struct  
    let inputs = read_as::&#x3C;FibonacciInputs>();
    
    // example 3: read a byte array
    let bytes: Vec&#x3C;u8> = read_vec();
}
</code></pre>

## End-to-end Proving

This section introduces more advanced CLI options and SDK APIs to complete the end-to-end proving process. The Proving process consists of multiple stages, including [RISCV, RECURSION, and  EVM Phases](advanced/proverchain.md#provechain-proving-phase). Pico SDK includes various ProverClients in different proving backends. Here, we use the KoalaBearProverClient (based on STARK on KoalaBear) in the example code.&#x20;

### RISCV-Phase

Prove RISC-V programs and generate an uncompressed proof with the <mark style="color:orange;">--fast</mark> option. The command is mainly used to test and debug the program.

CLI:

```sh
RUST_LOG cargo pico prove --fast 
```

For example, when executing the fast proving with inputs in the Fibonacci, the input `n` is a `u32` data received through `pico::sdk::read_as`, and it must be in little-endian format and filled to 4 bytes.

```sh
RUST_LOG=info cargo pico prove --input "0x0A000000" --fast
```

SDK:

```rust
// Initialize the SDK.
let client = DefaultProverClient::new(&elf);

// Initialize new stdin and write the inputs by builder.
let mut stdin_builder = client.new_stdin_builder();
 
// Set up input
let n = 100u32;
stdin_builder.write(&n);

let riscv_proof = client.prove_fast(stdin_builder).expect("Failed to generate proof");
```

Fast proving is implemented by using only one FRI query which drastically reduces the theoretical security bits. **DO NOT USE THIS OPTION IN PRODUCTION. ATTACKERS MAY BE ABLE TO COMMIT TO INVALID TRACES**.

### RECURSION-Phase

CLI:

```sh
RUST_LOG cargo pico prove --field kb # kb: koalabear (default), bb:babebear 
```

Proving without the `--fast` argument will execute the prover up to and including the EMBED-Phase. The resulting proof can then be verified by the `Gnark` proof verification circuit, which can then be verified on-chain via contract.

options:

`--field`

Specify the field, When without this option, default to Koalabear field.

* kb: Koalabear
* bb: Babybear

`--output`

You can specify the output path to generate the files prepared for the `Gnark` verification and default is in the project root  `target/pico_out/`

```sh
RUST_LOG cargo pico prove --output outputs
```

SDK:

```rust
// Initialize the SDK.
let client = DefaultProverClient::new(&elf);
// ... write to stdin as previously described
let (riscv_proof, embed_proof) = client.prove(stdin_builder)?;
let output_dir = PathBuf::from_str(&"./outputs").expect("the output dir is invalid");

client.write_onchain_data(output, &riscv_proof, &embed_proof)?;
```

Outputs

* `constraints.json`: The schema of the stark proof constraints is used to transform to Gnark circuit constraints.
* `groth16_witness.json`: input witnessness of Gnark circuit.

### EVM-Phase

The Pico CLI provides an EVM option to generate the program Groth16 proof and verifier Contracts. You must ensure the [Docker](https://docs.docker.com/engine/install/ubuntu/) has been installed when using the evm option.

CLI:

```sh
# Setup groth16 PK/VK if its never generated or a new version update.
cargo pico prove --evm --setup 
# generate groth16 proof 
cargo pico prove --evm 
```

SDK:

<pre class="language-rust"><code class="lang-rust">// Initialize the SDK.
let client = KoalaBearProveVKClient::new(&#x26;elf);
let output_dir = PathBuf::from_str(&#x26;"./outputs").expect("the output dir is invalid");
// The second argument need_setup should be true when you haven't setup groth16 pk/vk yet.
<strong>// The last argument selects the proving backend: use "kb" for KoalaBear or "bb" for BabyBear.
</strong>prover_client.prove_evm(stdin_builder, true, output_path, "kb").expect("Failed to generate evm proof");
</code></pre>

The outputs:

`proof.data`:  `Groth16` proof generated by the Gnark Verifier Circuit.

`pv_file`: The public values hex string; it's the input of `Fibonacci` Contract

When executing EVM proving, the `Gnark` `Groth16` ProvingKey/VerificationKey is also generated at this step. The `--setup` only needs to be executed once to make sure the PK/VK is generated.

### EVM Verification

The generated `inputs.json` format is as follows:

```
{
  "riscvVKey": "bytes32",
  "proof": "bytes32[]",
  "publicValues": "bytes"
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
        uint256[8] calldata proof
    ) external view;
}
```

The `verifyPicoProof` function in `PicoVerifier.sol` takes a RISC-V verification key, public values, and a Pico proof, using the Groth16 verifier to validate the proof and public inputs via the pairing algorithm. For the full implementation of the PicoVerifier, please refer to the repository [here](https://github.com/brevis-network/pico-zkapp-template/blob/evm/contracts/src/PicoVerifier.sol).

In production, you need to verify riscvVKey and parse the public values verified by PicoVerifier. You can refer to the Fibonacci.sol example in the repository [here](https://github.com/brevis-network/pico-zkapp-template/blob/evm/contracts/src/Fibonacci.sol).

