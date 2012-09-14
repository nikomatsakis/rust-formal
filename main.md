## Grammar ##

    rv = x
       | x.f
       | *x
       | &_r mq lv
       | ~ mq x
       | ()
    lv = x
       | lv.f
       | *x
    ex = copy rv
       | move lv
    st = lv = ex 
       | lv <-> lv
       | g(exs)
       | r: "{" {st} "}"
    fn = "fn" g(xs:tys) "{"
             let ms xs:tys;
             sts
         "}"
    ty = "{" ms fs:tys "}"
       | ~ m ty
       | &_r m ty
       | ()
    rp = fns
    m  = mut
       | imm
       | const
       
Some of the things we left out or simplified:

- all locals, fields are mutable
- we assume local variables are not used until they are fully initialized
- functions do not return values, instead you can just pass a mutable record
  where the result should be stored
- no first-class functions
- no scalar types but unit
- no boxes
- no enums, vectors
  
## Operational Semantics ##

Let the *heap* `H` be a map from an address `a` to an atomic value, which is
either another address or unit.  

Let the *stack* `S` be a list comprised of entries:

    S = []
      | S, (A, {st})
      
where `A: {x -> a}` is an *activation record*, which maps from the
local variable names in the stack frame to the addresses where their
values are stored.

A machine configuration `C` is then defined as:

    C = H; S

We assume that all local variables and fields in the program have
unique names, and define a helper function `type(x)` that returns the
type declared for a given variable and `type(f)` that returns the type
declared for a given field.  The rules are always defined relative to
a given fixed program text.

The initial configuration of the machine is

    []; [], ([], main())

Let `sizeof(ty)` be a function defined as follows:

    sizeof(()) = 1
    sizeof(~ty) = 1
    sizeof(&_r ty) = 1
    sizeof("{" {f:ty} "}") = sum(sizeof({ty}))

Let `offsetof(f)` be the sum of the sizes of all fields prior to the
given field.

Let `a(H,A,lv)` be a function that computes the address where a given
lvalue is stored:

    a(H,A,x) = A(x)
    a(H,A,lv.f) = a(H,S,lv) + offsetof(f)
    a(H,A,*x) = H(A(x))

