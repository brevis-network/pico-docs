
# Function-level

Function-level coprocessors—commonly known as precompiles—are specialized circuits within Pico designed to optimize and streamline specific cryptographic operations and computational tasks. These precompiles handle operations such as elliptic curve arithmetic, hash functions, and signature verifications. In a general-purpose environment, these operations can be resource-intensive, but by offloading them to dedicated circuits, Pico significantly reduces computational costs, improves performance, and enhances scalability during proof generation and verification. Packaging these core operations into efficient, well-tested modules not only accelerates development cycles but also establishes a secure foundation for a wide range of zk-applications, including privacy-preserving transactions, rollups, and layer-2 scaling solutions.

## Work Flow

Below is an example workflow of Keccak256 hash permutation precompile in Pico.

<img src="../.gitbook/assets/file.excalidraw.svg" alt="" class="gitbook-drawing">

The Pico precompiles workflow involves several steps to efficiently execute and verify cryptographic operations. To illstrate how it works, we use Keccak-256 precompile as an example:

1. **Developer Preparation**: Developers begin by writing and preparing the necessary code, including the [tiny-keccak patch](https://github.com/brevis-network/tiny-keccak/tree/patch-v1.0.0) for cryptographic hashing functions. This library provides the core primitives needed for SHA2, SHA3, and Keccak-based operations.
2. **Tiny-Keccak Patch**: Pico uses a forked and zero-knowledge-compatible version of tiny-keccak (sourced from the public [debris repository](https://github.com/debris/tiny-keccak)). This patch optimizes hashing operations—particularly Keccak-256—to run efficiently within Pico.
3. **Keccak256 Precompile**: When a Keccak-256 hashing function is invoked, Pico’s Keccak256 precompile is triggered to handle the specific permutation operations. This specialized circuit, known internally as the `keccak256_permute_syscall`, is optimized for performance, minimizing overhead and improving provability.
4. **Rust Toolchain & ELF Generation**: The Rust toolchain compiles your code, including the tiny-keccak patch, into an Executable and Linkable Format (ELF) file, which is the RISC0's support for zkVM executables.

By following this workflow, developers can perform cryptographic operations more efficiently and securely, taking full advantage of Pico’s precompile features to reduce proof overhead and streamline the development of ZK apps.

## List of Syscalls

Pico is currently supporting [these syscalls](https://github.com/brevis-network/pico/blob/main/sdk/patch-libs/src/lib.rs).

## List of patches

Pico is currently supporting the following patches

<table data-full-width="true"><thead><tr><th width="208">Patch Name</th><th width="400">Github link</th><th>branch</th></tr></thead><tbody><tr><td>tiny-keccak</td><td>https://github.com/brevis-network/tiny-keccak</td><td><a href="https://github.com/brevis-network/tiny-keccak/releases/tag/pico-patch-v1.0.0-keccak-v2.0.2">pico-patch-v1.0.0-keccak-v2.0.2</a></td></tr><tr><td>sha2</td><td>https://github.com/brevis-network/hashes</td><td><a href="https://github.com/brevis-network/hashes/releases/tag/pico-patch-v1.0.1-sha2-v0.10.8">pico-patch-v1.0.1-sha2-v0.10.8</a></td></tr><tr><td>sha3</td><td>https://github.com/brevis-network/hashes</td><td><a href="https://github.com/brevis-network/hashes/releases/tag/pico-patch-v1.0.1-sha3-v0.10.8">pico-patch-v1.0.1-sha3-v0.10.8</a></td></tr><tr><td>curve25519-dalek</td><td>https://github.com/brevis-network/curve25519-dalek</td><td><a href="https://github.com/brevis-network/curve25519-dalek/releases/tag/pico-patch-v1.0.1-curve25519-dalek-v4.1.3">pico-patch-v1.0.1-curve25519-dalek-v4.1.3</a></td></tr><tr><td>bls12381</td><td>https://github.com/brevis-network/bls12_381</td><td><a href="https://github.com/brevis-network/bls12_381/releases/tag/pico-patch-v1.0.1-bls12_381-v0.8.0">pico-patch-v1.0.1-bls12_381-v0.8.0</a></td></tr><tr><td>curve25519-dalek-ng</td><td>https://github.com/brevis-network/curve25519-dalek-ng</td><td><a href="https://github.com/brevis-network/curve25519-dalek-ng/releases/tag/pico-patch-v1.0.1-curve25519-dalek-ng-v4.1.1">pico-patch-v1.0.1-curve25519-dalek-ng-v4.1.1</a></td></tr><tr><td>ed25519-consensus</td><td>https://github.com/brevis-network/ed25519-consensus</td><td><a href="https://github.com/brevis-network/ed25519-consensus/releases/tag/pico-patch-v1.0.1-ed25519-consensus-v2.1.0">pico-patch-v1.0.1-ed25519-consensus-v2.1.0</a></td></tr><tr><td>ecdsa-core</td><td>https://github.com/brevis-network/signatures</td><td><a href="https://github.com/brevis-network/signatures/releases/tag/pico-patch-v1.0.1-ecdsa-0.16.9">pico-patch-v1.0.1-ecdsa-0.16.9</a></td></tr><tr><td>secp256k1</td><td>https://github.com/brevis-network/rust-secp256k1</td><td><a href="https://github.com/brevis-network/rust-secp256k1/releases/tag/pico-patch-v1.0.1-secp256k1-v0.29.1">pico-patch-v1.0.1-secp256k1-v0.29.1</a></td></tr><tr><td>substate-bn</td><td>https://github.com/brevis-network/bn</td><td><a href="https://github.com/brevis-network/bn/releases/tag/pico-patch-v1.0.1-bn-v0.6.0">pico-patch-v1.0.1-bn-v0.6.0</a></td></tr><tr><td>bigint</td><td>https://github.com/brevis-network/crypto-bigint</td><td><a href="https://github.com/brevis-network/crypto-bigint/releases/tag/pico-patch-v1.0.0-bigint-v0.6.0">pico-patch-v1.0.0-bigint-v0.6.0</a></td></tr></tbody></table>
