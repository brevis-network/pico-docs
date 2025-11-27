# Cost Estimation

The Pico emulator also provides a method for obtaining a rough estimate for the answer to the question: how many CPU cycles would it take to prove this trace? This is done by setting the `EmulatorOpts::cost_estimator` flag to `true` or by using `EmulatorOpts::with_cost_estimator`. This produces a value in the `host_cycle_estimator` field of the `EmulationReport` per batch, and the `Vec` entries of type `CycleEstimator` can be used to estimate the cycles on a given model. Each entry corresponds to the chunk index within the specific batch. The current model used by Pico is present in `model.json`, found at the root of the repository, and can be loaded with `EstimatorModel::from_json(path)`. By using the estimator data with different models of host prover, different numbers may be obtained.

Currently, the Pico model universally divides the estimator result by 1000 as a means to avoid overflow. This is subject to change.
