### Common Collections
- Standard library includes a number of useful data structures called collections
- Most other data types represent one specific value, but collections contain multiple values
- Commonly used collections
	- Vector - store variable number of values next to each other
	- String - collection of characters
	- Hash map - associate values to particular key

#### Storing List Of Values with Vectors
- `Vec<T>` stores more than one value in a single data structure that puts all values of the same type next to each other in memory

#### Creating a New Vector

```rust
let v: Vec<i32> = Vec::new();
```

- When initializing a data type that implements generics, you have to specify the type in angle brackets if you are not initializing with initial values where Rust can infer the type
- You can use the `vec!` macro to create a new `Vec<T>` with initial values

```rust
let mut v = vec![1, 2, 3];

v.push(5); // variable must be mut
v.push(6);
```

#### Dropping a Vector Drops Its Elements

```rust
{
	let v = vec![1, 2, 3, 4];
}
```

- When a vector gets dropped, all of it's contents are dropped and cleaned up
- Becomes complicated when you introduce references to elements of vector

#### Reading Elements of Vectors

- Two ways to reference value storage in vector: indexing or using `get` method

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
	Some(third) => println!("The third element is {}", third),
	None => println!("There's no third element.");
}
```

- The reason for Rust providing two ways is so that you can choose how program behaves when trying to access out of bounds
- `get` method is valid for all ranges and will return an `Option<&T>`, which will require your code to handle having `Some(&element)` or `None`
- `[]` will cause the program to panic because it references a nonexistent element
- Borrowing rules also prevent having immutable and mutable references in the same scope

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0]; // immutable borrow

v.push(6); // mutable borrow

println!("The first element is: {}", first); // immutable borrow
```

- The above is an error because of the way vectors work
	- Adding a new value next to each other in memory could require allocation of new memory and copying of old elements into new space
	- If this happens, then the reference to the first eleemnt would be pointing to dealloc mem, which would be an issue
	- Borrowing rules prevent this

#### Storing UTF-8 Encoded Text with Strings
- What is a String?
	- Core language has one string type the string slice `str`
	- `String` type, provided by std is growable, mutable, owned, and UTF-8 encoded 
	- Both are used heavily; however, this section is primarily about `String`
- Creating a new string

```rust
let mut s = String::new();

let data = "initial contents";

let s = data.to_string(); // string literal to String
let s = String::from("initial contents") // same as above
```

 #### Updating a string
 - `push_str` takes a string slice because we don't necessarily want to take ownership of the parameter

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

#### Concatenation with `+` operator or `format!` macro

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2;
```

- The reason for s2 reference and s1 no longer being valid is because of  `+` operator using `add` method

```rust
fn add(self, s: &str) -> String {
```
- The compiler is able to use deref coercion to coerce `&String` type of `s2` into a `&str`
- `self` is not a reference so the variable is moved into the call
- `let s3 = s1 + &s2;` looks like it copies both strings to create a new one, but instead it takes ownership of `s1` and appends copy of `s2` making the implementation more efficient
- Use `format!` macro for concatenating multiple strings

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
let s = format!("{}-{}-{}", s1, s2, s3);
```

#### Indexing into Strings

- Accessing individual characters by index in a string is common and valid in other programming languages; however, you will get an error in Rust

```rust
let s1 = String::from("hello");
let h = s1[0]; // error - String cannot by indexed by integet
```

- Rust strings don't support indexing because of how Rust stores strings in memory

#### Internal Representation
- `String` is a wrapper over a `Vec<u8>`
- Indexing won't work because string's bytes won't always correlate to value Unicode scalar value

```rust
let hello = String::from("Здравствуйте"); // you might think 12, but it's 24 bytes
let hello = String::from("hello"); // 5 bytes long
```

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

- answer will not be `З` because `З` encoded in UTF-8 is made up of two bytes `208` and `151`; however, returning `208` is not a valid character on it's own and is probably not what the user expects
- Rust avoids this bug causing situation by preventing this code from compiling

#### Bytes, Scalar Values, and Grapheme Clusters

- Hindi word नमस्ते written in Devanagari script is stored as a vector of `u8` values that looks like  `[224, 164, 168, 224, 168...]`
- As unicode scalar values they look like `['न', 'म', 'स', '्', 'त', 'े']`
- But 4th and 6th are no letters and don't make sense on their own
- Indexing strings is expected to take O(1) time but Rust would have to walk through contents to check valid characters

#### Slicing Strings

- Indexing strings is often a bad idea because it is not clear on the return type 
- If you need to use indices to create string slices, Rust asks for you to be more specific by specifying a range
- `let s = &hello[0..4]`
- You should use caution when using ranges to create string slices since you will crash program if trying to access an invalid index like `&hello[0..1]` for the Hindi word

#### Methods for Iterating over Strings

```rust
for c in "".chars() {

}

for b in "".bytes() {

}
```

#### HashMaps
- `HashMap<K,V>` stores a mapping of keys type `K` to values type `V` with hasing
- Useful for looking up data with keys and not an index

#### Creating a New HashMaps
- One way to create an empty hash map is using `new` and adding with `insert`

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

- Among the three collections, this is the one least used so it isn't brought into scope automatically, so it is necessary to `use` from standard library
- We could have two separate vectors and `zip` them together to create an iterator of tuples then use `collect` to turn iterator of tuples into a hashmap

```rust
use std::collections::HashMap;

let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10,50];

let mut scores: HashMap<_,_> = teams.into_iter().zip(initial_scores.into_iter()).collect();
```

- Must specify `HashMap<_,_>` here because you can collect into many data structures
	- We can use `_` because Rust can infer type

#### HashMaps and Ownership
- Similar to normal ownership rules

#### Accessing Values in a HashMap
- `get` returns an `Option<&V>`
- Or use `for in`
```rust
for (key, value) in &scores {
	println!("{}: {}", key, value);
}
```

#### Updating a HashMap

- Overwriting values in the HashMap
	- Use `insert` to overwrite values
- Only inserting a value if key has no value
	- Hashmaps have special API called `entry`, which returns `Entry` enum
	- `scores.entry(String::from("Yellow")).or_insert(50)`
	- `or_insert` method on `Entry`  will insert the value at the key if it doesn't exist and return a mutable reference to the value if it exists
- Updating a value based on old value
	- Look up key value and update based on old value

```rust
for word in text.split_whitespace() {
	let count = map.entry(word).or_insert(0);
	*count += 1;	
}
```

- in order to increment we need to `*` deference count, which is a `&mut V`



