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
- We can also convert variables of one type to another by using the `parse()` method

#### Different Datatypes:
 Unsigned integer: *u8, u16, u32 ... usize*
 Signed integer: *i8, i16, i32 ... isize*

#### Fetching the max of data types:
```
println!("Max u32: {}", u32::MAX);
```

#### Booleans:
```
let is_true: bool = true
```

Another way to ignore errors and warnings for unused variables is by adding an `_` to the beginning of the variable ^533349

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

#### Array Methods:
```
let arr_1 = [1,2,3,4,5];
println!("1st: {}", arr_1[0]);
println!("Length: {}", arr_1.len());
```

*Looping through the Array*:
```
let arr = [1,2,3,5,6,7,8];
let mut loop_idx = 0;

loop{
	if arr[loop_idx] % 2 == 0 {
		loop_idx += 1;
		continue;
	}
	if arr[loop_idx] == 9 {
		break;
	}
	println!("Val: {}", arr[loop_ids]);
	loop_idx += 1;
}
```
^ Primitive loop

```
while loop_idx < arr.len(){
println!(...);
}
```
^ While Loops

```
for val in arr.iter() {
	println!(...);
}
```

#### Tuples
```
let my_tuple: (u8, String, f64) = (47, "Name".to_string(), 50_000.00)
println!("Name: {}", my_tuple.2);
println!("Age: {}", my_tuple.1);
```

**Unpacking a tuple:**
```
let(v1, v2, v3) = my_tuple;
println!("Age: {}", v1);
```

[[Creating a Guessing game in Rust]]