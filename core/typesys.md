## Additional grammar items ##

An environment `Env` is a function `x->T` mapping variable names to types.

A predicate `R = l(E*)` represents a label `l` applied to the path
`E`.  The labels `l` used in the core of Rust are `init`, `!init`.

A type substitution `%: tv->T` maps from type variables to types.

## Subtyping ##

    Nsub-equiv:
        ------------------------------------------------------------
        Env |- N <= N

    Nsub-const:
        ------------------------------------------------------------
        Env |- const <= N

    Tsub-equiv:
        ------------------------------------------------------------
        Env |- T <: T

    Tsub-mut-@:
        N_1 <= N_2
        Env |- T_1 <: T_2
        ------------------------------------------------------------
        Env |- @ N_1 T_1 <: @ N_2 T_2

## Type kinding ##

    K-Move:
        ------------------------------------------------------------
        Env |- T : move

    (omitting tedious rules)
    
## Typing expressions ##

The judgement `Env; Rs |- E : T; Rs_1` asserts that `E` is well-typed
in the environment `Env` given that the predicates `Rs` hold.  The
type of the expression is `T` and the set of predictes which hold
after the expression is `Rs_1`.  

To type a series of expressions `Es`, we write `Env; Rs |- Es : Ts;
Rs_N`, which is shorthand for a series of type judgements like:

    Env; Rs |- Es(1) : Ts(1); Rs_1
    Env; Rs_1 |- Es(2) : Ts(2); Rs_2
    ...
    Env; Rs_2 |- Es(N) : Ts(N); Rs_N
    
Note that the predicates resulting from typing each expression `Es(j)`
are used as the starting predicates for typing the next expression
`Es(j+1)`.  `Rs_N` is the final resulting set of predicates and `Ts`
is the list of types from each expression in `Es`.

    E-Var:
        init(x) in Rs
        ------------------------------------------------------------
        Env; Rs |- x : Env(x)

    E-Field:
        Env; Rs |- E : { Ms fs: Ts }
        ------------------------------------------------------------
        Env; Rs |- E.fs(i) : Ts(i); Rs

    E-FnItem:
        fn i<Ks tvs>(Ds xs: Ts) -> T
        % = [tvs -> Ts]
        Env; Rs |- Ts : Ks (forall)
        ------------------------------------------------------------
        Env; Rs |- i::<Ts> : fn(Ds %Ts) -> %T

    E-TagItem:
        tag g<tvs> { Vs }
        Vs(j) = i(Ts)
        % = [tvs -> Ts]
        ------------------------------------------------------------
        Env; Rs |- i::<Ts> : fn(Ds %Ts) -> g<Ts>
        
    E-Call:
        Env; Rs |- E : fn(Ds Ts_f) -> T_r; Rs_f
        Env; Rs_f |- Ds Es_a : Ts_a; Rs_a
        forall j. Env |- Ts_a(j) <: Ts_f(j)
        ------------------------------------------------------------
        Env; Rs |- E(Es_a) : T_r; Rs_a
        
    E-Ptr:
        Env; Rs |- E : T; Rs_1
        Env |- T : copy
        ------------------------------------------------------------
        Env; Rs |- S E : S T; Rs_1
        
    E-Deref:
        Env; Rs |- E : S N T; Rs_1
        ------------------------------------------------------------
        Env; Rs |- *E : T; Rs_1
        
    E-Rec:
        Env; Rs |- Es : Ts; Rs_1
        ------------------------------------------------------------
        Env; Rs |- { Ms fs: Es } : { Ms fs: Ts }; Rs_1
         
    E-Alt:
        Env; Rs |- E : T_1
        forall A in As. Env; Rs |- T_1 @ A : T; Rs
        ------------------------------------------------------------
        Env; Rs |- alt E { As } : T; Rs

    A:
        Env; Rs |- T_1 @ P : Env_1
        Env_1; Rs |- E : T; Rs_2
        Rs_3 <= Rs_2
        ------------------------------------------------------------
        Env; Rs |- T_1 @ P { E } : T; Rs_3
        
    E-Lambda:
        Env[xs->Ts, xs_v->Ts_v]; Rs |- E : T_1; Rs_1
        Env; Rs |- T_1 <: T
        Env; Rs |- Env(FV(E)) : copy
        ------------------------------------------------------------
        Env; Rs |- fn(Ds xs: Ts) -> T { let xs_v:Ts_v; E } : fn(Ds Ts) -> T; Rs

    E-Assign:
        Env; Rs |- lv(E_1) : T_1; Rs_1; Rs_2
        Env; Rs |- E_2 : T_2
        Env; Rs |- T-1 <: T_2
        // TODO: Filter out those affected by a write to E_1
        ------------------------------------------------------------
        Env; Rs |- E_1 = E_2 : (); Rs_1, Rs_2

## Modes ##

    M-ByValue:
        Env; Rs |- E : T; Rs_1
        ------------------------------------------------------------
        Env; Rs |- ++ E : T; Rs_1

    M-ByImmRef:
        Env; Rs |- E : T; Rs_1
        ------------------------------------------------------------
        Env; Rs |- && E : T; Rs_1

    M-ByCopy:
        Env; Rs |- E : T; Rs_1
        ------------------------------------------------------------
        Env; Rs |- + E : T; Rs_1

    M-ByMove:
        Env; Rs |- x : T; Rs_1
        Rs_2 = Rs_1 \ init(x)
        ------------------------------------------------------------
        Env; Rs |- - x : T; Rs_2

    M-ByMutRef:
        Env; Rs |- lv(E) : T; Rs_1
        // TODO: Filter out those affected by a write to E
        ------------------------------------------------------------
        Env; Rs |- & E : T; Rs_1

## Lvalues ##

    LV-Var:
        ------------------------------------------------------------
        Env; Rs |- lv(x) : Env(x); Rs; init(x)

    LV-Field:
        Env; Rs |- E : { Ns fs: Ts }; Rs_1
        Ns(j) = mut
        ------------------------------------------------------------
        Env; Rs |- lv(E.fs(j)) : Ts(j); Rs_1; ()

    LV-Star:
        Env; Rs |- E : S mut T; Rs_1
        ------------------------------------------------------------
        Env; Rs |- lv(*E) : T; Rs_1; ()

## Pattern Matching ##

    P-Var:
        ------------------------------------------------------------
        Env; Rs |- T @ x : Env[x: T]; Rs, init(x)

    P-Tag:
        tag g<tvs> { Vs }
        Vs(j) = i(Ts_v)
        % = [tvs -> Ts]
        Env; Rs |- %Ts_v @ Ps : Env_2; Rs_2 (chained)
        ------------------------------------------------------------
        Env; Rs |- g<Ts> @ i(Ps) : Env_2; Rs_2
        
