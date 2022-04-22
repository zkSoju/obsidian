### Structs and Structure Related Data
- struct is a custom data type that lets you package together and name multiple related values that make up a meaningful group
- Defining, instantiating, and modifying structs
```rust
struct User {
	active: bool,
	username: String,
	email: String,
	sign_in_count: u64,
}

fn main() {
	let mut user1 = User { // entire struct must be immutable
		email: String::from("someone@example.com"),
		username: String::from("someusername123"),
		active: true,
		sign_in_count: 1,
	};

	user1.email = String::from("anotheremail@example.com");

	let user2 = User { // struct update syntax, remaining fields not explicity set should have the same value as field in given instance
		email: String::from("another@example.com"),
		..user1, // this must come last to specify any remaining fields should get values from user1
	}
}

fn build_user(email: String, username: String) -> User {
	User {
		email, // field init shorthand when parameter names are same as struct field names
		username,
		active: true,
		sign_in_count: 1,
	}
}
```

- Note that the `=` symbol means data is being moved, so for `String` type the value from `user1` is moved to `user2` making `user1` invalid; however, for values like `active` and `sign_in_count` their types implement `Copy` trait

#### Tuple Structs
- Look similar to tuples, but have the added meaning the struct name provides without names for fields
- Useful for giving the whole tuple a name and making tuples a different type from one another

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
	let black = Color(0,0,0);
	let origin = Point(0,0,0);
}
```

- Note that Color and Point are different types even though fields within each struct have the same type
	- Example: Useful for functions that take in type Color but not Point

 #### Unit-Like Structs Without Any Fields
 - We can define structs without fields because they behave like `()` tuples
 - Useful for implementing a trait on some type without data stored in type

```rust
struct AlwaysEqual;

fn main() {
	let subject = AlwaysEqual;
}
```

- Fields -> Tuples -> Structs (add more structure and meaning to combination of fields)
- Note: to maintain ownership of struct when passing into function we need to use `&` in function signature and parameter
- We can add `#[derive(Debug)]` to opt into making debug information available for struct

```rust
#[derive(Debug)]
struct Rectangle {
	width: u32,
	height: u32,
}

fn main() {
	let rect1 = Rectangle {
		width: 30,
		height: 50,
	};

	println!("rect1 is {:?}", rect1);
	println!("rect1 is {:#?}", rect1); // more readable for larger structs
}
```

- We can also used the `dbg!` macro to print to the standard error console stream
	- Note: `dbg!` is a function, so use references with non-static types so that the function doesn't take ownership 

#### Method Syntax

- Methods are similar to functions; however, they are functions defined within the context of a struct (or enum or trait object) and their first parameter is always `self`

```rust
#[derive(Debug)]
struct Rectangle {
	width: u32,
	height: u32,
}

impl Rectangle { 
	fn area(&self) -> u32 {
		self.width * self.height
	}
}

fn main() {
	let rect1 = Rectangle {
		width: 30,
		height: 50,
	};

	println!("The area of the rectangle is {} square pixels.", rect1.area());
}
```

- Use `impl` (implementation) block to associate anything inside the block to the specified type
- `&self` is shorthand for `self: &Self`
- Understand use cases for `self`, `&self`, `&mut self`
- `rect.area` would refer to field and `rect.area()` refers to method
- Often same name for fields and methods for getter methods

#### Associated Functions
- All functions defined within `impl` block are called associated functions
- Associated functions that aren't methods are often used as constructor that return new instance of struct (usually in a more efficient way)

```rust
impl Rectangle {
	fn square(size: u32) -> Rectangle { 
		Rectangle {
			width: size,
			height: size,
		}
	}
}
```

- To call this associated function we use the `::` syntax 
	- `Rectangle::square(3);`
- It's possible to split methods into multiple `impl` blocks, no reason to here but it is valid