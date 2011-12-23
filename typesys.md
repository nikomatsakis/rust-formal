## Grammar ##

    P = Is
    I = tag g<tvs> { Vs }
      | fn x<vns>(Ds xs: Ts) -> T { E }
    V = x(Es)
    E = x
      | E.fn
      | E::<Ts>(Es)
      | let x = E in E
      | { Ms fns: Es }
      | [ M Es ]
      | x::<Ts>(Es)
      | alt E { As }
      | { Es; E }
      | fn(Ds xs: Ts) -> T { E }
    A = P { E }
    P = x
      | x::<Ts>(Ps)
    T = S N T
      | [N T]
      | { Ns fns: Ts }
      | fn(Ds Ts) -> T
      | gn
      | tv
    D = +
      | -
      | &
    M = mut
      | imm
    N = M
      | const
    S = @
      | ~

## Type Rules ##

    Nsub-equiv:
        ------------------------------------------------------------
        N <= N

    Nsub-const:
        ------------------------------------------------------------
        const <= N

    Tsub-equiv:
        ------------------------------------------------------------
        Env |- T <: T

    Tsub-mut-@:
        N_1 <= N_2
        Env |- T_1 <: T_2
        ------------------------------------------------------------
        Env |- @ N_1 T_1 <: @ N_2 T_2
    
## ##
