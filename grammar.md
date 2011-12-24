## Grammar ##

    I = tag g<tvs> { Vs }                    (item)
      | fn i<tvs>(Ds xs: Ts) -> T { E }
    V = i(Ts)                                (variant)
    E = x                                    (expression)
      | E.f
      | i::<Ts>
      | E(Es)
      | *E
      | let x: T = E in E
      | { Ms fs: Es }
      | [ M Es ]
      | alt E { As }
      | { Es; E }
      | fn(Ds xs: Ts) -> T { E }
      | E = E
    A = P { E }                              (arm)
    P = x                                    (pattern)
      | i(Ps)
    T = S N T                                (type)
      | [N T]
      | { Ns fs: Ts }
      | fn(Ds Ts) -> T
      | g<Ts>
      | tv
      | ()
    D = +                                    (mode)
      | -
      | &
    M = mut                                  (mutability)
      | imm
    N = M                                    (mutability in types)
      | const
    S = @                                    (pointer sigil)
      | ~
    

