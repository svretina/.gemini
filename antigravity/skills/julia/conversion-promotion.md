# Julia Conversion & Promotion

This skill guide covers the idiomatic use and extension of Julia's conversion and promotion mechanisms.

## 1. Conversion

Conversion is the process of obtaining a value of a specific type `T` from a value `x`.

- **Explicit Conversion**: Always prefer `T(x)` (constructor) unless implicit behavior is required.
- **Implicit Conversion**: Managed by `Base.convert(T, x)`. It is called automatically in certain contexts (e.g., array assignment `A[i] = x`).
- **Safety**: Only define `convert` methods for conversions that are "safe"—where the value is faithfully represented (e.g., `Int` to `Float64` is safe, but `Float64` to `Int` might lose information and should often remain explicit).
- **Implementation**:

  ```julia
  import Base: convert
  convert(::Type{MyType}, x) = MyType(x)
  ```

- **Parsing**: Do NOT use `convert` or constructors for string parsing. Use `parse(T, str)`.

## 2. Promotion

Promotion is the process of converting values of mixed types to a single common "greater" type.

- **`promote(x, y, ...)`**: Returns a tuple of values converted to a common type.
- **Catch-all Methods**: Use `promote` to handle mixed-type operations:

  ```julia
  +(x::MyNumber, y::Number) = +(promote(x, y)...)
  ```

- **Promotion Rules**: Define how types should be promoted together by overloading `Base.promote_rule`:

  ```julia
  import Base: promote_rule
  promote_rule(::Type{MyType}, ::Type{Float64}) = MyType
  ```

- **Symmetry**: `promote_rule(A, B)` implies `promote_rule(B, A)`; you only need to define one.
- **Splatting**: Use `promote_type(A, B)` to query the result type without actual values.

## 3. Best Practices

- **Avoid Direct `promote` Overloading**: Always overload `promote_rule` instead of `promote` or `promote_type`.
- **Inheritance vs. Promotion**: Promotion is about representation, not OOD subtyping. `Int32` is not a subtype of `Float64`, but it promotes to it.
- **Performance**: Julia's promotion mechanism is highly optimized and usually zero-cost after compilation.
