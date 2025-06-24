Following the tutorials at https://bfnightly.bracketproductions.com/rustbook/webbuild.html

#### What is Web Assembly?
Web assembly is a cool system that lets you run code compiled from non-web-based languages and runs them in the browser

However, it has a few limitations:
- You are sandboxed, so you can't access much in the way of files on the user's computer.
- Threads work differently in WASM, so normal multi-threading code may not work without help.
- Your rendering back-end is going to be OpenGL, at least until WebGL is finished.
- I haven't written code to access files from the web, so you have to embed your resources. The tutorial chapters do this with the various `rltk::embedded_resource!` calls. At the very least, you need to use `include_bytes!` or similar to store the resource in the executable. (Or you can help me write a file reader!)

#### Tools needed:
