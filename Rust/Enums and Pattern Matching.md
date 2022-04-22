### Enums and Pattern Matching
- Defines type by enumarating its possible variants

#### Defining An Enum
- Usage with where only one variant is possible at a time and both variants are functionalty similar

```rust
enum IpAddrKind {
	V4,
	V6
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;

fn route(ip_kind: IpAddrKind) {}

fn main() {
	route(IpAddrKind::V4);
	route(six);
}
```

- Using this information about enums you might be tempted to tackle adding an IP address but using a struct like:

```rust
struct IpAddr {
	kind: IpAddrKind,
	address: String
}
```

- However representing the same concept with just an enum is more concise, rather than an enum in a struct

```rust
enum IpAddr {
	V4(String),
	V6(String)
}

let home = IpAddr::V4(String::from("127.0.0.1"));
```

- The advantage of using enums rather than structs in this case is that variants can have different types and amounts of associated data

```rust
enum IpAddr {
	V4(u8, u8, u8, u8),
	V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
```

- IP Addresses are so common there's a standard library for it that looks like the following which is not much more complicated than what you may have come up with

```rust
struct Ipv4Addr {

}

struct Ipv6Addr {

}

enum IpAddr {
	V4(Ipv4Addr),
	V6(Ipv6Addr)
}
```

- We could define a complex enum containing a variety of types as such
- We can also define methods on enums using `impl` like structs

```rust
enum Message {
	Quit,
	Move {x: i32, y: i32},
	Write (String),
	ChangeColor(i32,i32,i32),
}

impl Message {
	fn call(&self) {
	}
}

let m = Message::Write(String::from("hello"));
m.call();
```

#### Option Enum and Its Advantages Over Null Values

- `Option` type encodes very common scenario which value could be something or nothing
- Many languages handle this with `null`, where variables can either be null or not-null
- The issue with this is that when you try to use a null value as a not-null value, you'll get an error 
	- Extremely easy to get this error
- Concept isn't the issue but the implementation
- Rust handles this through an enum that encodes the concept of presence or absence `Option<T>`

```rust
enum Option<T> {
	None,
	Some(T),
}

let some_number = Some(5); // Option<i32>
let some_string = Some("a string"); // Rust infers generic type by value variant is holding

let absent_number: Option<i32> = None; // Rust can't infer so must explicitly state
```

- `Option<T> and T` are different types so compiler won't let us use `Option<T>` as if it were definitely a valid value

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y; // compiler error - no implementation for `i8 + Option<i8>`
```

- A value of `T` Rust compiler will ensure we always have valid value
- Only when we have `Option<T>` we have to handle case of not having a value
- You have to convert `Option<T>` to a `T` before you can perform `T` operations
	- Helps catch one of the most common issues with assumptions with null
	- Increases confidence in code beause you have to explicitly opt into type `Option<T>`
- To handle the conversion and handle both cases `Some` and `None`, `Option<T>` has a number of methods which you can use with `match`

#### Match Control Flow Construct

- `match` is similar to `switch` in other languages
- Code associated with each arm is an expression, resulting value is returned from entire `match` expression

```rust
enum Coin {
	Penny,
	Nickel,
	Dime, 
	Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => {
			println!("Lucky penny!");
			1
		},
		Coin::Nickel => 5,
		Coin::Dime => 10,
		Coin::Quarter => 25, 
	}
}
```

#### Patterns That Bind To Values
- Another userful feature of match arms is that they can bind to parts of values that match the pattern, so we can extract values from enum variants
- If we were to call `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` would be `Coin::Quarter(UsState::Alaska)` and `state` would be `UsState::Alaska`

```rust
#[derive(Debug)]
enum UsState {
	Alabama,
	Alaska,
}

enum Coin {
	Penny,
	Nickel,
	Dime, 
	Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => {
			println!("Lucky penny!");
			1
		},
		Coin::Nickel => 5,
		Coin::Dime => 10,
		Coin::Quarter(state) => {
			println!("State quarter from {:?}!", state);
			25
		}, 
	}
}
```

#### Matching with Option

- A function to handle `Some` and `None` case for `Option` looks like

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
	match x {
		None => None,
		Some(i) => Some(i + 1),
	}
} 

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

- Matches in Rust are *exhaustive*, we must exhaust every last possiblity (variant) in order for code to be valid

```rust
match x {
	Some(i) => Some(i + 1), // compiler error
}
```

#### Catch-All Patterns and the _ Placeholder

- use `other` when we want to consume the value and `_` when we don't want to

```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	other => move_player(other), // Catch-all pattern that uses variable
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
```


```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	_ => (), // Pattern when we don't want to use value
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
```

#### Concise Control Flow with if let
- `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that match one pattern while ignoring rest
- The following are equivalent functions

```rust
let config_max = Some(3u8);
match config_max {
	Some(max) => println!("The maximum is configured to be {}", max),
	_ => (), // required to exhaust variants and annoying to add if we don't want to do anything
}
```

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
	println!("The maximum is configured to be {}", max);
}
```

- Think of `if let` as a syntax sugar for a `match` that runs code when value matches one pattern then ignores all others
- We could also do the following

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
	println!("State quarter from {:?}!", state);
} else {
	count += 1;
}
```

