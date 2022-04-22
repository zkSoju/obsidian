## Programming
- rust variables are immutable by default, use the mut declaration to make mutable (`let mut guess = 5` vs `let guess = 5`)
- `::` in `String::new()` indicates that the *associated function* is being called on String type
	- creates an empty instance of String
- without importing the io library you can call `std::io::stdin()` , but when importing you can call `io::stdin()`
- `&` indicates that variable is a reference, allowing you to access the same piece of data without copying into memory multiple times
- `read_line()` returns a `io::Result` which is an enum, commonly used with `match` a conditional which runs different code based on enum value
- `io::Result` has a `expect` function that can be called to catch the error and run code based on `Err` or `Ok`
	- if `expect` is not used, program will compile, but warning will be given
- string placeholders `println!("You guessed: {}", guess);` 
- library crates like `rand` can't be executed on its own, meant to be used in other programs
- version `0.8.3` is shorthand for `^0.8.3` which means any version at least `0.8.3` but below `0.9.0`
- `cargo.lock` locks dependency versions to ones used during build time
- `cargo update` ignores cargo.lock file and figures out latest version that fit specifications in `cargo.toml`
- importing `Rng` trait is needed to call `gen_range`
- `1..101` and `1..=100` are range expressions that are equivalent 
- you won't know which traits to use and which methods/function to call, so use `cargo doc --open`
- `std::cmp:Ordering` used to compare two values and returns `Less`, `Greater`, `Equal` as variants of enum `Ordering`
- Rust is a strong, static type system with some type inference
	- rust is able to infer guess as a string in `let mut guess = String::new()`
	- with number types, there are a few types that have a value between 1 and 100
	- rust defaults to `i32`
	- compiler error comes from comparing a string with an i32
- Rust allows you to shadow previous variables with new ones with the same name and any type
- `let guess: u32 = guess.trim().parse().expect("Please type a number!")`
- `trim()` eliminates white space and `\n` new line character
- `parse()` converts string to a number type specified
- Since `guess` is of type `u32` Rust infers `secret_number` to be `u32` as well when doing the comparison
- `loop {}` keyword creates an infinite loop
- `break` to exit loop

```rust
let guess: u32 = match guess.trim().parse() {
	Ok(num) => num,
	Err(_) => continue,
};
```

- `parse` returns Ok value with resulting number
- `_` is a catchall value, in this case it's to match all `Err` values no matter the information inside
[Variables](Variables.md)

[Data Types](Data%20Types.md)

[Ownership](Ownership.md)

[Structs and Structure Related Data](Structs%20and%20Structure%20Related%20Data.md)

[Enums and Pattern Matching](Enums%20and%20Pattern%20Matching.md)

[Managing Growing Projects with Packages, Crates, and Modules](Managing%20Growing%20Projects%20with%20Packages,%20Crates,%20and%20Modules.md)

[Common Collections](Common%20Collections.md)

[Error Handling](Error%20Handling.md)

[Generic Types Traits, And Lifetimes](Generic%20Types%20Traits,%20And%20Lifetimes.md)

[Writing Automated Tests](Writing%20Automated%20Tests.md)