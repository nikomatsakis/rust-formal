## Grammar ##

    rv = x
       | x.f
       | *x
       | id({x})
       | &lv
       | ~x
       | ()
    lv = x
       | lv.f
       | *x
    st = lv = rv 
       | lv <-> lv
    fn = "fn" id({x:ty}) -> ty "{"
             let {x:ty};
             {st}
         "}"
    ty = "{" {f:ty} "}"
       | ~ty
       | &_r ty
       | ()
       
## Operational Semantics ##

Let the *heap* `H` be a map from an address `a` to an atomic value, which is
either another address or unit.  

Let the *stack* `S` be a list comprised of entries:

    S = []
      | S, (a, A, {st})
      
where `A: {x -> a}` is an *activation record*, which maps from the
local variable names in the stack frame to the addresses where their
values are stored.

Let `a(H,A,lv)` be a function that computes the address where a given
lvalue is stored:

    a(H,A,x) = A(x)
    a(H,A,lv.f) = a(H,S,lv) + offset(f)
    a(H,A,*x) = H(A(x))

A machine configuration `C` is then defined as:

    C = H; S
    
We assume that all local variables and fields in the program have
unique names, and define a helper function `type(x)` that returns the
type declared for a given variable and `type(f)` that returns the type
declared for a given field.  The rules are always defined relative to
a given fixed program text.
    
The small step semantics are defined by a relation `C -> C'`.

    Rule Var:
    H; S, (a, A, (lv = x, {st})) -> H'; S, (a, A, {st}) where
        Z = sizeof(type(x))
        H' = H[a(H,S,lv) ->_Z H(A(x))]

    Rule Field:
    H; S, (a, A, (lv = x.f, {st})) -> H'; S, (a, A, {st}) where
        Z = sizeof(type(f))
        H' = H[a(H,S,lv) ->_Z H(A(x) + offset(f))]

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
    H; S, (a, A, (lv = id({x_a}), {st})) ->
    H'; S, (a, A, {st}), (a(H,S,lv), A', {st'}) where
        args(id) = {x_f:ty_f}
        {a_f} = fresh range of addresses appropriate for {x_f:ty_f}
        locals(id) = {x_l:ty_l}
        {a_l} = fresh range of addresses appropriate for {x_l:ty_l}
        H' = H[{a_f} <- H(A({x_a})), {a_l} <- 0]
        A' = [{x_f -> a_f}, {x_l -> a_l}]
        {st'} = stmts(id)
               

        

               
