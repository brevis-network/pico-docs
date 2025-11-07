
# ProverChain

Pico empowers developers with the **ProverChain**—a feature that enables you to seamlessly chain together machine instances to create a complete, end-to-end ZK proof generation workflow. Leveraging Pico’s modular architecture, ProverChain allows you to design workflows tailored precisely to the needs of your application.

## Proving Phases <a href="#provechain-proving-phase" id="provechain-proving-phase"></a>

Pico’s proving process is structured into distinct phases:

* **RISCV-Phase:** The `RISCV` instance executes a RISCV program, generating a series of initial proofs.
* **RECURSION-Phase:** The `CONVERT`, `COMBINE`, `COMPRESS`, and `EMBED` instances work together recursively to consolidate these proofs into a single STARK proof.
* **EVM-Phase:** This final STARK proof is then fed into a Gnark prover to generate a on-chain-verifiable SNARK that is ready for deployment on EVM-based blockchains.

## Default Proving Workflow

By default, Pico constructs a proving workflow by chaining the following machine instances:

**`RISCV → CONVERT → COMBINE → COMPRESS → EMBED → ONCHAIN (optional)`**

In this sequence:

* **RISCV-** and the **RECURSION-Phase** handle the initial execution and recursive proof generation. It takes a RISCV program and input and generates an embedded STARK proof.
* **ONCHAIN**—an optional instance—works at **EVM-Phase** and converts the embedded STARK proof into an EVM-verifiable SNARK.

## Customizing Your Workflow

While the default workflow is designed for uniform efficiency, ProverChain offers exceptional flexibility, enabling developers to tailor the proving process to their specific requirements:

* **Chain Modification:** Easily add, adjust, or remove machine instances. For example, if on-chain verification is not required, you can simply omit the ONCHAIN step.
* **Performance Optimization:** Experiment with different configurations to achieve the optimal balance between proof size and proving efficiency. In some scenarios, accepting a slightly larger proof can lead to faster overall performance.
* **Intermediate Access:** The ProverChain module exposes the intermediate steps—formatted as a sequence (e.g., `stdin -> proof -> proof -> ... -> final proof`)—allowing you to fine-tune internal parameters at each stage of the workflow.
