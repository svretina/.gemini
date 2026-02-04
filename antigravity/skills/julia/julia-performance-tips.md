# julia-performance.md

## 🎯 Purpose

This document provides a compressed knowledge base of Julia performance optimization. Use this as a reference for the **Analyzer**, **Implementer**, and **Critique** agents to ensure Julia code meets production-grade performance standards.

---

## 🚀 1. Fundamental Principles

* **Encapsulate in Functions:** Never run performance-critical code in the global scope. Global variables are untyped and hinder the compiler's ability to optimize.
* **Type Stability:** Ensure functions return the same type regardless of input values. Use `@code_warntype` to detect "Any" or "Union" return types.
* **Avoid Abstract Containers:** Use `Vector{Float64}` instead of `Vector{Real}`. Abstract types in containers force Julia to use boxed pointers, destroying performance.
* **Concrete Fields:** Always define `struct` fields with concrete types or type parameters:

    ```julia
    struct MyType{T<:AbstractFloat}
        val::T 
    end
    ```

## 🧠 2. Type System & Dispatch

* **Function Specialization:** Julia specializes functions based on argument types. Avoid over-specifying types in signatures unless needed for dispatch (it doesn't help performance).
* **Kernel Functions:** If a function has a type-unstable part, wrap the performance-critical loop in a separate "kernel" function to allow the compiler to specialize the inner loop once the type is known.
* **Value Types:** Use `Val{T}` for dispatching on values only when the value is known at compile time and the number of variations is small.

## 💾 3. Memory & Allocation

* **Pre-allocate Output:** Use mutating functions (ending in `!`) to avoid repeated allocations in loops.
  * *Good:* `filter!(x -> x > 0, my_vector)`
  * *Bad:* `my_vector = filter(x -> x > 0, my_vector)`
* **Views vs. Copies:** Use `@views` when slicing arrays to prevent creating temporary copies of data.
* **Dot Vectorization:** Use the fusion operator `.` (e.g., `f.(x) .+ g.(y)`) to merge multiple operations into a single loop, minimizing temporary allocations.
* **StaticArrays:** Use `StaticArrays.jl` for small, fixed-size arrays (usually < 100 elements) to allow stack allocation.

## 🛠 4. Tools & Diagnostics

* **Benchmarking:** Use `BenchmarkTools.jl` and its `@btime` macro. Never use `@time` for a single run as it includes compilation time.
* **Profiling:** Use the built-in `Profile` module or `StatProfilerHTML.jl` to identify bottlenecks.
* **Allocation Tracking:** Start Julia with `--track-allocation=user` to see exactly which lines are allocating memory.

## ⚡ 5. Advanced Tweaks

* **Inlining:** Use `@inline` for very small, frequently called functions.
* **SIMD & Loop Optimization:** Use `@simd` for loops to allow vectorization. Use `@inbounds` to skip array bounds checking **only** after you have verified the logic is safe.
* **Avoid String Interpolation in I/O:** Pass multiple arguments to `print` or `write` instead of creating a large interpolated string.
* **Subnormal Numbers:** If your math involves very small floats, use `set_zero_subnormals(true)` to avoid the performance penalty of subnormal floating-point arithmetic.

---

## 📋 Agent Checklist

### 🏗 Analyzer (Architect)

- [ ] Is the logic decomposable into small, type-stable functions?
* [ ] Are we using the correct data structures (e.g., Dict vs. NamedTuple)?
* [ ] Have we identified the "hot loops" for pre-allocation?

### 💻 Implementer (Coder)

- [ ] Are all fields in structs concrete?
* [ ] Is `@views` applied to array slicing?
* [ ] Are we using `.` broadcasting to fuse operations?
* [ ] Is there any global state that should be passed as a function argument?

### 🧐 Critique (Validator)

- [ ] Does `@code_warntype` show any red "Any" or "Union" types?
* [ ] Does `@btime` show unexpected allocations (look for `allocs` > 0 in tight loops)?
* [ ] Are there `@inbounds` annotations that lack safety checks?
* [ ] Can any `Vector` be replaced with a `StaticArray` or `Tuple`?
