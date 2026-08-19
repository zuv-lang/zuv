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
    items.psh 30
    lg items.ln

    // String manipulation
    msg = "Hello World"
    if msg.contains "World" {
        lg msg.ln
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

## 🧰 CLI Commands & Subsystem Status

| Command | Usage | Description | Implementation Status |
| :--- | :--- | :--- | :---: |
| `test` | `.\output.exe test` | Run automated native AOT unit test runner across all 27 test files | ✅ **Fully Implemented** |
| `checkall` | `.\output.exe checkall` | Run project-wide static syntax & borrow checker without compiling | ✅ **Fully Implemented** |
| `check` | `.\output.exe check [file.zv]` | Run fast lexing, parsing, and borrow checker on a single file | ✅ **Fully Implemented** |
| `build` | `.\output.exe build [file.zv]` | Compile Zuv source to native LLVM IR and executable binary (`.exe`) | ✅ **Fully Implemented** |
| `run` | `.\output.exe run [file.zv]` | Compile and immediately execute the target binary | ✅ **Fully Implemented** |
| `fmt` | `.\output.exe fmt [file.zv]` | Auto-format `.zv` source code with standard indentation | 🟡 *Stub* |
| `init` | `zuv init [project_name]` | Scaffold a new Zuv project with `zuv.yml` and `src/main.zv` | 🟡 *Stub* (in `zuv.exe`) |
| `lsp` | `zuv lsp` | Language Server Protocol (JSON-RPC) daemon for IDEs | 🟡 *Stub* (in `zuv-tools`) |

---

### 🔍 Self-Hosting Subsystems: Implemented vs. Stubs

- **Tokenizer / Lexer (`src/lexer.zv`)**: ✅ **Fully Implemented** — Line/column tracking, all keywords, strings, floats, operators.
- **Recursive Descent Parser (`src/parser.zv`)**: ✅ **Fully Implemented** — Full AST node hierarchy, Pratt expression precedence climber, imports, functions, structs, control flow.
- **Borrow & Safety Checker (`src/checker.zv`)**: ✅ **Fully Implemented** — Scoped symbol table, immutability enforcement, Rust-grade ownership movement and borrow exclusivity validation.
- **LLVM IR Codegen (`src/codegen.zv`)**: ✅ **Fully Implemented** — Unified 64-bit calling convention, dynamic array allocation, struct offset mapping, math/comparison ops, control branching.
- **Native AOT Test Runner (`src/cli.zv`)**: ✅ **Fully Implemented** — Dynamic directory scanning (`sF`), automated compilation, execution, timer diagnostics, negative test assertions.
- **Code Formatter (`handleFmt` in `src/cli.zv`)**: 🟡 **Stub** — Logs formatting confirmation; AST token-level formatter planned for next milestone.
- **Project Scaffolding (`init`)**: 🟡 **Stub** in self-hosting compiler (functional via C++ bootstrap `zuv.exe`).
- **Language Server (`lsp`)**: 🟡 **Stub** in self-hosting compiler (functional via TypeScript LSP in `sub_projects/zuv-tools`).

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
├── tests/                  # Automated Test Suite (27 test files)
└── src/
    ├── tokens.zv           # Token definitions and keyword map
    ├── lexer.zv            # Pure Zuv source tokenizer & scanner
    ├── parser.zv           # Recursive descent AST parser
    ├── checker.zv          # Compile-time borrow checker & safety validator
    ├── codegen.zv          # LLVM IR emitter & in-process C-API bindings
    ├── cli.zv              # CLI driver and AOT test runner
    └── main.zv             # Compiler entry point & command dispatcher
```

---

## 🛠️ Building & Running the Self-Hosting Compiler

### 1. Compile the Self-Hosting Compiler
Compile the pure Zuv compiler sources using the stage 1 bootstrap binary:
```powershell
d:\rujs\zuv.exe build src/main.zv
```
This produces `output.exe` in `sub_projects/zuv/`.

### 2. Run the Native AOT Test Runner
Execute all 27 unit test suites natively:
```powershell
.\output.exe test
```

### 3. Run Static Type & Borrow Checker
Verify syntax and borrow safety across all test suites without emitting binaries:
```powershell
.\output.exe checkall
```

### 4. Build and Run a Zuv Program
```powershell
.\output.exe build tests/functions.test.zv
.\output.exe run tests/functions.test.zv
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
