#### Why Rust?
- High Performance
- Garbage collection not necessary
- Concurrent Programming
- Easy to understand Errors

Cargo is Rust's package manager and build system
It is used to download libraries and build them etc

*Start a Rust Project*: `cargo new <Name>`
- This generates a `src` directory and a `Cargo.toml` file
- `Cargo.lock` stores versions of the dependencies

To disable warnings about unused variables:
`#![allow(unused)]` to the top of the file

*Adding dependencies*: Add the dependencies along with the version below `[dependencies]` in the `Cargo.toml`

 if we do not add the `mut` keyword while declaring variables, we will not be able to modify the variables, essentially making them a constant

*Declaring an empty string*: 
```
let mut name = String::new();
let greeting: &str = "Nice to meet you";
```

*Taking Inputs*
```
#![allow(unused)]

use rand::Rng;
use std::cmp::Ordering;
use std::fs::File;
use std::io;
use std::io::{BufRead, BufReader, ErrorKind, Write};

fn main() {
    println!("What is your name?");
    let mut name: String = String::new();
    let greeting: &str = "Nice to meet you";
    io::stdin()
        .read_line(&mut name)
        .expect("Didn't receive input");
    println!("Hello {}, {}", name.trim_end(), greeting);
}
```

*Creating constants*:
```
const ONE_MIL: u32 = 1_000_000;
const PI: f32 = 3.141592;
let age = "47";
let mut age: u32 = age.trim().parse()
	.expect("Age wasn't assigned a number")
```

##### This leads to the concept of **shadowing**:
- We can define variables with the same name but different datatypes
- We can also notice that we are building in our error handling directly as we code in the variables
- Since the age variable is now mutable, we can make changes on it
- We can define the datatypes by using a `: <type>`

#### Different Datatypes:
 Unsigned integer: *u8, u16, u32 ... usize*
 Signed integer: *i8, i16, i32 ... isize*

#### Fetching the max of data types:
```
println!("Max u32: {}", u32::MAX);
```