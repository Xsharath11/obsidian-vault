#### What is Web Assembly?
Web assembly is a cool system that lets you run code compiled from non-web-based languages and runs them in the browser

However, it has a few limitations:
- You are sandboxed, so you can't access much in the way of files on the user's computer.
- Threads work differently in WASM, so normal multi-threading code may not work without help.
- Your rendering back-end is going to be OpenGL, at least until WebGL is finished.
- I haven't written code to access files from the web, so you have to embed your resources. The tutorial chapters do this with the various `rltk::embedded_resource!` calls. At the very least, you need to use `include_bytes!` or similar to store the resource in the executable. (Or you can help me write a file reader!)

#### Tools needed:
1. Target:
   Rust needs to have the "Target" installed to handle compilation to Web Assembly (WASM)
   `rustup target add wasm32-unknown-unknown`
2. `wasm-bindgen-cli`:
   This is a pretty impressive tool that can scan your web assembly and build the bits and pieces need to make the code run on the web.
   `cargo install wasm-bindgen-cli`