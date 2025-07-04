
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
- Stack also contains stackframes which are created for every function that executes
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
- Here in b(), x is a string which cannot be stored on the stack
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
- s here is a string literal
- string literals are stored on the heap