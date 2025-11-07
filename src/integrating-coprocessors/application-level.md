---
description: Integrating Beyond Function-Level Precompiles
---

# Application-level

Application-level coprocessors extend far beyond individual function-level precompiles. Instead of optimizing a single cryptographic operation, these coprocessors integrate an array of specialized circuits that work together to tackle broader, domain-specific computational challenges. By incorporating application-level coprocessors, Pico not only enhances its performance but also serves as a versatile "glue" that seamlessly routes data between high-efficiency modules. This design enables Pico to be finely tuned for specific applications without sacrificing its overall flexibility and general-purpose utility—resulting in enhanced performance, improved scalability, and accelerated development cycles.

Pico can integrate a variety of exceptional coprocessors across different domains. For example:

* **On-Chain Data zkCoprocessors**: Engineered to provide efficient and secure access to historical blockchain data, these coprocessors enables developers to retrieve and analyze past transaction records, state data, and other on-chain information with confidence. Brevis Coprocessor has already been successfully integrated into Pico. This solution will offer a comprehensive framework for building applications that depend on verifiable, reliable on-chain data processing. _Detailed integration guidelines will be available soon._
* **zkML (Zero-Knowledge Machine Learning) Coprocessors** : These coprocessors leverages ZK proofs to enable secure, privacy-preserving training and inference for machine learning models. It ensures that sensitive data and/or proprietary model information remain confidential throughout the process, opening the door to advanced, secure ML applications.

These application-level coprocessors empower Pico to support highly specialized, domain-specific tasks while preserving the generality and flexibility that make it a robust platform for a wide range of zero-knowledge applications.

