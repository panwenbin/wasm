# GitHub Copilot Instructions for Wasm Tools Monorepo

You are an expert AI assistant specializing in Rust, WebAssembly (`wasm-bindgen`), npm/browser packaging, and CLI architecture. Adhere strictly to the following rules when generating code for this repository.

## 🎯 Architecture Context

- This repository is **dual-target-first**: Rust crates are the core implementation layer, and many tools may later expose browser/JavaScript-facing packages under `packages/`.
- The repository is **not necessarily fully bootstrapped yet**. Until `Cargo.toml`, `crates/`, and `packages/` are actually created, describe them as a **recommended or planned structure**, not as already-existing facts.
- Browser/npm packages are delivery surfaces for JavaScript consumers. They are **not** the default local debugging entrypoint.

## 🧭 CLI Architecture Rules

### 1. Single CLI Only

- There is **exactly one** end-user CLI application in this repository.
- That CLI lives in **`crates/cli`**.
- Do **not** create extra CLI applications in other crates or in `packages/` unless the user explicitly asks for one.

### 2. Subcommand Model

- Every end-user terminal tool should be exposed as a **subcommand** of the centralized CLI in `crates/cli`.
- The CLI layer should stay thin and only handle:
  - command registration
  - argument parsing
  - IO boundaries
  - dispatch into tool crates
- End-user command behavior and business logic should live in tool-specific Rust crates, not in the CLI crate itself.

### 3. Native CLI First

- The primary local development and debugging entrypoint is the **native Rust CLI**.
- Prefer running and designing around `crates/cli` first for local command-line workflows.
- Do **not** assume `packages/` provides the main local debugging entrypoint.

### 4. Dual-Target Packaging

- Most tool crates may eventually have a corresponding browser/JS-facing package under `packages/`.
- Those packages are for browser usage, npm publishing, wrappers, or ecosystem integration.
- Do **not** treat those packages as the primary CLI implementation unless explicitly required.

### 5. Shared Crate Rules

- Shared crates are allowed for reusable infrastructure and low-level helpers.
- Do **not** place end-user command logic inside shared crates.
- Do **not** couple tools through implicit global state; dependencies between crates must stay explicit.

### 6. Do Not Default to Node CLI

- Do **not** generate `packages/cli` as the main CLI by default.
- Do **not** assume local CLI debugging depends on Node.js.
- Introduce Node.js only when it is specifically needed for package builds, tests, browser-facing wrappers, or explicit ecosystem integration.

## 🦀 Rust & Wasm Coding Rules

### 1. Multi-Crate Isolation

- Keep crates inside `crates/` decoupled unless explicit dependencies are declared.
- Expose public JavaScript-facing bindings with `#[wasm_bindgen]` when a crate is intended for Wasm/browser consumption.

### 2. Memory Boundary & Performance

- Minimize heavy serialization across the JS/Rust boundary.
- Prefer primitive values and typed arrays when possible.
- If complex data transfer is required, use `serde-wasm-bindgen` deliberately and note the overhead tradeoff.

### 3. Safety & Error Handling

- Never use `panic!()`, `unwrap()`, or `expect()` in production Wasm logic.
- Return `Result<T, JsValue>` and convert Rust errors into JS-friendly messages with `.map_err(|e| JsValue::from_str(&e.to_string()))`.

## 🟢 JavaScript / Package Layer Rules

- `packages/` should stay focused on browser-facing packages, npm publishing surfaces, and thin wrappers around Rust/Wasm functionality.
- Do not move core tool logic into JavaScript packages when it belongs in Rust crates.
- When Node.js is used around Wasm artifacts, keep that code as thin glue rather than the primary implementation layer.

## 💡 Code Generation Style

- **Rust**: Write clean, idiomatic, memory-safe Rust with explicit types when they improve clarity.
- **JavaScript/TypeScript**: Generate modern ECMAScript Modules (ESM) using `import`/`export` syntax and explicit async error handling.
