### Writing Automated Tests

- We can write tests that assert that we receive the return value we expect

#### How to Write Tests

- Test functions typically perform these three actions
	- Setup any needed data or state
	- Run code you want to test
	- Assert results are what you expect
- Includes `test` attribute, a few macros, and `should_panic` attribute

#### Anatomy of a Test Function

```rust
#[cfg(test)]

mod test {
	#[test]
	#[should_panic(expected="")]
	fn it_works() {
		assert_eq!(2 + 2, 4);
	}
}
```

- Expected searches for panic with string as substring of panic message
- Refer to `adder/src/lib.rs` 

#### Controlling How Tests Are Run

- By default tests are run in parallel with threads meaning tests will run fasts and you will get feedback quicker
- Tests are running at same time, so make sure tests aren't depending on each other or shared data
- Example: multiple tests read/write data and asserts file contains particular value
	- Because tests run at the same time one might overwrite file when another is writing/reading causing tests to fail
	- `--tests-threads=1` to limit number of threads used

#### Showing Function Output

- By default if a test passes, Rust's test library captures `println!` and we only see line that inidicates test pass
- `--show-output` to show printed values for passing tests as well

#### Running a Subset of Tests by Name

- We can pass a name of any test function to `cargo test` to run only that test
- We can pass a part of test name to filter names that don't match `cargo test add` runs tests that contain `add`

#### Ignoring Some Tests Unless Specifically requested

- You can use the `#[ignore]` attribute to exclude them
- `-- --ignored` to run only ignored tests

#### Test Organization

- `#[cfg(test)]` tells Rust to compile and run test code only when you run `cargo test`, so that resulting compiled library doesn't include tests
- Intergration tests that go in a different directory don't need `#[cfg(test}]` 
- Unit tests that go in same file of could you need to specify so it isn't included in compiled result

#### Testing Private Functions

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

- Rust will not prevent testing private functions because `tests` module works as any other module
	- `internal_adder` is a function of the parent `crates` module, so it is accessible to the child module `tests`

#### Integration Testing

- Completely external to your library and use library in same way any other code would
	- Can only call functions part of your library's public API
- Add file to `tests/` directory then bring crates into scope with `use`
- Cargo treats `tests` directory specially and compiles files only when we run `cargo test
	- We don't need to annotate any code with `#[cfg(test)]` 
- `cargo test --test integration_test` to run a particular integration test file
- We can add setup files in `tests/common/` Rust knows to treat these modules not as an integration test file otherwise the setup function will show up as a test

`tests/common/mod.rs`

```rust
pub fn setup() {

}
```

`tests/integration_test.rs`

```rust
use adder;

mod common;

#[test]
fn it adds_two(){
	common::setup();
	assert_eq!(4, adder::add_two(2));
}
```

- TDD (Test driven development)
	- write failing tests and make sure it fails for the reason you expect
	- write or modify enough code to make test pass
	- refactor code you just added to make sure tests continue to pass