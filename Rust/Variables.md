### Variables

#### Immutable/Mutable

#### Constants

Constants are similar to immutable in that it value is bound to a name and is not allowed to change. The difference is that you can't use mut with constants, they are always immutable. They can be declared in any scope  and can only be defined by constant expression
Shadowing

#### Shadowing

First variable can be *shadowed* by second meaning the second is what the program sees when variable is used using `let`.

You can use inner shadowing to scope shadow.

```rust
let x = 5;
let x = x + 1;

{
	let x = x * 2;
}
```

Shadowing is different from `mut` because variable will be immutable after transformations are applied with `let`.

Without `let` and `mut` we will get a compile time error. Shadowing is basically creating a new variable where you can define any type and allows us to reuse the same name.

```rust
let spaces = "   ";
let spaces = spaces.len();
``` 

```rust
let mut spaces = "   ";
spaces = spaces.len(); // this will result in a compiler error because you're not able to mutate variable type
``` 