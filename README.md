This is an attempt to create a "Featherweight Rust"---that is, a
formal version of Rust where we omit all the complicated, annoying
details.  The intention is that Featherweight Rust should be a subset
of Rust: ideally, all Featherweight Rust programs will also be legal
Rust programs, but I am also willing to accept various small syntactic
differences. The subset should be enough to write executable programs.

The formal model will be broken up into several layers of increasing
complexity.  For now I am focusing on the core.  Included in the model are:

- Records
- Boxes, mutable and immutable
- Uniques, mutable and immutable
- Tags
- Pattern matching against tags
- Lambdas

Excluded from the core are:

- Tasks (Rust's strict memory separation makes these easily layerable)
- Objects (can be modeled with records of fn ptrs)
- Resources (eh)
- Uninitialized variables and ret statements (eh)
- While loops (use functions)
- Tuples (modeled by records)
- Blocks and unique closures (modeled by lambdas and mutable boxes)
- Typestate and refined types (can wait for another layer)

The document is broken up into several parts:

- The [typographical conventions][tc] explains my notation.
- The [grammar][gr] describes the
  syntax of a Featherweight Rust program.
- The [type system][ts] document gives the type rules for checking
  whether a Rust program is legal.
- The [dataflow module][df] implements a simplified version of our
  typestate algorithm.

Still to come:

- Operational semantics
- Proofs
- Extensions (Task system, variations, etc)

[tc]: rust-formal/blog/master/core/notation.md
[gr]: rust-formal/blob/master/core/grammar.md
[ts]: rust-formal/blob/master/core/typesys.md
[df]: rust-formal/blob/master/core/dataflow.md
