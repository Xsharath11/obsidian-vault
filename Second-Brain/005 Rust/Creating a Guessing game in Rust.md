### Key Learnings:
- To get the input on the same line after a prompt, we need to do: `io::stdout().flush().unwrap();`
- We can take am input as a string very easily
- This must usually be converted to an integer
- For that we use the `parse()` method [[General Rust notes#This leads to the concept of **shadowing**]]
- Suppose we do not enter an integer, the program panics and fails
- To avoid this, we need to handle the exception rather than just using `expect` since we need to program to continue
- Basically, any boilerplate print statements on stdout need to be flushed

```rust
#![allow(unused)]

use rand::Rng;
use std::cmp::Ordering;
use std::fs::File;
use std::io;
use std::io::{BufRead, BufReader, ErrorKind, Write};

fn main() {
    println!("Guessing Game!");
    let random_int: u32 = rand::rng().random_range(1..=10);

    // To have the input on the same line after the prompt, we need to flush stdout
    // flush just puts the text on the terminal immediately without being buffered
    // unwrap -> either gives you the value or panics
    io::stdout().flush().unwrap();
    for i in 0..10 {
        println!("---------------------------------");
        println!("Turn: {}", i);
        print!("please enter your input: ");
        io::stdout().flush().unwrap();
        let mut guess = String::new();
        io::stdin()
            .read_line(&mut guess)
            .expect("Did not receive an input");
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("Please enter an Integer");
                continue;
            }
        };
        //If we pass in &guess to the println!, it passes in the trailing \n which read_line adds by
        //default. Which is why we need to pass in something like guess.trim()
        //println!("Your guess is: {}", guess.trim());
        match guess.cmp(&random_int) {
            Ordering::Less => println!("Your guess was lesser than the secret number"),
            Ordering::Equal => {
                println!("You are RIGHT!!!");
                println!("Yurrrr");
                println!("---------------------------------");
                break;
            }
            Ordering::Greater => println!("Your Guess is Greater"),
        }
    }
}
```

#### Generating Random numbers within a certain range:
```
use rand::Rng;

fn main() {
	let random_int: u32 = rand::rng().random_range(1..101);
	println!("Random Number: {}",random_int)
}
```

#### Match 
```
fn main() {
	let age2: i32 = 8;
	match age2 {
		1..=18 => println!("Important Birthday"),
		21 | 50 => println!("Important Birthday"),
		65..=i32::MAX => println!("Important Birthday")
		_ => println!("Not an important birthday"),
	};
}
```