The small step semantics are defined by a relation `C -> C'`.

    Rule Var:
    H; S, (a, A, (lv = x, {st})) -> H'; S, (a, A, {st}) where
        Z = sizeof(type(x))
        H' = H[a(H,S,lv) ->_Z H(A(x))]

    Rule Field:
    H; S, (a, A, (lv = x.f, {st})) -> H'; S, (a, A, {st}) where
        Z = sizeof(type(f))
        H' = H[a(H,S,lv) ->_Z H(A(x) + offsetof(f))]

    Rule Deref:
    H; S, (a, A, (lv = *x, {st})) -> H'; S, (a, A, {st}) where
        type(x) = ~ty or &_r ty
        Z = sizeof(ty)
        H' = H[a(H,S,lv) ->_Z H(H(A(x)))]

    Rule AddrOf:
    H; S, (a, A, (lv = &lv', {st})) -> H'; S, (a, A, {st}) where
        H' = H[a(H,S,lv) ->_1 a(H,S,lv')]

    Rule Alloc:
    H; S, (a, A, (lv = ~x, {st})) -> H'; S, (a, A, {st}) where
        Z = sizeof(type(x))
        a_1..a_Z = continuous range of addresses fresh in H
        H' = H[a_1 ->_Z H(A(x)),
               a(H,S,lv) ->_1 a_1]

    Rule Unit:
    H; S, (a, A, (lv = (), {st})) -> H'; S, (a, A, {st}) where
        H' = H[a(H,S,lv) ->_1 0]

    Rule Swap:
    H; S, (a, A, (lv <-> lv', {st})) -> H'; S, (a, A, {st}) where
        Z = sizeof(type(lv))
        H' = H[a(H,S,lv) ->_Z a(H,S,lv'),
               a(H,S,lv') ->_Z a(H,S,lv)]

    Rule Call:
    H; S, (a, A, (id({x_a}), {st})) ->
    H'; S, (a, A, {st}), (A', {st'}) where
        args(id) = {x_f:ty_f}
        {a_f} = fresh range of addresses appropriate for {x_f:ty_f}
        locals(id) = {x_l:ty_l}
        {a_l} = fresh range of addresses appropriate for {x_l:ty_l}
        H' = H[{a_f} <- H(A({x_a})), {a_l} <- 0]
        A' = [{x_f: a_f}, {x_l: a_l}]
        {st'} = stmts(id)
               
## Type Rules

### Type-checking functions

A Rust program `rp` is said to type check if all of its 
constituent functions type check according to the following judgement:

    Ty-Fn:
      |- tys_f WF
      |- tys_l WF
      \env = xs_f:tys_f, xs_l:tys_l
      \env; [] |- sts
      ------------------------------------------------------------
      |- "fn" id(xs_f:tys_f) "{" let mut xs_l:tys_l; sts "}"
     
### Type-checking statement lists

A given statement type checks according to the judgement `\env; lns |- sts`.

    Ty-Stmts:
      ------------------------------------------------------------
      \env; lns |- [] 

    Ty-Stmts:
      \env; lns |- st -> \env'; lns'
      \env'; lns' |- sts
      ------------------------------------------------------------
      \env; lns |- st, sts

### Type-checking individual statements

A given statement type checks according to the judgement
`\env; lns |- sts -> \env'; lns'`.

    Ty-Stmt-Assign:
      \env |- lv : ty
      \env |- rv : ty'
      ty <: ty'
      \env; lns |- rv -> lns'
      \env; lns' |- Assignable(lv)
      ------------------------------------------------------------
      \env; lns |- lv = rv -> \env; lns'

    Ty-Stmt-Swap:
      \env |- lv : ty
      \env |- lv' : ty
      \env; lns |- Assignable(lv)
      \env; lns |- Assignable(lv')
      ------------------------------------------------------------
      \env; lns |- lv <-> lv' -> \env; lns

    Ty-Stmt-Call:
      formals(g) = rs_f tys_f
      \Theta = [rs_f -> rs_a]
      \forall i. rs_a(i) in \env
      \forall i. \env(xs(i)) <: \Theta(tys_f(i))
      ------------------------------------------------------------
      \env; lns |- g(xs) -> lns

### Type-checking rvalues

    Ty-Rv-Var:
      ------------------------------------------------------------
      \env; lns |- x -> lns

    Ty-Rv-Field:
      ------------------------------------------------------------
      \env; lns |- x.f -> lns

    Ty-Rv-Deref:
      ------------------------------------------------------------
      \env; lns |- *x -> lns

    Ty-Rv-Uniq:
      ------------------------------------------------------------
      \env; lns |- ~x -> lns

    Ty-Rv-Null:
      ------------------------------------------------------------
      \env; lns |- () -> lns

    Ty-Rv-AddrOf:
      r in Regions(\env)
      \env |- Mutability(lv) = m_lv
      \env |- Guarantee(lv, m, r) = lns_lv
      \forall i, j. Compatible(lns(i), lns_lv(j))
      ------------------------------------------------------------
      \env; lns |- &_r m lv -> lns, lns_lv

## Type subtyping

    ST-Unit:
      ------------------------------------------------------------
      () <: ()

    ST-Uniq:
      m ty <: m' ty'
      ------------------------------------------------------------
      ~ m ty <: ~ m' ty'

    ST-Rec:
      \forall i. ms(i) tys(i) <: ms'(i) tys'(i)
      ------------------------------------------------------------
      { ms fs: tys } <: { ms' fs: tys' }

    ST-Region:
      m ty <: m' ty'
      r <: r'
      ------------------------------------------------------------
      &_r m ty <: &_r' m' ty'

    MT-Mut:
      ------------------------------------------------------------
      mut ty <: mut ty

    MT-Imm:
      ty <: ty'
      ------------------------------------------------------------
      imm ty <: imm ty'

    MT-Const:
      ty <: ty'
      ------------------------------------------------------------
      m ty <: const ty'

### Loan compatibility

The compatibility check aims to guarantee that the same memory is not
loaned out as mutable and as immutable simultaneously.

    LC-Different-Memory:
      lv != lv'
      ------------------------------------------------------------
      Compatible(lv m r, lv' m' r')

    LC-Const:
      ------------------------------------------------------------
      Compatible(lv const r, lv' m' r')
      Compatible(lv m r, lv' const r')

    LC-Same:
      ------------------------------------------------------------
      Compatible(lv m r, lv m r')

## Borrow Rules

### Guarantee

The judgement `\env |- Guarantee(lv, m, r) = lns` means that a pointer
to `lv` with mutability `m` is guaranteed to be valid if the (possibly
empty) set of lns `lns` is honored.  This brings together the various
checks.

    G-Loan-Mut:
      \env |- Loan(lv, mut, r) = lns
      \env |- Mutability(lv) = mut
      ------------------------------------------------------------
      \env |- Guarantee(lv, mut, r) = lns

    G-Loan-Nonmut:
      \env |- Loan(lv, m, r) = lns
      m != mut
      ------------------------------------------------------------
      \env |- Guarantee(lv, m, r) = lns

    G-Stable-Mut:
      \env |- Stable(lv)
      \env |- Mutability(lv) = mut
      ------------------------------------------------------------
      \env |- Guarantee(lv, mut, r) = []

    G-Stable-Imm:
      \env |- Stable(lv)
      \env |- Mutability(lv) = imm
      ------------------------------------------------------------
      \env |- Guarantee(lv, imm, r) = []

    G-Stable-Const:
      \env |- Stable(lv)
      ------------------------------------------------------------
      \env |- Guarantee(lv, const, r) = []

### Mutability

Rules for computing the mutability of an lvalue.  The judgement is
`\env |- Mutability(lv) = m`:

    M-Local:
      \env(x) = m ty
      ------------------------------------------------------------
      \env |- Mutability(x) = m

    M-Field-Imm:
      \env |- Mutability(lv) = m
      \env |- lv : "{" ms fs:tys "}"
      ms(i) = imm
      ------------------------------------------------------------
      \env |- Mutability(lv.fs(i)) = m

    M-Field-Mut:
      \env |- lv : "{" ms fs:tys "}"
      ms(i) = mut
      ------------------------------------------------------------
      Mutability(lv.fs(i)) = mut

    M-Deref-Uniq:
      \env |- lv : ~ m ty
      ------------------------------------------------------------
      \env |- Mutability(*lv) = m

    M-Deref-Region:
      \env |- lv : &_r m ty
      ------------------------------------------------------------
      \env |- Mutability(*lv) = m

### Loan

Let a loan `ln` be defined as:

     ln = (lv, m, r)
     
This means that the lvalue `lv` is loaned out with the mutability `m`
for the region `r`.

Then we define a judgement `\env |- Loan(lv, m, r) = lns` which means
that, in the environment `\env`, we can safely create a reference to
the lvalue `lv` with the mutability `m` with duration `r` assuming
that the loans `lns` can be honored.

Note that these rules do not guarantee that the mutability of the
pointer matches the mutability of the data being pointed at.  This is
intentional, because we want to be able to temporarily treat mutable
data as immutable.  The typing rules will guarantee that immutable
data is never aliased with a mutable pointer.

#### Loaning variables

    L-Var:
      ------------------------------------------------------------
      \env |- Loan(x, m, r) = (x, m, r)

#### Loaning fields

    L-Field-MutConst:
      m != imm
      \env |- Loan(lv, const, r) = lns
      ------------------------------------------------------------
      \env |- Loan(lv.f, m, r) = lns, (lv.f, m, r)

    L-Field-Imm:
      \env |- Loan(lv, imm, r) = lns
      ------------------------------------------------------------
      \env |- Loan(lv.f, imm, r) = lns, (lv.f, imm, r)

#### Loaning derefences of unique pointers

    L-Deref-Mut:
      \env |- Loan(lv, imm, r) = lns
      \env |- lv : ~ m_lv ty_lv
      ------------------------------------------------------------
      \env |- Loan(*lv, m, r) = lns, (lv.f, m, r)

### Stable

The judgement `\env |- Stable(lv)` indicates that the lvalue `lv` is
stable without the need for any loans. Unlike the loan check, the
stability check only works on aliasable data.  Also, the resulting
pointer must be compatible with the mutability of the data being
pointed at, though this is not enforced here in these rules but rather
elsewhere.

    S-Field:
      \env |- Stable(lv)
      ------------------------------------------------------------
      \env |- Stable(lv.f)

    S-Deref-Mut:
      \env |- lv : &_r m_lv ty_lv
      ------------------------------------------------------------
      \env |- Stable(*lv)

    S-Deref-Uniq:
      \env |- lv : ~ imm ty_lv
      \env |- Stable(lv)
      \env |- Mutability(lv) = imm
      ------------------------------------------------------------
      \env |- Stable(*lv)

## Wellformedness checks

    WF-Ty-Unit:
      ------------------------------------------------------------
      \env |- () WF

    WF-Ty-Rptr:
      \env |- ty WF
      r in Regions(\env)
      ------------------------------------------------------------
      \env |- &_r m ty WF

    WF-Ty-Uptr:
      \env |- ty WF
      ------------------------------------------------------------
      \env |- ~ m ty WF

    WF-Ty-Uptr:
      \forall i. \env |- tys(i) WF
      ------------------------------------------------------------
      \env |- { ms fs:tys } WF


    
