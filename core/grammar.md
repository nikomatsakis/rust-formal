## Grammar ##

    I = tag g<tvs> { Vs }                    (item)
      | fn i<Ks tvs>(Ds xs: Ts) -> T { let xs:Ts; E }
    V = i(Ts)                                (variant)
    E = x                                    (expression)
      | E.f
      | i::<Ts>
      | E(Es)
      | S E
      | *E
      | { Ms fs: Es }
      | alt E { As }
      | fn(Ds xs: Ts) -> T { let xs:Ts; E }
      | E = E
      | E; E
    A = P { E }                              (arm)
    P = x                                    (pattern)
      | i(Ps)
    T = S N T                                (type)
      | { Ns fs: Ts }
      | fn(Ds Ts) -> T
      | g<Ts>
      | tv
      | ()
    D = ++                                   (mode)
      | +
      | -
      | &
      | &&
    M = mut                                  (mutability)
      | imm
    N = M                                    (mutability in types)
      | const
    S = @                                    (pointer sigil)
      | ~
    K = send
      | copy
      | move

