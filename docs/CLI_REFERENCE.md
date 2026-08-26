# Zuv CLI Tooling & Package Manager Reference (`zuv`)

The `zuv` command-line tool manages Zuv projects, compilation, static checks, formatting, and unit testing.

---

## Command Overview

```bash
zuv <command> [options]
```

| Command | Usage | Description | Status |
| :--- | :--- | :--- | :--- |
| `build` | `zuv build [file.zv]` | Compile Zuv source file into a native binary or `.dll`/`.so`/`.dylib`. | **Implemented** |
| `run` | `zuv run [file.zv]` | Compile and execute the Zuv program immediately. | **Implemented** |
| `check` | `zuv check [file.zv]` | Perform fast syntax analysis and borrow checking without emitting binary. | **Implemented** |
| `checkall`| `zuv checkall` | Run batch syntax & borrow checks across all test files. | **Implemented** |
| `test` | `zuv test` | Run full automated unit test suite. | **Implemented** |
| `init` | `zuv init [project_name]` | Initialize a new project with `zuv.yml` and starter `src/main.zv`. | **Stub** |
| `fmt` | `zuv fmt [file.zv]` | Auto-format Zuv source code indentation. | **Partial** |
| `lsp` | `zuv lsp` | Launch Language Server Protocol (JSON-RPC) server for editor integration. | **Stub / Planned** |
| `--help` | `zuv --help` | Display CLI usage summary. | **Stub** |

---

## Detailed Command Manual

### 1. `zuv init`
Creates a standard directory layout and `zuv.yml` manifest file:
```bash
zuv init my_app
```
**Generated Files:**
- `zuv.yml`
- `src/main.zv`

---

### 2. `zuv check`
Checks a file for syntax or borrow checker errors without running Clang/LLVM codegen:
```bash
zuv check src/main.zv
```

---

### 3. `zuv build`
Bundles required module imports recursively, generates LLVM IR (`output.ll`), and compiles to a native Windows PE binary (`output.exe`) or shared dynamic library (`.dll` / `.so` / `.dylib`):
```bash
# Build standalone executable
zuv build src/main.zv

# Build release-optimized binary (-O3)
zuv build src/main.zv --release -o myapp.exe

# Build native shared C library (.dll / .so / .dylib)
zuv build math.zv --cdylib -o math.dll
```

**Options:**
- `-o, --output <file>`: Specify output filename (defaults to `output.exe` or `output.dll`).
- `--release, -r, -O3`: Enable release-mode optimizations (-O3 Native).
- `--cdylib, --lib`: Compile as a shared dynamic library with C export symbol table (`.dll` / `.so` / `.dylib`).
- `--emit-llvm, -S`: Emit textual LLVM IR (`output.ll`).

---

### 4. `zuv run`
Builds the project and executes the compiled executable directly:
```bash
zuv run src/main.zv
# Or run shortcut:
zuv src/main.zv
```

---

### 5. `zuv test`
Inbuilt, zero-dependency Vitest-style test runner. Automatically scans the current project strictly for `*.test.zv` test files, compiles each in-process, runs tests, and reports duration with formatted output:
```bash
# Run all tests in the project
zuv test

# Run tests matching a specific pattern or file
zuv test math.test.zv
zuv test str
```

---

### 6. `zuv checkall` *(Implemented)*
Fast batch type checker that scans all `.zv` and test files in the project, checking AST parsing and borrow checker rules without binary emission:
```bash
zuv checkall
```

---

### 7. `zuv fmt` *(Partial)*
Formats `.zv` source files to conform to standard 4-space indentation:
```bash
zuv fmt src/main.zv
```

---

### 8. `zuv lsp` *(Stub / Planned)*
Launches the Language Server Protocol (JSON-RPC) server over `stdin`/`stdout` for IDE integrations (VS Code extension syntax diagnostics, hover types, and completion). Currently a planned stub.
```bash
zuv lsp
```
