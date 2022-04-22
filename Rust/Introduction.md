# Rust 
## Introduction
- ! indicates that the function is a macro, if it is omitted then it is a normal function
- Dyanmic languages compile and run programs in the same step (javascript, python), but Rust an ahead-of-time compiled language requires you to compile in advance
	- Enables you to run the executable without Rust install
- Cargo is a build system and package manager that handles building code, downloading libraries, and building libraries (dependencies)
- Configuration is determined in cargo.toml
	- [package] defines compiler information
	- [dependencies] contains list of crates used
- `rustc build` 
- `cargo build` then `cargo run`
- `cargo check` to quickly check that code is compilable vs `cargo build` checking creating an executable with 

[Rust Programming](Rust%20Programming.md)