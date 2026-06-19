# GitHub Copilot Instructions for Wasm Tools Monorepo

You are an expert AI assistant specializing in Rust, WebAssembly (`wasm-bindgen`), Node.js Workspaces, and CLI development. Adhere strictly to the following rules when generating code for this repository.

## 🎯 Architecture Context
- **Monorepo Design**: The project consists of independent Rust crates inside `crates/` and corresponding JS packages inside `packages/`.
- **Target**: Browser (via individual npm packages) and Node.js (via a centralized local CLI wrapper tool in `packages/cli`).

## 🦀 Rust & Wasm Coding Rules

### 1. Multi-Crate Isolation
- Keep crates inside `crates/` entirely decoupled. Do not share global states across different crates unless explicitly declared as a shared dependency in `Cargo.toml`.
- Expose all public JavaScript-accessible bindings using the `#[wasm_bindgen]` attribute.

### 2. Memory Boundary & Performance
- Minimize heavy data serialization (e.g., JSON stringifying) across the JS/Rust boundary.
- Prioritize passing primitive types (`i32`, `f64`) or typed primitive arrays (`Vec<u8>`).
- If complex data transfer is required, utilize `serde-wasm-bindgen` and document the potential performance overhead.

### 3. Safety & Error Handling
- Never use `panic!()`, `unwrap()`, or `expect()` in production Wasm logic.
- Always bubble up errors gracefully by returning a `Result<T, JsValue>`. Map Rust native errors to JS-friendly strings using `.map_err(|e| JsValue::from_str(&e.to_string()))`.

## 🟢 Node.js CLI Coding Rules

### 1. Dynamic Wasm Loading
- The Wasm libraries are built targeting the browser (`--target web`). When mocking or calling these libraries inside the Node.js CLI environment, ensure files are handled via standard Node.js mechanisms (e.g., loading bytes using `fs.readFileSync` or standard environmental polyfills).

### 2. Decoupled CLI Design
- Keep the CLI layer inside `packages/cli` extremely thin.
- The CLI should only handle argument parsing (e.g., using `commander` or `yargs`), reading local files into buffers, feeding those buffers directly into the Wasm modules, and writing outputs to the disk. 
- Core utility logic must *never* be written inside the CLI directory; it belongs inside the Rust crates.

## 💡 Code Generation Style
- **Rust**: Write clean, idiomatic, and memory-safe Rust. Prefer explicit type annotations for clarity.
- **JavaScript/TypeScript**: Generate modern ECMAScript Modules (ESM) using `import`/`export` syntax. Implement strict asynchronous error handling (`try/catch` with `async/await`).

