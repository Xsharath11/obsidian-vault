[[Web Assembly]]
Following the tutorials at https://bfnightly.bracketproductions.com/rustbook/webbuild.html

#### Creating a Project
- Run: `cargo init hellorust`
Auto generated `main.rs`:
```
fn main() {
	println!("Hello, world!);
}
```
- Rust started out as a mashup between ML (Meta Language) and C
- Does not need a VM like java and C# do

#### Rust Macros:
- `println!` is a rust macro
- Rust macros have an `!` after their name.
- More about macros: [here](https://doc.rust-lang.org/1.2.0/book/macros.html)
- They are basically special functions that are parsed into other code during compilation

#### Cargo Commands:
1. `cargo init` - init a new project
2. `cargo build` - downloads all dependencies and compiles them and then compiles your program
3. `cargo update` - fetch new versions of **crates** 