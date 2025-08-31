# Thyse Language

**Thyse** (pronounced /ˈtɪsə/ or TI-sa) is a domain-specific language (DSL) for describing fully homomorphic encryption (FHE) schemes and computations. It aims to make FHE programs **concise, precise, and expressive**, while exposing mathematical structure to the compiler for optimization, verification, and MLIR code generation.

Thyse emphasizes **immutability, strong typing, and mathematical clarity**. Noise growth, levels, scales, and moduli are modeled directly in the type system, so invalid or unsafe FHE operations can be rejected at compile time.


## Language Features

Thyse provides these syntactic and semantic categories:

* Comments & documentation
* Imports & contexts
* Variable declarations
* Function definitions
* Function calls
* Operator expressions
* Relational expressions
* Conditionals
* Iteration
* Data structures
* Effects & purity
* Modules & exports


## Example Program

```thyse
import fhe.core (enc, dec, keyswitch) using BGV(q=2^60, p=257, n=2^15, λ=128)

/// Key switching via mod-up → eval-key mult → mod-down.
/// Compiler checks level and noise budgets at compile time.
def keyswitch(a : BP, key : Key) : BP[2] !Keyed {
  let bs = mod_up(a)          # : BP[n] inferred
  let y  = mult_eval_key(bs, key)  # : BP[2]
  mod_down(y)                 # last expression = return
}

let x = enc(7)
let y = keyswitch(x, Key())
let z = dec(y)
```


## Comments & Documentation

* **Line comments:** introduced with `#`
* **Trailing comments:** allowed after code
* **Block comments:** enclosed by `/* … */`
* **Doc comments:** triple-slash `///` captured as documentation

```thyse
# Single line
let a : CT   # trailing

/* Multi-line
 * explanation
 */

/// Encodes a plaintext integer into a ciphertext.
def enc(x : Int) : CT
```


## Imports & Contexts

Contexts define scheme-level parameters (modulus, plaintext space, dimension, security level).
An `import … using …` statement pulls functions or types into scope **parameterized by a context**.

```thyse
import fhe.bgv (enc, dec, add, mult) using BGV(q=2^60, p=257, n=2^15, λ=128)

let a = enc(42)
let b = enc(17)
let c = a + b    # addition under BGV context
```


## Variable Declarations

* `let` introduces immutable variables (preferred)
* `var` introduces mutable variables (explicit, discouraged)
* Type annotations may be inferred locally but are required at module boundaries.
* Multiple variables of the same type can be grouped.

```thyse
let a = enc(7)        # inferred type : CT
var counter : Int     # mutable
let u, v, w : BP      # grouped decl
```


## Function Definitions

Functions are defined with `def … end` (or with inline body syntax).
The return type must be declared.
Last expression in the body is returned implicitly.

```thyse
def id(x : CT) : CT = x

def rotate(a : CT, k : Int) : CT
  let r = galois_eval(a, k)
  r
end
```


## Function Calls

Functions are called with positional arguments.
Trailing commas and multi-line argument formatting are supported.

```thyse
let a = enc(5)
let b = enc(6)

let c = add(
  a, # message
  b  # mask
)
```


## Operator Expressions

* Standard arithmetic operators `+ - *` are overloaded by type.
* `*` on ciphertexts is resolved to FHE multiplication.
* Unary `-` and `not` are supported.

```thyse
let a, b, c : CT
(a + b) * (c - a)

let p : Bool = true
let q = not p
```


## Relational Expressions

Relational operators return boolean values:

```thyse
a < b
a <= b
a > b
a >= b
a == b
a != b
```


## Conditionals

Conditionals are expressions and produce values:

```thyse
let r = if noise(x) < q/4 then fast(x) else safe(x) end

if flag
  do_this()
else if cond
  do_that()
else
  do_other()
end
```


## Iteration

Supports both imperative loops and functional style.

```thyse
for i in 0..n {
  use(i)
}

let ys = [ f(x) for x in xs if pred(x) ]
let total = reduce(xs, +, 0)
```


## Data Structures

### Tuples & Records

```thyse
let pair = (a, b)
let key : { galois : KeyG, relinear : KeyR }
key = { galois: gk, relinear: rk }
```

### Vectors

Length is part of the type when known.

```thyse
let v : Vec[CT, 8]
let x = v[3]   # bounds-checked, 0-based
```


## Effects & Purity

Effects mark operations requiring secret or key-dependent resources.
Purity is the default.

```thyse
def keyswitch(a : BP) : BP[2] !Keyed
def sample_noise() : Noise !Random
```


## Modules & Exports

Modules provide namespace organization.
Explicit exports define public API.

```thyse
module lwe
  export enc, dec, sample

  def enc(x : Int) : CT = ...
end

import lwe (enc, dec) using LWE(n=512, q=2^30)
```


## Compile-time Constants & Constraints

Constants and type-level naturals allow the compiler to enforce invariants.

```thyse
const L = 10

def rescale(x : CT[L]) : CT[L-1] where L > 0 = ...
```


## Pipelines & Dataflow

Pipelines improve readability for FHE circuits.

```thyse
x
  |> enc
  |> poly_eval(f)
  |> rescale
  |> mod_switch(q=2^50)
```


## Ring & Polynomial Literals

Algebraic structures are explicit in syntax.

```thyse
ring R = Z_(2^60)[x]/(x^n + 1)
let p = poly(x^3 + 2*x + 1) in R
```


## Summary

Thyse provides a **math-first DSL for FHE** that:

* Uses **immutable, strongly typed constructs** with inference.
* Exposes **scheme metadata** (levels, modulus, precision) to the type system.
* Supports **contexts via import using** syntax to parameterize scope.
* Compiles directly to optimized IR (e.g., MLIR) without dependency on external crypto backends.
* Offers a modern, minimal syntax inspired by functional and algebraic languages.
