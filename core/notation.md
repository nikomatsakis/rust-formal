# Typographical Conventions #

- Uppercase letters indicate nonterminals
- Lowercase letters indicate terminals:
  - `f` is a field name
  - `x` is a variable name
  - `i` is an item name
  - `g` is a tag name
  - `tv` is a type variable name
- A trailing s (as in `fs`) indicates a list
- Adjacent lists or lists separated by `:` indicate lists of tuples.
  So `Ds Ts` is a list `D1 T1, D2 T2, D3 T3` and so forth, not `D1 D2
  D3, T1 T2 T3`.
- Lists are functions from index to item, so `fs(j)` indicates the 
  `j`th member of the list `fs`

