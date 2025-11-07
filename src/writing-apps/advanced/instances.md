---
description: Customizable zkVM Instantiations
---

# Instances

Pico is architected as a chain of modular components, each tailored to perform a specific role in the overall ZK proof generation process. These components—known as **machine instances**—are instantiations of a virtual machine and comprise several submodules, including chips, compilers, emulators, and proving backends. This modular design not only simplifies the internal workflow but also provides developers with the flexibility to customize and extend the system to meet diverse application needs.

## Built-in Machine Instances&#x20;

The current release of Pico includes several built-in machine instances, each designed for a distinct phase of the proof generation pipeline:

### `RISCV`&#x20;

The `RISCV`  instance is responsible for executing RISCV programs and generating the initial STARK proofs. It achieves this by:

* Execution & Chunking: Running the program and dividing it into smaller, manageable chunks.
* Parallel Proof Generation: Proving these chunks concurrently to generate a series of proofs, with the total number of proofs equaling the number of chunks.

### `CONVERT`&#x20;

Acting as the first step in the recursion process, the `CONVERT` instance transforms each STARK proof produced by the `RISCV` instance into a recursion-compatible STARK proof. This conversion is crucial for setting the stage for recursive proof composition.

### `COMBINE`&#x20;

The `COMBINE` instance aggregates `m` of recursion proofs generated from the **same** machine instance into a single STARK proof. By default, `m` is set to 2 in Pico, though it can be configured to a larger value. This instance is applied recursively to collapse a large collection of proofs into one final proof, forming a recursion tree. For example, if you start with `n` proofs, the first layer uses `n/m` COMBINE calls to produce `n/m` proofs; these are then aggregated in subsequent layers (`n/m²`, `n/m³`, etc.) until only one proof remains. This consolidation streamlines subsequent processing and reduces overall complexity.

### `COMPRESS`&#x20;

Aiming to optimize efficiency in later recursive stage, the **COMPRESS** instance compresses a recursion STARK proof into a smaller-sized proof.

### `EMBED`&#x20;

As the final stage in generating a STARK proof, the **EMBED** instance embeds the STARK proof into the BN254 field. This prepares the proof for later conversion into an on-chain-verifiable SNARK.

## Modularity and Internal Extensibility

Pico’s machine instances are designed with a strong emphasis on modularity and internal extensibility:

* **Purpose-Driven Specialization:** Each machine instance is engineered to execute a specific phase of the proof generation process. This targeted design enhances performance and simplifies debugging, as each instance handles a distinct, well-defined task.
* **Isolated Upgradability:** The self-contained nature of each machine instance allows developers to update, optimize, or replace individual components independently. This isolation promotes rapid iteration and integration of cutting-edge cryptographic techniques without disrupting the overall system.
* **Flexible Submodule Architecture:** Within each instance, core functionalities are implemented via interchangeable submodules (e.g., chips, compilers, emulators, proving backends). This design enables targeted enhancements, such as swapping out a proving backend to leverage a more efficient prime field, without modifying the instance’s primary function.
* **Seamless Future Integration:** By compartmentalizing functionalities into discrete units, Pico is primed for the adoption of new technologies. As innovative proving systems and cryptographic primitives emerge, they can be integrated into the framework without a complete overhaul, ensuring the platform evolves alongside technological advancements.
