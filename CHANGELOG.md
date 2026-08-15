# Changelog

All notable changes to the **Zuv** programming language and self-hosting compiler are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.0.2] - 2026-08-15

### In-Process LLVM Compiler & CLI Toolchain
- **100% Pure Self-Hosting**: Complete compiler implementation in pure Zuv (`tokens.zv`, `lexer.zv`, `parser.zv`, `checker.zv`, `codegen.zv`, `cli.zv`, `main.zv`).
- **In-Process LLVM-C Runtime**: Direct machine code generation and linking via in-process `LLVM-C` and `lld-link` without requiring external compiler driver executables.
- **Optimization Modes**: The compiler supports debug and `-O3` optimized release builds internally.
- **Current Entry Point**: The self-hosting compiler currently builds `src/main.zv` in release mode. Command-line subcommands are not implemented yet.

## [0.0.1] - 2026-08-15

### C++ Bootstrap Release
- Version 0.0.1 is built with the C++ compiler and distributed as the bootstrap compiler for Zuv.

### 🚀 Initial Public Release of Zuv (`.zv`)

This release marks the debut of the **Zuv** programming language—a modern, statically-typed, memory-safe compiled language featuring a revolutionary chat-like, parenthesis-free direct syntax and high-performance LLVM backend.

### 🌟 Language Features & Syntax
- **"Write Less, Do More" Design**: Minimalist keywords (`prnt`, `mut`, `ret` or `->`, `obj`, `wh`, `fr`, `brk`, `cont`, `asc`, `awt`, `spn`, `jn`, `wrk`).
- **Parenthesis-Free Declarations**: Functions declared without enclosing parameter parentheses (`add a, b { -> a + b }`).
- **Direct Parenthesis-Free Invocation**: Zero-ceremony function and command calls (`factorial 5`, `handleBuild "src/main.zv", 1, 0`).
- **Zero-Argument Direct Calls**: Call parameterless functions directly by name (`incrementCounter`, `resetCounter`).
- **Type Inference & Static Typing**: Strongly-typed variables with smart compile-time type deduction.
- **Objects & Structs (`obj`)**: Named struct declarations with properties and methods, plus anonymous object literals (`{ x: 10, y: 20 }`).
- **Generics**: Monomorphized generic functions and data types with bracket syntax (`identity[T] item { -> item }`).
- **Result Types & Error Handling**: Built-in `ok` and `err` constructors.
- **Pattern Matching (`mch`)**: Exhaustive pattern matching over Result values and expressions (`mch res { ok v => prnt v, err e => prnt e }`).

### 🛡️ Memory Safety & Ownership
- **Compile-Time Static Borrow Checker**: Rust-grade ownership model enforcing memory safety with zero garbage collection overhead.
- **Immutable by Default**: Variables are immutable unless explicitly marked with `mut`.
- **References & Borrows**: Immutable borrows (`&x`) and mutable borrows (`&mut x`) with lifetime validation.

### ⚡ Concurrency & Multithreading
- **Native OS Multithreading**: Win32 / POSIX thread creation and joining via `spn` and `jn`.
- **Worker Task Declarations**: `wrk` keyword for clean thread routine definitions (`wrk workerTask { ... }`).
- **Native Async / Await**: Asynchronous coroutine execution with `asc` and `awt` keywords.

### 📚 Standard Library (`std/*`)
- **Single-Line Multi-Module Imports**: Import multiple libraries concisely (`imp str, arr, time, fs, thr`).
- **Selective Symbol Imports**: Import specific functions directly (`imp addNumbers frm math_helper`).
- **String Utilities (`std/str`)**: Dot-method calling with `.len`, `.contains`, `.substr`, `.charAt`, `.concat`, `.slice`, `.eq`.
- **Dynamic Arrays (`std/arr`)**: Dynamic heap arrays with `.push`, `.pop`, `.len`, and bracket indexing `arr[i]`.
- **File System Operations (`std/fs`)**: Dynamic file reading (`rF`), file writing (`wF`), and file existence checking (`fE`).
- **Date & Time (`std/time`)**: Millisecond clock ticks (`nw`) and thread sleep (`sl`).
