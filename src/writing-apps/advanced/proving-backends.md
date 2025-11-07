
# Proving Backends

One of Pico’s most innovative feature is its ability to seamlessly switch between multiple proving backends. This functionality enables you to select the optimal backend for your specific application requirements, resulting in significant efficiency gains without altering your existing proving logic.

## Why Multiple Proving Backends Matter

Specialized circuits for different application features often demand advanced proving systems optimized for specific prime fields. Consider, for example, the recursive proving of a hash function like **Poseidon2**—a critical component in Pico’s recursive proving strategy. Although the same STARK proving system is used, working on the **KoalaBear** field can be much more efficient than on the **BabyBear** field due to the inherent properties of these fields. As a result, when a program requires extensive Poseidon2 proving, simply switching to KoalaBear can yield considerable performance improvements.

## Supported Proving Backends

Currently, Pico supports generating proofs in all phases—RISCV, RECURSION, and EVM—with both STARK on KoalaBear and STARK on BabyBear. For CircleSTARK on Mersenne31, Pico currently supports the RISCV-Phase, with RECURSION and EVM phases coming soon.

* STARK on KoalaBear (prime field $$p=2^{31}-2^{24}+1$$): Supports generating proofs for
  * [x] &#x20;RISCV-Phase
  * [x] &#x20;RECURSION-Phase
  * [x] &#x20;EVM-Phase
* STARK on BabyBear (prime field $$p=2^{31}-2^{27}+1$$): Supports generating proofs for
  * [x] &#x20;RISCV-Phase
  * [x] &#x20;RECURSION-Phase
  * [x] &#x20;EVM-Phase
* CircleSTARK on Mersenne31 where $$p=2^{31}−1$$). Supports generating proofs for
  * [x] &#x20;RISCV-Phase
  * [ ] &#x20;RECURSION-Phase
  * [ ] &#x20;EVM-Phase

## Seamless Backend Switching

Switching between proving backends in Pico is designed to be straightforward. The underlying proving logic is abstracted away, allowing you to change the backend configuration through a simple parameter update—without needing to rewrite any part of your application.

The Pico SDK provides a suite of `ProverClient` implementations, each corresponding to a different proving backend:

* **KoalaBearProverClient:** Uses STARK on KoalaBear for fast proving without VK (verification key) verification.
* **KoalaBearProverVKClient:** Uses STARK on KoalaBear for full proving with VK verification.
* **BabyBearProverClient:** Similar to KoalaBearProverClient, but for STARK on BabyBear.
* **BabyBearProverVKClient:** Similar to KoalaBearProverVKClient, but for STARK on BabyBear.
* **M31RiscvProverClient:** Performs RISCV proving using CircleSTARK on Mersenne31.

We could initialize the `ProverClient` for different backend configurations:

```rust
// An example for initializing the different prover clients
fn main() {
    // Initialize logger.
    init_logger();

    // Load the ELF file.
    let elf = load_elf("./elf/riscv32im-pico-zkvm-elf");

    // Initialize a client for fast proving (without VK verification) 
    // using STARK on KoalaBear.
    let client = KoalaBearProverClient::new(elf);

    // Initialize a client for full proving with VK verification 
    // using STARK on KoalaBear.
    let client = KoalaBearProverVKClient::new(elf);

    // Initialize a client for fast proving (without VK verification) 
    // using STARK on BabyBear.
    let client = BabyBearProverClient::new(elf);

    // Initialize a client for full proving with VK verification 
    // using STARK on BabyBear.
    let client = BabyBearProverVKClient::new(elf);

    // Initialize a client for RISCV proving using CircleSTARK on Mersenne31.
    let client = M31RiscvProverClient::new(elf);
}
```

## Benefits of Switchable Proving Backends

* **Performance Gains:** Optimize your proof generation by selecting the backend that best suits the computational demands of your workload.
* **Flexibility:** Experiment with different backends and configurations to achieve the ideal balance between proof size, proving efficiency, and on-chain compatibility.
* **Seamless Upgrades:** As new prime fields and proving systems are integrated into Pico, you can upgrade your proving backend with minimal disruption.
* **Future-Proofing:** Stay at the forefront of zero-knowledge technology advancements by taking advantage of the latest proving systems as they become available.
