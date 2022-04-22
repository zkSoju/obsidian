### Ownership

- Ownership is how Rust is able to make memory safety guarantees without needing a garbage collector

#### What is Ownership?
- Ownership is a set of rules that determine how Rust manages memory
- Some languages use a garbage collection that constantly looks for no-longer used memory 
- Some languages programmer must explicitly allocate and free memory
- Rust uses a system of ownership with a set of rules used by compiler checks to manage memory
- Languages like Rust require consideration of the stack and heap
	- Stack - last in first out data structure
	- Heap - reference by pointer to memory allocation
		- Accessing and storing data on heap is slower than stack because a allocator needs to find space and you have to follow pointer to access
- When calling functions, values passed in (could be pointers to data) are pushed and popped to/from the stack
- Ownership allows you to minimize the amount of duplicate data on the heap and cleans up unused data so you don't run out of space
- Ownership Rules
	- each value in Rust has a variable that's called its owner
	- only one owner at a time
	- once owner goes out of scope, value will be dropped
- Previous types were all a known size, which can be stored/popped from the stack and trivially copied
- `String` is a good example of data that is stored on heap
- String literals can be hardcoded and immutable, but `String` is mutable and allocated on heap where length is unknown at compile time
- `::` operator allows us to namespace the `from` functino from `String` type instead of using `string_from`
- The following is immutable
```rust
let s = String::from("hello"); // string from string literal but immutable
```
- The following is mutable
```rust
let s = String::from("hello"); 

s.push_str(", world!");

println!("{}", s); // 'hello, world!'
```

#### Memory And Allocation
- String literals are fast and efficient b/c the contents are known at compile time and hardcoded into final executable
- For strings to be mutable and growable we need to allocate memory on heap:
	- Memory must be requested from memory allocator at runtime
		- Used when calling `String::from`, the implementation requests memory it needs
	- We need a way of returning memory to allocator when done w/ `String`
		- Garbage collector or explicit memory return in other languages
		- In Rust, memory is automatically returned once variable goes out of scope using a function called `drop`
- Seems simple at first but this behavior can cause unexpected situations where we want multiple variables using data alloc on the heap
```rust
let x = 5; // behaves as expected
let y = x;
```

```rust
{
	let s1 = String::from("hello");
	let s2 = s1; 

	println!("{}, world!", s1); // compiler error
}
```

- s2 becomes a pointer to the s1 `String` 
	- however, this is an issue because s2 and s1 point to `hello` on the heap and when they go out of scope Rust will try to run `drop` twice causing a *double free* error
	- instead for memory safety Rust considers `s1` invalid after `let s2 = s1`
	- other programming languages have concepts of `shallow copy` (ex. copy pointer) or `deep copy` (contents of heap copied)
	- Rust is similar to `shallow copy` but invalidates the first variable, so it's called `move`
		- rust won't automatically create "deep" copies
		- therefore, *automatic* copies can be assumed to be inexpensive in terms of runtime performance
- in order to perform a deep copy of `String`, we would use the `clone` function
- referencing the following example again, seems to contradict what we just learned b/c we don't have to call `clone` but these integers are known at compile time so they are stored on the stack and cheap to copy
```rust
let x = 5; // behaves as expected
let y = x;
```
- `Copy` trait defines types that are still valid after assignment to another variable (primarily simple scalar values)
	- Integer types
	- Boolean types
	- Floating types
	- Character types
	- Tuples if they only contain types that also implement Copy

#### Ownership and Functions
- Passing a value to a function is similar to assigning a value to a variable, it will move or copy depending on type

```rust
fn main() {
	let s = String::from("hello");
	takes_ownership(s); // s value moves to function and is not longer valid here

	let x = 5;
	makes_copy(x); // x is copied into function, so it is still valid here
}
```

#### Return Values and Scope
- Returning values can transfer ownership

```rust
fn main() {
	let s1 = gives_ownership();

	let s2 = String::from("hello");
	
	let s3 = takes_and_gives_back(s2);
}

fn gives_ownership() -> String {
	let some_string = String::from("yours");

	some_string
}

fn takes_and_gives_back(a_string: String) -> String {
	a_string
}
```

- if a value on the heap goes out of scope, value will be cleaned up by `drop` unless ownership of the data has been moved to another variable
- while this works, taking ownership then returning ownership is tedious
	- Rust has a feature called references to mitigate this
- return multiple values using a tuple

```rust
fn calculate_length(s: String) -> (String, usize) {
	let length = s.len();

	(s, length)
}
```

#### References and Borrowing
- References allow use to pass in a pointer, which is an address that we can follow to access data owned by some other variable
- Reference is different than pointer in that it is guaranteed to point to a valid value of particular type
- This is called *borrowing*, someone else owns the variable, the function is just borrowing

