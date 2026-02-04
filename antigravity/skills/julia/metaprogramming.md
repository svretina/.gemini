# Julia Metaprogramming

This skill guide covers the principles and traps of Julia's metaprogramming system (Macros, Expressions, and Generated Functions).

## 1. Expressions and Evaluation

- **Symbols**: Represent identifiers. Created with `:x` or `Symbol("x")`.
- **Expressions**: Tree structures representing code. Created with `:(x + y)` or `Expr(:call, :+, :x, :y)`.
- **Evaluation**: `eval(expr)` evaluates an expression in the global scope of the current module. **Avoid `eval` inside functions** for performance-critical code.

## 2. Macros

Macros transform expressions at parse time.

- **Definition**: `macro name(ex) ... end`.
- **Hygiene**: Macros are hygienic by default. Locals are `gensym`'d to avoid name collisions.
- **Escaping**: Use `esc(ex)` for arguments that should be resolved in the caller's environment (e.g., variables passed to the macro).

  ```julia
  macro my_macro(ex)
      return quote
          local tmp = ... # gensym'd
          $(esc(ex))      # resolved in caller
      end
  end
  ```

- **QuoteNode**: Use `QuoteNode(:x)` to prevent interpolation of symbols when passing them into expressions.

## 3. Generated Functions

`@generated` functions allow code generation based on argument *types* rather than values.

- **Capabilities**: Expanded when argument types are known, before compilation.
- **Restrictions**:
  - Must be **pure** (no side effects, no global state access except constants).
  - Can only access types, not values.
  - Cannot define closures or generators.
- **Usage**: Use for extreme performance optimizations where dispatch is insufficient.

  ```julia
  @generated function my_func(x)
      if x <: AbstractFloat
          return :(x * 2.0)
      else
          return :(x + 1)
      end
  end
  ```

## 4. Best Practices

- **Macros are for syntax, not logic**: If it can be done with a function, use a function.
- **Minimize `@generated` usage**: They are powerful but restrictive and harder to debug/maintain.
- **Interpolation**: Use `$` for interpolation in `quote` blocks and `$(esc(ex))` for macro arguments.
- **Macro Expansion**: Use `@macroexpand` to debug what a macro actually produces.
