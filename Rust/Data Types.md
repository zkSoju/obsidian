### Data Types

There are two data type subsets: scalar and compound. The compiler can usually infer what type we want to use, but when multiple types are possible we must explicitly annotate the type.

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

#### Scalar Types

##### Integer Types

A scalar type represents a single value. Rust has 4 primary scalar types: integers, floating-point numbers, Booleans, and characters.

| Length  | Signed | Unsigned |
| ------- | ------ | -------- |
| 8-bit   | i8     | u8       |
| 16-bit  | i16    | u16      |
| 32-bit  | i32    | u32      |
| 64-bit  | i64    | 864      |
| 128-bit | i128   | u128     |
| arch    | isize  | usize    | 

You can use `_` as a visual separator to make numbers easier to read like `1_000`. Defaults are a good place to start if you don't know what type to use.

Integer overflow is possible. Rust compiler will include checks for overflow that cause program to panic if the behavior occurs. When compiling, Rust does not include checks instead Rust performs *two's complement wrapping*. For example, a `u8` type value 256 would become 0 and 257 would become 1.

`isize` and `usize` are 32-bit or 64-bit depending on PC architecture. 

To handle overflow families of methods are provided:
- `wrapping_*`
- return `none` if overflow with `checked_*` methods
- return value and boolean indicating overflow with `overflowing_*` methods
- saturate at value's min/max values with `saturating_*` methods

##### Floating Point Types

- `f32` and `f64` (defaults to f64 because modern CPUs is roughly same speed)
- all floating point numbers are signed

```rust
fn main() {
	let x = 2.0;
	let y: f32 = 3.0;
}
```

##### Boolean Types

##### Character Types

#### Compound Types

#### Tuple Type
- A tuple is a way of grouping together a number of values with a variety of types into one compound type
- Use pattern matching to destructure tuple value or use a period followed by the index of value

```rust
fn main() {
	let tup: (i32, f64, u8) = (500, 6.4, 1);
	let (x,y,z) = tup;
}
```

```rust
fn main() {
	let tup: (i32, f64, u8) = (500, 6.4, 1);
	let five_hundred = tup.0;
	let six_point_four = tup.1;
	let one = x.2;
}
```

#### Array Type
- Allocated to stack rather than heap
- Fixed number of elements
- Not as flexible as vector type, which is allowed to grow or shrink
- Useful when number of elements will not need change

```rust
let a = [1,2,3,4,5];
let a: [i32; 5] = [1,2,3,4,5];
let a = [3; 5] // [3,3,3,3,3]
```

- Accessing an out of bounds index is a runtime error because Rust can't know what you are trying to access in advance
- This error is an example of a memory safety principle preventing invalid memory access from occuring 

 ### Functions

#### Statements and Expressions
- Statements are instructions that perform an action and don't return a value
- Expressions evaluate to a resulting value
	- Calling a function
	- Calling a macro
	- New scope blocks created with curly brackets

```rust
fn main(){
	let y = 6; // statement
	let x = (let y = 6); // invalid because statement
}
```

```rust
fn main() {
	let y = {
		let x = 3;
		x + 1 // expressions have no semicolon (return statement)
	}
}
```

#### Functions with return values
- In Rust, we don't name return values, but we must declare their type after an arrow

```rust
fn plus_one(x: i32) -> i32 {
	x + 1
}

fn main() {
	let x = plus_one(5);
}
```

### Control Flow

#### If Statements
- Rust does not automatically convert non-Boolean types to a Boolean

```rust
fn main() {
	if number { // error
	}
	
	if number != 0 {
	}
}
```

- Using too many else if expressions can clutter code, you may want to use `match` in thse cases

#### Using If in a let Statement
- Because if is an expression, we can use it in the right side of a let statement to assign outcome as variable
```rust
fn main() {
	let condition = true;
	let number = if condition {5} else {6};
}
```

### Repetition with Loops
- 3 kinds of loops: `loop`, `while`, and `for`
- `continue` to skip over remaining code and continue to next iteration
- `break` to exit out of the loop
```rust
fn main() {
	loop {
		println!("again!")
	}
}
```
- optional loop labels can be specified so we can choose which loop to break out of
```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```
- we can return a value from a loop like so
```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

- we can implement every loop like this; however, the pattern is so common that Rust has a built in `while` 
- `for in` is also available and used as such, which provides safety of code and eliminates chance of bugs
```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {}", element);
    }
}
```

- usage of a `for in` loop with range and a `rev` method
```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```