```rust
fn main() {
	let s1 = String::from("hello");
	let len = calculate_length(&s1); // create reference that refers to value s1 but does not own it

	println!("The length of {} is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize { // s is reference to string
	s.len()
} // because this function does not have ownership, it does not get dropped (reference)
```

- You can't modify something you're borrowing because references are immutable by default

```rust
fn main() {
	let s = String::from("hello");

	change(&s);
}

fn change(some_string: &String) {
	some_string.push_str(", world"); // compiler error
}

```

#### Mutable References

- We can use mutable references to achieve this effect

```rust
fn main() {
	let mut s = String::from("hello");

	change(&mut s);
}

fn change(some_string: &mut String) {
	some_string.push_str(", world");
}
```

- You can only have one mutable reference to a piece of data at a time

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s; // compiler error - cannot borrow s as mutable more than once at a time

println!("{}, {}", r1, r2);
```

- This may be difficult to wrap head around, but it prevents *data races* similar to race condition which happens when three behaviors occurs:
	- Two or more pointers access same data at same time
	- At least one of pointers being used to write data
	- There's no mechanism being used to synchronize access to data
- Possible to create multiple mutable references just not simultaneous ones

```rust
let mut s = String::from("hello");

{
	let r1 = &mut s;
}

let r2 = &mut s;
```

- We also can't have a mutable reference while having an immutable one because users of immutable reference don't expect value to change

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
let r3 = &mut s; // compiler error - cannot borrow s as mutable b/c it is also borrow as immutable
```

- The following is valid because the scope of immutable reference ends when they are last used

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2);

let r3 = &mut s;
println!("{}", r3);
```

#### Dangling References

- In languages with pointers, it is easy to erroneously create a *dangling* pointer, which is a ponter that references a location in mem that may be given to someone else
	- Freeing memory while preserving pointer to that memory
- In Rust, the compiler ensures that the data will not go out of scope before reference does

```rust
fn main() {
	let reference_to_nothing = dangle();
}

fn dangle() -> &String {
	let s = String::from("hello");

	&s // we try to return a reference to a String that is dropped after going out of scope
}
```

- Solution is to return the String directly

#### References Key Points

- At any given time, you can either have one mutable reference or any number of immutable references
- References must always be valid

#### Slices

- Slices allow you to reference a contiguous sequence of elements in a collection rather than whole collection
- Kind of reference so it doesn't have ownership
- Consider the problem: write a function that takes a string and returns first word, entire string if no space is found
```rust
fn first_word(s: &String) -> ? {

}

fn first_word(s: &String) -> usize {
	let bytes = s.as_bytes();

	for (i, &item) in bytes.iter().enumerate() {
		if item == b' ' {
			return i;
		}
	}

	s.len();
}
```

```rust
fn main() {
	let mut s = String::from("hello world");

	let word = first_word(&s); // 5

	s.clear(); // this empties String making it equal to 0

	// word will still have value 5 here, but there's no more string
	// that we could use the value 5 with
}
```

- **Issue**: indexes with no connection to the state at all will cause data to become out of sync and difficult to track
- **Solution:** string slices

#### String Slices

- *string slice* is a reference to part of a String and it looks like

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];

// or

let hello = &s[..5];
let world = &s[6..];

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

- The previous problem now using string slices would look like

```rust
fn first_word(s: &String) -> &str { // &str is type that refers to string slice
	let bytes = s.as_bytes();

	for (i, &item) in bytes.iter().enumerate() {
		if item == b' ' {
			return &s[0..i];
		}
	}

	&s[..]
}
```

- Without string slices, the user would not be notified of a bug even though the index became invalid and code is logically incorrect
- Using string slices, the following would throw a compiler error
```rust
fn main() {
	let mut s = String::from("hello world");

	let word = first_word(&s); // immutable borrow

	s.clear(); // mutable

	println!("the first word is: {}", word); // immutable
	
}
```

- Rust throws an error because `s.clear()` uses an mutable borrow and `println!` uses an immutable reference

#### String Literals are Slices

- Recall string literals being stored inside binary, string literals pointers to that specific point of binary
- Literals are slices of type `&str`

```rust
let s = "Hello world"; // immutable &str
```

- Knowing that you can take slices of literals and `String` values allows us to do, which is more general and useful without losing functionality
	- This flexibility is called *deref coercions*

```rust
fn first_word(s: &str) -> &str {}

fn main() { // the following are all valid
	let my_string = String::from("hello world");

	let word = first_word(&my_string[0..6]);
	let word = first_word(&my_string[..]);

	let word = first_word(&my_string);

	let my_string_literal = "hello world";

	let word = first_word(&my_string_literal[0..6]);
	let word = first_word(my_string_literal)
}

```

#### Other Slices

```rust
let a = [1,2,3,4,5];

let slice = &a[1..3];

assert_eq!(slice, &[2,3])
```
