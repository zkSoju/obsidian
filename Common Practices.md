## Optimization

Over-optimization is **bad.** 

Packing together variables will save gas in situations where the variables are commonly accessed together.

* However in this [ ERC721A pull request](https://github.com/chiru-labs/ERC721A/pull/109)  `_currentSupply` and `_burnCounter` were given type `uint128` to allow compiler to pack together
* Both variables were more commonly accessed separately
* Only one case was this setup justifiable in `totalSupply()` where both variables were and accessed and calculation performed

## Gotchas 

- Unchecked 