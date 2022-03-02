## Keywords 
- `virtual` indicates that child class can `override` the function
- `override` overrides the `virtual` function of the parent
- `immutable` can never be modified after contract construction but can still be set at construction within the constructor
- `constant` can never be modified after contract construction but can only be set at compilation
- both `immutable` and `constant` are inlined in contract bytecode saving significant gas compared to storage variables

## Cross Contract calls
- `call()`
- `delegatecall()`
- `staticcall()`

## Libraries 
- **linked libraries** are deployed separately on the blockchain and address of library is supplied to linked contract
- **embedded libraries** are compiled within the contract

