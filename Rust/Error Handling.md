### Error Handling
- In many cases Rust requires you to acknowledge possiblity of error and take some action before code will compile
- Ensures you discover errors and handle them before deploying to production
- Two types of errors: **recoverable** and **unrecoverable**
	- Recoverable, ex file not found, we most likely want to report problem to user and retry operation
	- Unrecoverable errors are symptoms of bugs like out of bounds array and we want to stop program
	- Rust doesn't have exceptions, instead it uses type `Result<T,E>` and `panic!` macro
#### Unrecoverable Errors With panic!
- Unwinding the stack is default for panics, but Rust allows you to immediately abort with `panic = 'abort'`

```rust
fn main() {
	panic!("crash and burn");
}
```

#### Using panic! Backtrace

```rust
fn main() {
	let v = vec![1, 2, 3];

	v[99];
}
```

- `panic!` call comes from library rather than calling macro directly

#### Recoverable Errors with Result
- Some errors you can catch and deal with instead of terminating program
- Recall the `Result<T,E>` enum

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

```rust
use std::fs::File;

fn main() {
	let f = File::open("hello.txt");

	let f = match f {
		Ok(file) => file,
		Err(error) => panic!("Problem opening the file: {:?}", error),
	};
}
```

- `Result` enum is brought into scope by prelude

#### Matching on Different Errors

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
	let f = File::open("hello.txt");

	let f = match f {
		Ok(file) => file,
		Err(error) => match error.kind() {
			ErrorKind::NotFound => match File::create("hello.txt") {
				Ok(fc) => fc,
				Err(e) => panic!("Problem creating file {:?}", e);
			}
			other_error => {
				panic!("Problem opening file: {:?}", other_error)
			}
		}
	}
}
```

#### Shortcuts to Panic on Error
- `match` is verbose and doesn't always communicate intent well
- `unwrap` method in Result is a shortcut method which calls `panic!` macro if the `Result` type is `Err` type otherwise wraps value in `Ok` type

```rust
fn main() {
	let f = File::open("hello.txt").unwrap(); // panics with unwrap panic method
}
```

- We can improve `panic` message by using `expect` to make it easier to track down error rather than going through several generic messages

```rust
fn main() {
	let f = File::open("hello.txt").expect("Failed to open hello.txt"); 
}
```

#### Propogating Errors

- "bubbling up" errors, which is returning the error to the calling code so it can decide what to do
	- Might have more information or logic in calling code to determine how to handle error 

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
	let f = File::open("hello.txt");

	let mut f = match f {
		Ok(file) => file,
		Err(e) => return Err(e),
	};

	let mut s = String::new();

	match f.read_to_string(&mut s){
		Ok(_) => Ok(s),
		Err(e) => Err(e),
	}
}
```

#### Shortcut For Propogating Errors: the ? Operator

```rust
fn read_username_from_file() -> Result<String, io::Error> {
	let mut f = File::open("hello.txt")?;
	let mut s = String::new();
	f.read_to_string(&mut s)?;
	Ok(s)
}
```

- `?` placed after `Result` is defined to work in same way as match expressions
	- if `Ok`, value inside is returned
	- if `Err`, Err will be return from whole function
	- difference is that `?` calls `from` which converts errors from one type to error type defined in function
- Chaining `?` could shorten a lot more code

```rust
File::open("hello.txt")?.read_to_string(&mut s)?;

fs::read_to_string("hello.txt"); // open, create String, read contents and return (common operation so it's included in std)
```

#### Where The ? Operator Can Be Used
- Only compatible with functions that return the appropriate value

```rust
use std::fs::File;

fn main() {
	let f = File::open("hello.txt")?; // compiler error- function should return Result or Option to accept ?
}
```

- Also compatible with `Option<T>` values `None` is returned early and value inside `Some` is returned and function continues
```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
	text.lines().next()?.chars().last()
}
```

- Notice that both won't be compatible together, in that case you will need to use methods like `ok` in `Result` or `ok_or` in `Option` to convert manually
- So far all `main` functions we've seen return `()` and there are restrictions on what its return type can be for programs to behave as expected
- `main` can return a `Result<(), E>` or any type that implements the `std::process:Termination` trait
```rust
fn main() -> Result<(), Box<dyn Error>> {
	let f = File::open("hello.txt");

	Ok(())
}
```

#### To Panic or Not to Panic

- When should I return `Result` or call `panic!`
	- Returning `result` gives calling code options 
		- Handle appropriate for situation
		- Or determine error as unrecoverable and call `panic!`
	- Using `panic!` is common in examples, prototype code, and tests
- `unwrap` and `expect` serve as placeholder until you make program more robust and figure out how to handle errors

#### Cases Which You Have More Information Than Compiler

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

- Appropriate to use unwrap here beacuse you know that `unwrap` will return `Ok` but compiler doesn't

#### Guidelines for Error Handling

- `panic!` when code could end up in a bad state
	- bad state is something that is unexpected, like user entering data in wrong format
	- code after this point needs to rely on not being in bad state rather than checking for every line
	- there's no good way to encode information in types used
- `panic!` is appropriate when calling external code that returns invalid state that you have no way of fixing
- functions have *contracts* - their behavior is only guaranteed if inputs meet particular requirements
- lots of Error checks in your functions is verbose and annoying, Rust type system solvess this
- ex. rather than a function accepting `Option` you can accept `Some` so that you don't have to handle both cases

#### Custom Types for Validation

```rust
loop {
	let guess: i32 = match guess.trim().parse() {
		Ok(num) => num,
		Err(_) => continue,
	}

	if guess < 1 || guess > 100 {
		println!("The secret number will be between 1 and 100");
		continue;
	}
}
```

- We could do a check like the above by parsing the guess as a `i32` and checking if it is within these ranges; however, if it was absolutely critical the program met these requirements having a check in every function would be tedious and might impact performance
- Instead we can make a new type and put validations in a function to create an instance rather than repeat validations everywhere

```rust
pub struct Guess {
	value: i32
}

impl Guess {
	pub fn new(value: i32) -> {
		if value < 1 || value > 100 {
			panic!("Guess value must be between 1 and 100");
		}
		Guess { value }
	}

	pub fn value(&self) -> i32 {
		self.value	
	}
}
```

- We implement a method named `self` which doesn't have any other parameters and returns `i32`
- Keeping value private is important because we shouldn't like code outside module set values directly
	- Must use `::new` function and pass required checks

