### Managing Growing Projects with Packages, Crates, and Modules

- Organizing code by splitting into multiple modules and then multiple files is important when writing large programs
- Package can contain multiple binary crates and optionally one library crate
- As package grows, you can extract parts into separate crates as external dependencies
- Encapsulate implementation details so you can reuse code
- Module system
	- **Packages** - cargo feature that lets you build, test, and share crates
	- **Crates** - tree of modules that produces a library or exe
	- **Module** - controls organization, scope, and privacy of paths
	- **Path** - a way of naming an item (struct, function, module)

#### Packages and Crates
- Crate is a binary or library
- Crate root is a source file that the Rust compiler starts from
- Package is one or more crates 
	- Contains *cargo.toml* that describes how to build crates
	- Created with `cargo new`
	- `src/main.rs` and `src/lib.rs` are crate roots
- Crate's functionality is scoped to prevent potential conflicts
	- ex. the struct `Rng` will not conflict rand's `rand::Rng`

#### Defining Modules to Control Scope and Privacy
- `use` keyword brings path into scope
- `pub` keyword to make items public
- *modules* let us organize code within a crate into groups for readability, reuse, and privacy
- `mod` keyword with name of module defines a module
- used to group related definitions together and name why they're related

```rust
mod serving {

}
```

- crate roots form a module named a crate at the root forming a *module tree*

```rust
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

#### Paths for Referring to an Item in the Module Tree

- To show Rust where to find an item in module tree, we use a path similar to filesystem navigation
- A path can take two forms
	- Absolute path - starts from crate root by using literal `crate`
	- Relative path - starts from current module using `self`, `super`, or an identifier in current module
- `::` separates path identifiers

```rust
mod front_of_house {
	mod hosting {
		fn add_to_waitlist() {}
	}
}

pub fn eat_at_restaurant() { // marked as pub because function is part of library crate's public API
	crate::front_of_house::hosting::add_to_waitlist();

	front_of_house::hosting::add_to_waitlist();
}
```

- Choosing whether to use a relative path or absolute path is based on whether you're more likely to move item definite separately from or together with code 
- The above still doesn't compile because hosting is a private module
- Hiding inner implementation details is default that way you can modify inner code without breaking outer code
- Use `pub` to make module public; however, contents are still private

```rust
mod front_of_house {
	pub mod hosting { 
		fn add_to_waitlist() {} // this won't compile - private function
	}
}
```

- Siblings or children of the same module can reference each other without `pub`
	- ex. `eat_at_restaurant` and `front_of_house`

#### Starting Relative Paths with Super
- We can also construct relative paths with `super` similar to `..` syntax with filesystems
- Good for when we think modules will stay together, so we have less places to update code

```rust
fn serve_order() {}

mod back_of_house {
	fn fix_incorrect_order() {
		cook_order();
		super::serve_order();
	}

	fn cook_order() {}
}
```

#### Making Structs and Enums Public
- Note that making a struct `pub` doesn't making the fields public as well
- We can make each field public or not on a case-by-case basis

```rust
mod back_of_house {
	pub struct Breakfast {
		pub toast: String,
		seasonal_fruit: String,
	}

	impl Breakfast {
		pub fn summer(toast: &str) -> {
			Breakfast {
				toast: String::from(toast),
				seasonal_fruit: String::from("peaches")
			}
		}
	}
}

pub fn eat_at_restaurant() {
	let mut meal = back_of_house::Breakfast::summer("Rye");

	meal.toast = String::from("Wheat")
}
```

- Example case where customer can pick type of bread they want but only chef can decide fruit
- `pub toast` allows `eat_at_restaurant` to write and read to `toast`
- Setting an enum as `pub` sets all of its variants to public

```rust
mod back_of_house {
	pub enum Appetizer {
		Soup,
		Salad,
	}
}

pub fn eat_at_restaurant() {
	let order1 = back_of_house::Appetizer::Soup;
	let order2 = back_of_house::Appetizer::Salad;
}
```

- Difference is that structs are often useful without access to their fields while enums aren't without their variants

#### Bringing Paths into Scope with Use

- Inconviently long paths can be solved with `use` to bring in modules into scope
- You can use relative paths as well as absolute paths

```rust
mod front_of_house {
	pub mod hosting {
		pub fn add_to_waitlist() {}
	}
}

use crate::from_of_house::hosting; // absolute path
use self::from_of_house::hosting; // relative path

pub fn eat_at_restaurant() {
	hosting::add_to_waitlist();
}
```

- While you could specify the `use` path all the way out to `add_to_waitlist()` and just call `add_to_waitlist()`, specifying the parent module makes it clear that function isn't locally defined
- However, bringing in structs, enums, and other items it's appropriate to bring in the full path

```rust
use std::collections::Hashmap;

fn main() {
	let mut map = HashMap::new();
	map.insert(1,2);
}
```

- Bringing two types with same name into same scope requires parent modules

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {

}

fn function2() -> io:Result<()> {

}
```

- Providing new names with the as keyword

```rust
use std::io::Result as IoResult;
```

- Re-exporting names with `pub use` so that any code from new scope can use name
- Useful for when internal structure of code is different from how programmers calling your code would think about the domain
	- Write code with one structure but expose a different structure

```rust
pub use crate::front_of_house::hosting;
```

#### Using External Packages

- Add name from crates.io as dependency to `cargo.toml` then add `use` line starting with name of crate

```rust
rand = "0.8.3"
```

```rust
fn main() {
	let secret_number = rand::thread_rng().gen_range(1..101);
}
```

- `std` (Standard library) ships with the Rust language so you don't need to change `cargo.toml`

#### Using Nested Paths to Clean Up Large use Lists

```rust
use std::cmp::Ordering;
use std::io;
```

```rust
use std::{cmp::Ordering, io};
```

```rust
use std::io;
use std::io::Write;
```

```rust
use std::{self, Write};
```

#### Glob Operator

- if we want to bring all public items defined in path into scope
```rust
use std::collections::*;
```

#### Separating Modules into Different Files
- Previous examples defined multiple modules in one file; however, once modules get large, you might want to move definitiions into separate files
- Using a semicolon after `mod` tells Rust to load contents of module from another file with same name as module
- Extract hosting module

`src/lib.rs`
```rust
mod front_of_house; // declare module whose body is in another file

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
	hosting::add_to_waitlist();
}
```

`src/front_of_house.rs`
```rust
pub mod hosting {
	pub fn add_to_waitlist() {}
}
```
