
# Migrating to 3.0

Pico 3.0 replaces the proof system: it proves over the KoalaBear field with an MLE proof system (LogUp-GKR, ZeroCheck and WHIR), and the host-side API moved to a new crate. 2.x proofs do not verify under 3.0, and 2.x on-chain verifiers do not accept 3.0 proofs.

This page lists the changes most projects hit. The full guide is [Migrating from 2.x to 3.0](https://github.com/brevis-network/pico/blob/v3.0.0/docs/migration-2.x-to-3.0.md), and the [3.0 release notes](https://github.com/brevis-network/pico/blob/v3.0.0/docs/release-notes/3.0.md) summarize the release.

### Toolchain

* The Rust toolchain moved from `nightly-2025-08-04` to `nightly-2026-09-27`. Reinstall the CLI with it (see [Installation](../getting-started/installation.md)), then run `cargo pico install` again for the matching guest toolchain.
* Set the project's own `rust-toolchain` file to `nightly-2026-09-27`: the host crate builds `pico-prover`, which uses nightly-only features.
* The guest target is still `riscv64im-pico-zkvm-elf`.
* The guest depends on `pico-sdk = { git = "https://github.com/brevis-network/pico" }`, as before.

### Fields and phases

* KoalaBear is the only field. BabyBear and Mersenne31 are removed with no replacement: the `bb` and `m31` features of `pico-sdk` are gone, and `kb` is kept as a no-op so existing guests still build.
* The `COMPRESS` phase is removed. The chain is now `RISCV → CONVERT → COMBINE → EMBED → WRAP`; see [ProverChain](../writing-apps/advanced/proverchain.md).

### CLI

| 2.x                                  | 3.0                                                                                                                                                                                                                                       |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cargo pico prove --fast`            | `cargo pico prove --phase riscv`. The RISC-V phase is full-security; `--fast` now fails with this hint                                                                                                                                    |
| `cargo pico prove --field`           | removed; KoalaBear is the only field, and the flag fails with an error                                                                                                                                                                    |
| `--elf` optional                     | `--elf` required                                                                                                                                                                                                                          |
| `--input "0x…"` or a file path, once | repeatable and typed: `--input u64:100`, `--input 0x…`, `--input file:<path>`                                                                                                                                                             |
| output in `<target>/pico_out`        | output in `pico_out` under the current directory (`--output` to change)                                                                                                                                                                   |
| `prove` stops after EMBED            | `prove` stops after COMBINE by default; `--phase` chooses                                                                                                                                                                                 |
| `--evm` with a fixed Docker image    | `--evm` with gnark from `$PICO_GNARK_BIN` or `$PICO_GNARK_IMAGE`, in whitelist mode only, with keys from `--keys DIR` or `$PICO_GROTH16_KEYS`. See the [gnark README](https://github.com/brevis-network/pico/blob/v3.0.0/gnark/README.md) |

New subcommands: `cargo pico execute` (emulate without proving) and `cargo pico aot` (generate AOT crates). See the [CLI section of the migration guide](https://github.com/brevis-network/pico/blob/v3.0.0/docs/migration-2.x-to-3.0.md#cli-cargo-pico).

### Host SDK: `pico-sdk` client → `pico-prover`

The prover clients left `pico-sdk`. The host program depends on the new crate `pico-prover`:

```toml
pico-prover = { git = "https://github.com/brevis-network/pico" }
```

| 2.x (`pico_sdk::client`)                                          | 3.0 (`pico_prover`)                                                                                                                             |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `DefaultProverClient::new(&elf)` (and the other `*ProverClient`s) | `Prover::new(&elf)?`, or `Prover::builder(&elf)…build()?` for options                                                                           |
| `client.new_stdin_builder()` + `write`                            | `Stdin::new()` + `write` / `write_bytes`, or `Stdin::from(&value)`                                                                              |
| `prove_fast(stdin)`                                               | `prove_to(stdin, Phase::Riscv)`                                                                                                                 |
| `prove(stdin)` → `(riscv_proof, embed_proof)`                     | `prove(stdin)` → one `Proof` at COMBINE, already verified                                                                                       |
| `verify(..)`                                                      | no separate call: every returned `Proof` is verified. A proof read from a file goes through `ProofArtifact::read` and `Prover::verify_artifact` |
| `emulate(stdin)`                                                  | `execute(stdin)`; its output is not proof-authenticated                                                                                         |
| `prove_evm(stdin, need_setup, out, field)`                        | `prove_evm(stdin)` → `EvmBundle`, whitelist mode only. Groth16 itself runs outside the library (`cargo pico prove --evm`)                       |
| `write_onchain_data(..)`                                          | `Proof::write_wrap_artifacts` or `EvmBundle::write_to`                                                                                          |
| `proof.pv_stream`                                                 | `proof.committed().as_bytes()`                                                                                                                  |

See [Proving](../writing-apps/proving.md) for the new API.

### Guest SDK

* `pico_sdk::poseidon2_hash` is deprecated: its single-element output is not collision-resistant. Use `pico_sdk::poseidon2::hash` (fixed-length input).
* `syscall_poseidon2_permute` now takes `*const Poseidon2State, *mut Poseidon2State`.

### Output files

* `constraints.json` and `groth16_witness.json` are now `wrap_constraints.json` and `wrap_groth16_witness.json`.
* RISC-V and COMBINE proofs are written as `proof-riscv.pico` and `proof-combine.pico`.
* `inputs.json` gains a `committedValuesDigest` field.

### On-chain verifier

`verifyPicoProof` now takes a twelve-word proof:

```solidity
function verifyPicoProof(bytes32 riscvVkey, bytes calldata publicValues, uint256[12] calldata proof) external view;
```

In 2.x it took `uint256[8]`. The extra four words are the Pedersen commitment and its proof of knowledge that the 3.0 wrap circuit carries. Redeploy the verifier contracts; see the [gnark README](https://github.com/brevis-network/pico/blob/v3.0.0/gnark/README.md).
