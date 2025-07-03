### Key Learnings:
- To get the input on the same line after a prompt, we need to do: `io::stdout().flush().unwrap();`
- We can take am input as a string very easily
- This must usually be converted to an integer
- For that we use the `parse()` method [[General Rust notes#This leads to the concept of **shadowing**]]
- Suppose we do not enter an integer, the program panics and fails
- To avoid this, we need to handle the exception rather than just using `expect` since we need to program to continue
- 