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
3. `cargo update` - fetch new versions of **crates** listed in `cargo.toml`
4. `cargo clean` - deletes all intermediate work files, freeing up storage
5. `cargo verify-project` - check if cargo settings are correct
6. `cargo install` - install programs via cargo
7. `cargo search` - search for packages

#### Setup Cargo.toml
- We are using `rltk` -> Rougelike toolkit library
- Add the dependancy:
---
  ```
[dependencies]
rltk = { version = "0.8.7" }
```
---
It's a good idea to occasionally run `cargo update` - this will update the libraries used by your program.

#### Hello World in RLTK:
```
# src/main.rs
use rltk::{GameState, Rltk};

struct State {}
impl GameState for State {
    fn tick(&mut self, ctx: &mut Rltk) {
        ctx.cls();
        ctx.print(1, 1, "Hello Rust");
    }
}

fn main() -> rltk::BError {
    use rltk::RltkBuilder;
    let context = RltkBuilder::simple80x50()
        .with_title("RougeLike Tutorial")
        .build()?;
    let gs = State {};
    rltk::main_loop(context, gs)
}
```
[[Errors Faced]] ^err1