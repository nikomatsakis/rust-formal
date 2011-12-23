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

