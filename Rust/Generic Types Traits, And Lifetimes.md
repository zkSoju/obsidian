#### Generic Types Traits, And Lifetimes
- Generics are abstract stand-ins for concrete types
- Traits define behavior in a generic way, constrains generic types to only types with particular behavior
- Lifetimes is a variety of generics that give compiler information about how references relate to each other

#### Removing Duplication by Extracting a Function

```rust
fn main() {
	let number_list = vec![34, 50, 25, 100, 65];

	let mut largest = number_list[0];

	for number in number_list {
		if number > largest {
			largest = number;
		}
	}

	let number_list = vec![34, 50, 25, 6000, 100, 65];

	let mut largest = number_list[0];

	for number in number_list {
		if number > largest {
			largest = number;
		}
	}
}
```

- Duplicate code is tedious and error prone
- To elimate duplication we can create an abstraction with a function

```rust

fn largest(list: &[i32]) -> i32 {
	let mut largest = list[0];

	for &item in list {
		if item > largest {
			largest = item;
		}
	}
	
	largest
}

fn main() {
	let number_list = vec![34, 50, 25, 6000, 100, 65];

	let result = largest(&number_list);
}
	
```

- Use generics eliminate code duplication for abstract type

#### Generic Data Types

```rust
fn largest<T>(list: &[T]) -> T {
	let mut largest = list[0];

	for &item in list {
		if item > largest {
			largest = item
		}
	}
}
```

- ex. Ability to handle largest `i32` in slice and largest `char` in slice
- Read as: the function `largest` is generic over some type `T`. This function has one parameter named `list` which is a slice of values of type `T`. The function will return a value of type `T`
	- Think of `<T>` as a parameter for function signature
- However, this function won't work for all possible type `T` because only some types can be ordered with binary operators
	- We learn about using `std::cmp::PartialOrd` trait to enable comparisons later

#### In Struct Definitions

```rust
struct Point<T> {
	x: T,
	y: T,
}

impl<T> Point<T> {
	fn x(&self) -> &T {
		&self.x
	}
}

impl Point<f32> {
	fn distance_from_origin(&self) -> f32 {
		0 // this code means type Point<f32> will have this method and other types won't
	}
}

struct NewPoint<T, U> {
	x: T,
	y: U,
}

fn main() {
	let integer = Point { x: 5, y: 10 };
	let float = Point { x: 1.0, y: 4.0 };
	let wont_work = Point { x: 1, y: 4.0 }; // must be same type
	let works = NewPoint { x: 1, y: 4.0 }; // must be same type
	
}
```

- At first glance, generics may seem to impact performance but Rust handles this with *monomorphization*, which turns generic code into specific code by filling concrete types used when compiled
	- Duplicates each definition behind the scene making it extremely efficient at runtime

```rust
let integer = Some(5);
let float = Some(5.0);
```

- Rust reads values used by `Option<T>` instances and expands generic definition of `Option<T>` into `Option_i32` and `Option_f64`

```rust
enum Option_i32 {
	Some(i32),
	None,
}

enum Option_f64 {
	Some(i32),
	None,
}

fn main() {
	let integer = Option_i32::Some(5);
	let float = Option_f64::Some(5.0);
}
```

#### Traits: Defining Shared Behavior

- Trait definitions are a way to group method signatures together to define a set of behaviors across multiple types
- ex. Multiple structs to hold various kinds of text
	- `NewsArticle` struct which holds a news story at a location
	- `Tweet` struct that has at most 280 characters and metadata about tweet type
	- We want to make a media aggregator crate that display summaries of data stored in `NewsArticle` or `Tweet`

```rust
pub trait Summary {
	fn summarize(&self) -> String;
}
```

- Compiler will enforce any type that has `Summary` trait will have the method defined `summarize`
- Think of traits as an interface for types

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

