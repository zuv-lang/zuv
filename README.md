<div align="center">

# Zuv (`.zv`) ⚡
### *Write Less, Do More. Code Like You Chat.*

[![GitHub Repo](https://img.shields.io/badge/GitHub-zuv--lang%2Fzuv-792ee5?style=for-the-badge&logo=github)](https://github.com/zuv-lang/zuv)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)
[![Compiler-Backend](https://img.shields.io/badge/Backend-LLVM%20In--Process-orange.svg?style=for-the-badge)](https://llvm.org/)
[![Pure Self-Hosting](https://img.shields.io/badge/Self--Hosting-100%25%20Zuv-success.svg?style=for-the-badge)](src/)

<br/>

**Zuv** is a modern, high-performance, statically-typed compiled programming language built on top of LLVM. It is designed around a single revolutionary motive: **strip away unnecessary syntax and ceremony so developers can express logic as quickly and naturally as sending a text message on social media.**

</div>

---

## 🎯 Our Motive: *Write Less, Do More*

Traditional programming languages force you to write excessive boilerplate—parentheses around function calls, noisy argument separators, verbose function headers, and cumbersome type ceremony. 

In **Zuv**, we believe:
> **"Typing code should feel as fast, direct, and natural as chatting on a social media app."**

- 💬 **Zero Unnecessary Parentheses**: Direct declarations (`add a, b { -> a + b }`) and direct calls (`factorial 5`, `handleBuild "main.zv", 1, 0`).
- ⚡ **Minimalist Keywords**: `prnt`, `mut`, `ret` or `->`, `obj`, `wh`, `fr`, `brk`, `cont`, `asc`, `awt`, `spn`, `jn`, `wrk`.
- 🛡️ **Memory Safety Without GC**: Rust-grade static borrow checker (`&` and `&mut`) preventing memory bugs with zero runtime latency.
- 🚀 **Blazing Fast Native Code**: Compiled Ahead-of-Time (AOT) to optimized machine binaries via an in-process LLVM engine.
- 🔄 **100% Pure Self-Hosting**: The compiler in this repository is written completely in pure Zuv (`.zv`).

---

## 🆚 Code Comparison: Traditional vs. Zuv

### 1. Function Definition & Invocation
```zuv
// Traditional (C/JS style)
function calculateTotal(price, tax, discount) {
    return (price + tax) - discount;
}
let total = calculateTotal(100, 15, 5);

// 🔥 Zuv (Clean, Direct, Chat-like)
calculateTotal price, tax, discount {
    -> price + tax - discount
}
total = calculateTotal 100, 15, 5
```

### 2. Multi-Module Import & Dot Method Syntax
```zuv
// Single-line import & fluent dot methods
imp str, arr, time, fs

main {
    prnt "Welcome to Zuv!"

    // Dynamic arrays without parenthesis noise
    mut items = [10, 20]
    items.push 30
    lg items.len

    // String manipulation
    msg = "Hello World"
    if msg.contains "World" {
        lg msg.len
    }
}
```

### 3. Concurrency & Async in One Line
```zuv
wrk workerTask {
    lg "Background thread running..."
}

main {
    h = spn workerTask
    jn h
    prnt "Thread finished!"
}
```

---

## 🌟 Key Features

| Feature | Description |
| :--- | :--- |
| **Parenthesis-Free Syntax** | Clean function declarations, command calls, and expressions. |
| **Static Borrow Checker** | Compile-time memory ownership (`&`, `&mut`) with zero GC overhead. |
| **In-Process LLVM Engine** | Direct machine code emission via `LLVM-C` library without external compiler dependencies. |
| **Fluent Dot Methods** | Built-in `.len`, `.push`, `.pop`, `.contains`, `.substr` for strings and arrays. |
| **Native Multithreading** | OS thread creation (`spn`) and synchronization (`jn`) with `wrk` task definitions. |
| **Async / Await** | Asynchronous coroutines with `asc` and `awt`. |
| **Self-Hosting Compiler** | Builds the Zuv compiler source through its current self-hosting entry point. |

---

## 📁 Repository Structure

```text
zuv/
├── zuv.yml                 # Zuv project configuration
├── README.md               # Main documentation
├── CHANGELOG.md            # Release history and updates
├── CONTRIBUTING.md         # Developer contribution guidelines
├── CODE_OF_CONDUCT.md      # Contributor Covenant v2.1
├── LICENSE                 # MIT License
└── src/
    ├── tokens.zv           # Token definitions and keyword map
    ├── lexer.zv            # Pure Zuv source tokenizer & scanner
    ├── parser.zv           # Recursive descent AST parser
    ├── checker.zv          # Compile-time borrow checker & safety validator
    ├── codegen.zv          # LLVM IR emitter & in-process C-API bindings
    ├── cli.zv              # Compiler build and validation helpers
    └── main.zv             # Compiler entry point
```

---

## 🛠️ Building & Running Zuv

### Build the Self-Hosting Compiler

The current entry point builds `src/main.zv` in release mode. Command-line commands such as `zuv build` and `zuv test` are not implemented yet.

### Run the Generated Binary
```bash
.\output.exe
```

---

## 🤝 Contributing

We welcome contributions from developers worldwide! Please review our [Contributing Guide](CONTRIBUTING.md) and [Code of Conduct](CODE_OF_CONDUCT.md) before submitting pull requests.

1. Fork the repo: [https://github.com/zuv-lang/zuv](https://github.com/zuv-lang/zuv)
2. Create your branch: `git checkout -b feature/my-new-feature`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push to branch: `git push origin feature/my-new-feature`
5. Open a Pull Request!

---

## 📜 Documentation & Links

- 📖 **[Changelog](CHANGELOG.md)** - Release notes and version history.
- 💡 **[Contributing Guide](CONTRIBUTING.md)** - How to get involved.
- 🛡️ **[Code of Conduct](CODE_OF_CONDUCT.md)** - Our community standards.
- 🧰 **[Zuv Developer Tools & LSP](https://github.com/zuv-lang/zuv-tools)** - VS Code extension and language server.

---

## 📄 License

This project is licensed under the terms of the **[MIT License](LICENSE)**.
