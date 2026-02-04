---
name: julia-hpc
description: Advanced Julia performance optimization focusing on SIMD, zero-allocations, and LLVM/Native code profiling.
---

# Julia HPC Performance Mandate

## 1. Zero-Allocation Design

- **Rule:** Performance-critical inner loops must have **zero heap allocations**.
- **Action:** - Use `StaticArrays.jl` for small fixed-size vectors/matrices.
  - Use `Views` (`@views`) to avoid slicing copies.
  - Pre-allocate work buffers and pass them as arguments (mutating functions `f!(...)`).
  - Use `@allocated` to verify 0 bytes are being used in tight loops.

## 2. Type Stability & Introspection

Before finalizing any code, you MUST run and analyze the following macros:

- `@code_warntype`: Identify any `Any` types or `Union` types highlighted in red/yellow. Fix these by using type annotations or constant globals.
- `@code_llvm`: Check if the intermediate representation is clean. Look for `alloca` (stack allocation) vs heap calls.
- `@code_native`: Look for **SIMD instructions** (e.g., `vaddpd`, `vmulpd` on x86_64). If you see scalar instructions in a loop, the loop did not vectorize.

## 3. Hardware-Aware Metrics

Report the following in a generated `performance.md` file in the project root:

- **Roofline Analysis Estimates:** - Calculate **Arithmetic Intensity** ($I = \frac{FLOPs}{Bytes}$).
  - Estimate % of **Peak FLOPs** (Theoretical Max vs Observed).
  - Estimate **Memory Bandwidth** usage (Is it L3 cache bound? Bound by DRAM throughput?).
- **Cache Locality:** Ensure column-major access (iterate over the first index first) to maximize cache line utilization.

## 4. Vectorization & Parallelism

- Use `@simd` and `@inbounds` only after ensuring loop independence.
- Leverage `LoopVectorization.jl` (`@turbo`) for complex kernels to force AVX-512/AVX2 utilization.
- Use `Threads.@threads` or `Polyester.jl` (`@batch`) for low-overhead multi-threading.

## 5. Performance Reporting Template

Every project must output a `performance.md` containing:

| Metric | Value | Observation |
| :--- | :--- | :--- |
| Allocations | [X] bytes | Must be 0 for inner kernels. |
| Type Stability | [Pass/Fail] | Checked via @code_warntype. |
| SIMD Status | [Active/Inactive]| Verified via @code_native. |
| Bottleneck | [Compute/Memory] | Is it L3 bound or DRAM bound? |