```

- Now users of the crate can call trait methods on instances of `NewsArticle` and `Tweet`
- Just need to bring the trait and type into scope

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

#### Default Implementations

- You can specify default behavior instead of requiring implementations for all methods on every type

```rust
pub trait Summary {
	fn summarize(&self) -> String {
		String::from("(Read more...)")
	}
}
```

#### Traits as Parameters

```rust
pub fn notify(item: &impl Summary){
	println!("Breaking news! {}", item.summarize());
}

// these two functions are equiv
pub fn notify<T: Summary>(item: &T){ 
	 ... 
}
```

- Instead of a concreate type we can specify `impl` and trait name accepting any type that implements the trait
- `impl Trait` syntax is sugar coat for second function which defines a *trait bound*

```rust
// can be diff types
pub fn notify(item1: &impl Summary, item2: &impl Summary)

// must be same types
pub fn notify<T:Summary>(item1: &T, item2: &T)
```

- You can also specify more than one trait bound

```rust
pub fn notify(item: &(impl Summary + Display))

pub fn notify<T: Summary + Display>(item: &T)
```

- Too many trait bounds can make function signature hard to read, so Rust has `where` clauses

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {

fn some_function<T, U>(t: &T, u: &U) -> i32 
	where T: Display + Clone, 
	U: Clone + Debug {
```

- Returning types that implement traits with `impl Trait`
	- However this function can only return a single type not either NewsArticle or Tweet

```rust
fn returns_summarizable() -> impl Summary {}
```

#### Fixing largest Function with Trait Bounds

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {}
```

- We don't need to bring PartialOrd into scope because it's in prelude
- Now function only works on types we can compare
- However, we get a new set of errors with types that don't implement the `Copy` trait because we don't have ownership
	- We can modify the above function to handle this

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {}
```

#### Using Trait Bounds Conditionally Implement Methods 

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}

```

- Implement trait for any type that implements another trait called *blanket implementations*

```rust
// implement ToString trait for all types that implement Display
impl<T: Display> ToString for T {}
```

#### Preventing Dangling References with Lifetimes

```rust
{
	let r; 
	{
		let x = 5;
		r = &x; // compiler error - borrowed value does not live long enough
	} // x dropped here while still borrow

	println!("r: {}", r);
}
```

- The aim of lifetimes is to prvent dangling references, which cause a program to reference data other than data it's intended
- Variable `x` does not "live long enough" but `r` is still valid
- If Rust allowed this to work, `r` would be referncing memory that was dellocated when `x` went out of scope
- Rust determines code is invalid through borrow checking

#### The Borrow Checker

- Borrow checker compares scopes to determine whether all borrows are valid

```rust
fn main() {
    {
        let r;                // ---------+-- 'a
                              //          |
        {                     //          |
            let x = 5;        // -+-- 'b  |
            r = &x;           //  |       |
        }                     // -+       |
                              //          |
        println!("r: {}", r); //          |
    }                         // ---------+
}

```

- Rust sees that the inner `'b` block has a much smaller lifetime than outer `'a` block
- At compile time Rust sees that `r` has a lifetime of `'a` but references memory with lifetime of `'b`
- The following fixes the code such that `'b` is larger than `'a` 
	- `r` can now reference `x` because Rust knows that the reference in `r` will always be valid while `x` is valid

```rust
fn main() {
    {
        let x = 5;            // ----------+-- 'b
                              //           |
        let r = &x;           // --+-- 'a  |
                              //   |       |
        println!("r: {}", r); //   |       |
                              // --+       |
    }                         // ----------+
}

```

#### Generic Lifetimes in Functions

```rust
fn main() {
	let string1 = String::from("abcd");
	let string2 = "xyz";

	let result = longest(string1.as_str(), string2);
}

