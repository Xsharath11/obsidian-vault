1. On the first `cargo run` ran into an error caused due  to `cmake` not being installed and `libfontconfig1-dev` not being installed. [[Getting Started with Rust - Hello World#^err1]]

# Further Explorations
---
1. What does the `?` operator do?
   [[Entities and Components#^Further-Explorations-1]]
```
error[E0308]: mismatched types
    --> src/main.rs:65:21
     |
65   |     rltk::main_loop(context, gs)
     |     --------------- ^^^^^^^ expected `Rltk`, found `Result<Rltk, Box<dyn Error + Send + Sync>>`
     |     |
     |     arguments to this function are incorrect
     |
     = note: expected struct `BTerm`
                  found enum `Result<BTerm, std::boxed::Box<dyn std::error::Error + Send + Sync>>`
note: function defined here
    --> /home/sharath/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/bracket-terminal-0.8.7/src/bterm.rs:1088:8
     |
1088 | pub fn main_loop<GS: GameState>(bterm: BTerm, gamestate: GS) -> BResult<()> {
     |        ^^^^^^^^^
help: use the `?` operator to extract the `Result<BTerm, std::boxed::Box<dyn std::error::Error + Send + Sync>>` value, propagating a `Result::Err` value to the caller
     |
65   |     rltk::main_loop(context?, gs)
     |                            +
```
2. 