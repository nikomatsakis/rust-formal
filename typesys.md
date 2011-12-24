## Additional grammar items ##

An environment `Env` is a function `x->T` mapping variable names to types.

A capability set `C` is a set of variable names.

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
    
## Typing expressions ##

The judgement `Env |- x : T`

    E-Var:
        ------------------------------------------------------------
        Env |- x : T

    E-Field:
        Env |- E : { Ms fs: Ts }
        ------------------------------------------------------------
        Env |- E.fs(i) : Ts(i)

    E-FnItem:
        fn i<tvs>(Ds xs: Ts) -> T
        % = [tvs -> Ts]
        ------------------------------------------------------------
        Env |- i::<Ts> : fn(Ds %Ts) -> %T

    E-TagItem:
        tag g<tvs> { Vs }
        Vs(j) = i(Ts)
        % = [tvs -> Ts]
        ------------------------------------------------------------
        Env |- i::<Ts> : fn(Ds %Ts) -> g<Ts>
        
    E-Call:
        Env |- Es_a : Ts_a
        Env |- E : fn(Ds Ts_f) -> T_r
        Env |- Ts_a <: Ts_f
        ------------------------------------------------------------
        Env |- E(Es_a) : T_r
        
    E-Deref:
        Env |- E : S N T
        ------------------------------------------------------------
        Env |- *E : T
        
    E-Let:
        Env |- E_1 : T_1
        Env |- T_1 <: T
        Env, (x: T) |- E_2 : T_2
        ------------------------------------------------------------
        Env |- let x: T = E_1 in E_2 : T_2
        
    E-Rec:
        Env |- Es : Ts
        ------------------------------------------------------------
        Env |- { Ms fs: Es } : { Ms fs: Ts }
         
    E-Vec:
        forall j. Env |- Es(j) : T
        ------------------------------------------------------------
        Env |- [ M Es ] : [ M T ]
         
    E-Alt:
        Env |- E : T_1
        Env |- T_1 @ As : T (forall)
        ------------------------------------------------------------
        Env |- alt E { As } : T

    A:
        Env |- T_1 @ P : Env_1
        Env_1 |- E : T
        ------------------------------------------------------------
        Env |- P { E } : T
        
    E-Block:
        Env |- Es : Ts
        Env |- E : T
        ------------------------------------------------------------
        Env |- { Es; E } : T

    E-Lambda:
        Env, (xs: Ts) |- E : T_1
        Env |- T_1 <: T
        ------------------------------------------------------------
        Env |- fn(Ds xs: Ts) -> T { E } : fn(Ds Ts) -> T

    E-Assign:
        Env |- lv(E) : T_1
        Env |- E : T_2
        Env |- T-1 <: T_2
        ------------------------------------------------------------
        Env |- E = E : ()

## Pattern Matching ##

    P-Var:
        ------------------------------------------------------------
        Env |- T @ x : Env[x: T]

    P-Tag:
        tag g<tvs> { Vs }
        Vs(j) = i(Ts_v)
        % = [tvs -> Ts]
        Env |- %Ts_v @ Ps : Env_2 (chained)
        ------------------------------------------------------------
        Env |- g<Ts> @ i(Ps) : Env_2
        
