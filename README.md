This is an attempt to create a "Featherweight Rust"---that is, a
formal version of Rust where we omit all the complicated, annoying
details.  The intention is that Featherweight Rust should be a subset
of Rust: ideally, all Featherweight Rust programs will also be legal
Rust programs, but I am also willing to accept various small syntactic
differences.

The formal model will be broken up into several layers of increasing
complexity.  For now I am focusing on the core.  Included in the model are:

- Records
- Boxes, mutable and immutable
- Uniques, mutable and immutable
- Tags
- Pattern matching against tags
- Lambdas

Excluded are:

- Tasks
- Objects
- Resources
- Tuples
- Blocks and unique closures

The document is broken up into several parts:

- The [grammar](rust-formal/blob/master/core/grammar.md) describes the
  syntax of a Featherweight Rust program.
- The [type system](rust-formal/blob/master/core/typesys.md) document gives
  the type rules for checking whether a Rust program is legal.

Still to come:

- Operational semantics
- Proofs
- Extensions (Task system, variations, etc)
