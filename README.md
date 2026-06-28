# Wasm Tools Monorepo

A dual-target-first monorepo for Rust-based tools that can be delivered both as WebAssembly packages and through a centralized native Rust CLI.

## 📂 Architecture

This repository is designed for **multiple tools in one project**:

- Each tool has its own Rust implementation under `crates/`.
- Most tools are expected to have a corresponding browser/JavaScript-facing package under `packages/`.
- There is **exactly one CLI entrypoint**, implemented as a **native Rust CLI**, and it exposes tools as **subcommands**.
- Shared low-level logic may live in dedicated shared crates, but end-user command behavior belongs to tool-specific crates.

## 📁 Recommended Structure

```text
├── crates/                    # 🦀 Rust workspace members
│   ├── cli/                   # The only native Rust CLI application
│   ├── tool_a/                # Tool A Rust implementation
│   ├── tool_b/                # Tool B Rust implementation
│   └── shared/                # Optional shared low-level utilities
├── packages/                  # 📦 Browser/JS-facing wrappers or published packages
│   ├── tool-a/                # JS/Wasm wrapper for tool_a
│   └── tool-b/                # JS/Wasm wrapper for tool_b
├── Cargo.toml                 # Rust workspace configuration
├── package.json               # Optional JS workspace configuration
└── Makefile                   # Build and orchestration tasks
```

## 🧭 Structural Rules

- `crates/cli` is the **only** CLI application in this repository.
- Do **not** create additional standalone CLI apps for other tools unless explicitly requested.
- Each end-user tool should appear as a **subcommand** of the centralized native CLI.
- Tool business logic belongs in its own Rust crate, not in the CLI crate.
- `packages/` is for browser/npm-facing packaging and glue code, not for the primary local debugging entrypoint.
- Shared crates are allowed, but should only contain reusable infrastructure or low-level helpers.

## 🛠️ Prerequisites

Ensure you have the following tools installed locally before developing:

- **Rust**: Latest stable toolchain (`rustup`)
- **wasm-pack**: Wasm compiler for browser/package builds (`cargo install wasm-pack`)
- **Node.js**: Version 18 or higher (for package tooling and browser-facing wrappers)

## 🚀 Development Model

Recommended workflow:

- Use the **native Rust CLI** in `crates/cli` for local command-line debugging.
- Keep each tool’s reusable logic inside its own Rust crate.
- Build browser-facing Wasm packages from the corresponding tool crates when JavaScript or browser consumption is needed.
- Keep local CLI concerns and browser packaging concerns separated.

As the repository grows, the top-level `Cargo.toml`, optional `package.json`, and `Makefile` should orchestrate workspace-wide build, test, packaging, and CLI workflows.
