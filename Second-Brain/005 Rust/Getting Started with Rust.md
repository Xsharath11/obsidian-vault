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

##### Breaking down the code:
1. The first line is equivalent to C++'s `#include`. It tells the compiler that we are going to require `Rltk` and `GameState` from the namespace `rltk`
2. `struct State{}` - We are basically creating a new `structure`. These are basically like classes. We can store a bunch of data in them and also attach methods to them. More about structs: [here](https://doc.rust-lang.org/book/ch05-00-structs.html)
3. `impl GameState for State` - We are telling rust that our `State` structure implements the trait `GameState`. Traits are like base classes or base classes in other languages or an Abstract class: They setup a structure for you to implement in our own code, which can then interact with the library that provides them, without that library having to know anything else about your code.
   In this case, `GameState`  is a trait provided by RLTK. It uses it to call into your program on each frame.
   More about traits: [here](https://doc.rust-lang.org/book/ch10-02-traits.html)
4. `fn tick(&mut self, ctx : &mut Rltk)`- It is a function definition: [ref](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html)
   It does not end with a `-> type` which means it is essentially equivalent to a void function in C.
	1. `&mut self` - "This function requires access to the parent structure (State) and may change it" `mut` = mutable
	2. We can also have functions inside a structure that have just `&self` meaning they have access  to the contents of the struct but cannot change it
	3. `ctx: &mut Rltk` - ctx = context. The colon specifies the type of variable it must be
	4. `&` - pass as reference. Which is a pointer to an existing copy of the variable. The variable isn't copied, you are working on the version that was passed in. Any changes made are made on the original
	5. `Rltk` is the type of variable you are receiving. It is a struct defined inside the RLTK library that provides various things you can do to the screen