fn longest(x: &str, y: &str) -> &str {
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```

- The above won't compile because Rust can't tell whether the reference being returned refers to `x` or `y` 
- We don't know concrete values and concrete lifetimes of references passed in or whether `if` or `else` will execute, so we can't look at the scopes to determine validity of references
- We. add generic lifetime parameters that define relationship between references so borrow checker can perform analysis

#### Lifetime Annotation Syntax
- Lifetime annotations don't change how long any references live
- Functions can accept any type with generic type parameter
- We can also setup functions to accept references with any lifetime with generic lifetime parameter

```rust
&i32
&'a i32
&'a mut i32
```

- Lifetime annotation by itself doesn't mean much because it's meant to tell Rust how multiple references relate to each other
- A value `first` with `'a` lifetime should live as long as `second` with lifetime `'a`

#### Lifetime Annotation in Function Signatures

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```

- Lifetime annotations are in the function signature, so that they become part of the function *contract* making it easier for Rust compiler

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

- The above code does not compile because the lifetime of `string2`  is shorter than `result`. Rust doesn't think `string1` is valid either because we told `longest` that lifetimes of the references are the shorter of the two

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

#### Thinking In Terms of Lifetimes

- The way in which we need to define lifetime parameters depends on what the function is doing

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str { x }
```

- This would compile because:
	- the lifetime of `x` and the return type has no relationship with `y`
	- the lifetime of the return type matches one of the parameters

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

- This would not compile because:
	- the lifetime of the return type does not match any parameters
	- we are trying to return a reference to a value that goes out of scope, we can solve this by returning an owned data type instead
- Lifetime syntax is meant to connect lifetimes of multiple parameters and return values, so that Rust can allow memory safe operations and prevent dangling references or violate memory safety

#### Lifetime Data Type in Structs

```rust
struct ImportantExcerpt<'a> {
	part: &'a str,
}

fn main() {
	let novel = String::from("Call me...");
	let first_sentence = novel.split('.').next().expect("Couldn't find .");
	let i = ImportantExcerpt {
		part: first_sentence,
	}
}
```

- `ImportantExcerpt` holds a reference to first sentence of `String` owned by `novel`. `novel` exists before `ImportantExcerpt` and goes out of scope after, so the reference is valid

#### Lifetime Elision

- You learned that every reference has a lifetime and you need to specify lifetime parameters for functions or structs that use references, but you have use functions with references and no lifetime annotation before

```rust
fn first_word(s: &str) -> &str {}
```

- Annotations were required in early Rust but situations for usage became deterministic and developers programmed patterns into compiler's code, so borrow checker could infer
- *Lifetime elision* rules determine cases where you don't need to explicity annotate lifetime
	- Each parameter that is a reference gets its own lifetime parameter
		- `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`
	- If there is one input lifetime parameter that lifetime is assigned to all output lifetime parameters
		- `fn foo<'a>(x: &'a i32) -> &'a i32`
	- If there are multiple input lifetime parameters, but one is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output
	- Example
		- 1. `fn first_word(s: &str) -> &str`
		- 2. `fn first_word<'a>(s: &'a str) -> &str`
		- 3. `fn first_word<'a>(s: &'a str) -> &a' str`
	- However a situation like 

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {}
```

- Rust is unable to infer from any rules what the return type's lifetime is based on references, which is why Rust will give you an error

#### Lifetime Annotations in Method Definitions

```rust
impl<'a> ImportantExcerpt<'a> {
	fn level(&self) -> i32 {
		3
	}

	fn announce_and_return_part(&self, announcement: &str)
}
```

- Lifetime parameter declaration after `impl` and use after type name are required but we don't need to annotate in method because of third elision rule

#### Static Lifetime

- `'static` lifetime means reference can live for entire duration of the program
	- All string literals have the `'static` lifetime, since it's stored in programs binary
- You might see suggestions to use `static` in error messages but think if reference you have can/should live entire lifetime

#### Generic Type Parameters, Trait Bounds, and Lifetimes Together

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a,T>(
	x: &'a str,
	y: &'a str,
	ann: T,
) -> &'a str
where
	T: Display,
{
	println!("Announcement! {}", ann);
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```

