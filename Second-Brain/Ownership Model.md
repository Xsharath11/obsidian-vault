
|                          | Pros                                                                              | Cons                                                                                                |
| ------------------------ | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Garbage Collection       | - Error Free<br>- Faster dev time                                                 | - No control over memory<br>- Slower and unpredictable runtime performance<br>- Larger Program size |
| Manual Memory Management | - Fine control over memory<br>- Faster <br>Runtime<br>- Smaller Program size      | - Error Prone<br>- Slower dev time                                                                  |
| Ownership Model          | - Control over memory<br>- Error Free<br>- Faster Runtime<br>Smaller Program size | - Slower dev time<br>(Borrow Checker)<br>- Slower Compilation time                                  |
|                          |                                                                                   |                                                                                                     |
- Rust is also a memory safe language. Which means there is no fine control over memory
# Stack and Heap:
- During runtime, the program has access to both the stack and the heap
- Stack is fixed size and cannot grow during runtime
- Stack also contains stack frames which are created for every function that executes
- Stack Frame also contains local variables of the function being executed
- Variables in the Stack frame must have a known fixed size
-  Variables in a stack frame live only as long as the stack frame lives
- Size of  the stack is calculated at the compile time
- Heap can grow or shrink during runtime
- Heap is more unorganised
```rust
fn main() {
	fn a() {
		let x = "hello";
		let y = 22;
		b()
	}

	fn b() {
		let x:String = String::from("world");
	}
}

```
- Here in `b()`, `x`is a string which cannot be stored on the stack
- The stack stores pointers to the data which exists in the heap
- It is faster to push onto the stack than the heap since in the heap we will need to look for where to put the data
# Rust Ownership Rules:
1. Each value in rust has a variable that is called its owner.
2. There can only be one owner at a time
3. When the owner goes out of scope, the value will be dropped
```rust
fn main() {
	{ // s is not valid here, it's not declared yet
		let s = String::from("hello");
		// do stuff with s
	} // This scope is now over and s is no longer valid
}
```
- `s` here is a string literal
- string literals are stored directly in the binary and are fixed in size
- [Good Read](https://stackoverflow.com/questions/77591134/are-rust-string-literal-values-stored-on-the-heap)
# Memory and Allocation:
```rust
fn main() {
	let x: i32 = 5;
	let y: i32 = x; // This is a normal copy. y now has the value 5

	let s1: String = String::from("hello");
	let s2: String = s1; // Move (Not a shallow copy)
	println!("{}, World!", s1);
}
```
Let's break this down
- The first block of code is just a normal copy and functions as expected
- In the next block of code, `s1` contains:
	- a pointer which points to the actual location on the heap where 'hello' is stored
	- length of the string
	- capacity of the variable as in how much memory has been allocated on the heap for the string
- When we declare `s2` to `s1`, it does not create a deep or a shallow copy
- Instead, *ownership* of the string is *transferred/ moved* to `s2` which means that `s1` has been invalidated in the current scope
- Which is why the final `println!` will throw an error/ panic with:
	- *Value Borrowed After Move*
## What if we actually had to clone the variable?
```rust
let s2: String = s1.clone();
```

# Ownership and Functions:
```rust
fn main() {
	let s: String = String::from("hello");
	takes_ownership(s);
	println!("{}", s);
}

fn takes_ownership(some_string: String) {
	println!("{}", some_string);
}
```
- Here, ownership of `s` is moved to the `some_string` variable
- the `some_string` variable is then dropped once the function has finished executing
```rust
fn main() {
	let s: String = String::from("hello");
	takes_ownership(s);
	println!("{}", s);

	let x: i32 = 5;
	makes_copy(x);
	println!("{}", x);
}

fn takes_ownership(some_string: String) {
	println!("{}", some_string);
}

fn makes_copy(some_integer: i32) {
	println!("{}", some_integer);
}
```
- Integers are always copied so we can still use integers after the function call

Similarly, we can give and take back ownership as well

This becomes very tedious if we need to always give and take ownership when we need to work on the data in a variable

This is when the concept of references comes in

# References and Borrowing:
Let us start with an example
- Suppose we have a function that returns the length of a string and we also need to have the string in scope to work on it
- If we just calculate the length of a string and return the length, the string itself will be dropped from scope once the function has finished executing.
- The solution to this is to return a tuple that contains the string as well as the length of the string
```rust
fn main() {
	let s1: String = String::from("hello");
	let (s2, len) = calculate_length(s1);
	println!("The total length of {}, is {}", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
	let length: usize = s.len();
	(s, len)
}
```
- This is very weird and must never be done this way
- To fix it, we will modify it as follows
```rust
fn main() {
	let s1: String = String::from("hello");
	let len = calculate_length(&s1);
	println!("The total length of {}, is {}", s1, len);
}

fn calculate_length(s: &String) -> usize {
	let length = s.len();
	length
}
```
- Now we are basically defining the new function to take in a reference to a string rather than the string itself
- We are consequently passing in a reference to `s1` in the function call
- Once the function call is complete, only `s` is dropped which is fine since we still have `s1` in scope
- Passing in function params as references is called *Borrowing*
- References are immutable by default. which means we cannot modify the string
	- we cannot do `s.push_str("oops");`

## Mutable references:
What if we need to change the value in a function?
```rust
fn main() {
	let mut s1: String = String::from("hello");
	change(&mut s1);
}

fn change(some_string: &mut String) {
	some_string.push_str("world");
}
```
- `&mut` is a mutable reference
### Limitations to mutable references:
1. We can only have one mutable reference to a particular piece of data in a particular scope
```rust
fn main() {
	let mut s: String = String::from("hello");
	let r1 = &mut s;
	let r2 = &mut s; // This is an error
}
```
- The benefit due to this restriction is that it prevents any sort of race cases

2.  We cannot have a mutable reference if an immutable reference already exists in the same scope
```rust
fn main() {
	let mut s: String = String::from("hello");
	let r1 = &s;
	let r2 = &s; // We can have multiple immutable refs
	let r3 = &mut s; // Error

	println!("{}. {}", r1, r2);
}
```
- Cannot borrow 's' as mutable because it is also borrowed as immutable
- A scope of a ref starts when it is first introduced and ends when it is used for the last time. What this means:
```rust
fn main() {
	let mut s: String = String::from("hello");
	let r1 = &s;
	let r2 = &s; // We can have multiple immutable refs

	println!("{}. {}", r1, r2);

	let r3 = &mut s; // This is valid
	println!("{}", r3); 
}
```
- Here, `r3` is a valid ref since the scope of `r1` and `r2` have ended with their last use (`println!`)

### Dangling References:
```rust
fn main() {
	let ref_to_nothing = dangle()
}

fn dangle() -> &String {
	let s: String = String::from("hello");
	&s
}
```
- Here we are returning a reference to s
- But s is dropped at the end of the function execution
- This results in an error