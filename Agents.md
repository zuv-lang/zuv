# Agents.md — Self-Hosting Compiler Guide (sub_projects\zuv)

## Folder Scope
This guide applies to **D:\rujs\sub_projects\zuv** — the self-hosting compiler project. 
It complements the top-level `Agents.md` by focusing on pure Zuv compiler development, 
grammar implementation, and self-hosting workflows.

**Key binary:** `zuv_selfhost.exe` — built from the Zuv source in this folder.
**Bootstrap compiler:** `zuv.exe` (located at `D:\rujs\sub_projects\zuv\zuv.exe` or `..\..\zuv.exe`) — 
used to build the self-host compiler.

---

## Project Structure (sub_projects\zuv\)

```
sub_projects\zuv\
├── src/                      # Zuv compiler source files (.zv)
│   ├── tokens.zv             # Token definitions & classification
│   ├── diagnostics.zv        # Error formatting & diagnostics
│   ├── lexer.zv              # Lexer/scanner implementation
│   ├── main.zv               # CLI entry point
│   ├── checker.zv            # Borrow checker & semantic validation
│   ├── cli.zv                # Command-line interface handlers
│   └── codegen.zv            # LLVM code generation
├── tests/                    # Test suites
│   ├── *.test.zv             # Runtime test files (run with `zuv test`)
│   ├── *.zv                  # Semantic check test files (run with `zuv checkall`)
│   └── my_helper.zv          # Helper/test utilities
├── build/                    # Build output
│   ├── zuv_selfhost.exe      # Self-hosting compiler binary
│   ├── zuv.exe               # C++ bootstrap compiler binary
│   ├── *.lib, *.obj          # Linker/output artifacts
│   └── output_selfhost.ll    # LLVM IR output (if --emit-llvm)
├── docs/                     # Language/documentation files
├── CMakeLists.txt            # Sub-project CMake configuration
├── zuv.yml                   # Project configuration
└── output_temp.o, test*.lib  # Temporary build artifacts
```

---

## Core Source Files

### `src/tokens.zv`
Token definitions and classification functions:
- `createToken`, `isTokenKind`, `isTokenVal`
- `isKeywordTok`, `isIdentTok`, `isNumberTok`, `isStringTok`, `isBoolTok`, `isSymbolTok`, `isEOFTok`

### `src/lexer.zv`
*(Not fully read yet — typically contains scan/tokenize functions)*

### `src/main.zv`
CLI entry point with command handlers:
- `handleBuild` — Build a Zuv source file
- `handleCheck` — Semantic check a file
- `handleCheckAll` — Check all test files in `tests/`
- `handleTest` — Run all `.test.zv` test files
- `handleRun` — Build and run a file
- `handleFmt` — Format a file

### `src/parser.zv`
Recursive descent parser with:
- Precedence climbing algorithm
- Expression parsing (`parseExpression`)
- Statement parsing (`parseStatement`, `parseBlock`)
- Pattern matching (`mch`/`sw`)
- Control flow (`fr ... of`, `fr ... in`)
- Exception handling (`try`, `cth`, `fin`, `thr`)
- Import handling (`imp ... frm`)
- Extern/FFI declarations
- Macros (`@test`, `@inline`, `macro`)

### `src/checker.zv`
Borrow checker & semantic validator:
- Variable declaration checking
- Scope rules & duplicate detection
- Type safety
- Borrow/lifetime rules
- Error format matching (E0001-E9999)

### `src/cli.zv`
Low-level CLI & LLVM integration:
- `extern` declarations for LLVM C API functions
- `emitDirectObjectFile` — Generate .obj from IR
- `linkDirectObjectFile` — LLD linking
- `handleCheckAll` — Iterate over test files
- `handleTest` — Run test suite with per-file temp executables
- `emitRes`, `linkRes` — Result handlers

### `src/codegen.zv`
*(Likely contains LLVM IR code generation functions)*

---

## Testing Framework

### Test File Conventions
- **`.test.zv`** files — Runtime tests, run with `zuv test` or `zuv_selfhost.exe test`
- **`.zv`** files (in `tests/`) — Semantic/check tests, run with `zuv checkall` or `zuv_selfhost.exe checkall`

### Running Tests

**Using the self-host compiler:**
```powershell
# Check all semantic/test files
.\zuv_selfhost.exe checkall

# Run all runtime tests
.\zuv_selfhost.exe test
```

**Using the C++ bootstrap compiler:**
```powershell
# From project root
..\..\zuv.exe checkall
..\..\zuv.exe test
```

### Test Discovery
`handleTest` in `cli.zv` discovers test files via:
```zuv
let tests = sF "tests", ".test.zv"
```
Searches the `tests/` directory for `*.test.zv` files.

`handleCheckAll` discovers check files via:
```zuv
let tests = sF "tests", ".zv"
```
Searches the `tests/` directory for `*.zv` files (excluding `.test.zv`).

---

## Building the Self-Host Compiler

### From Source

```powershell
# 1. Ensure the C++ bootstrap compiler is available
#    (zuv.exe at D:\rujs\sub_projects\zuv\zuv.exe or in PATH)

# 2. Build self-host compiler from source
zuv build src/main.zv -o zuv_selfhost.exe

# 3. Verify the build
.\zuv_selfhost.exe --help
# or
.\zuv_selfhost.exe checkall
```

### Using CMake (sub_projects\zuv\)
```powershell
cd D:\rujs\sub_projects\zuv
mkdir build
cd build
cmake -G "Visual Studio 17 2022" ..  # Or your VS version generator
cmake --build . --config Release
```

### Build Artifacts
After building, you'll have:
- `zuv_selfhost.exe` — The self-hosting compiler
- `zuv.exe` — The C++ bootstrap (also present from prior build)
- `.lib`/`.obj` files for linker
- `output_selfhost.ll` (if compiled with `--emit-llvm`)

---

## Grammar Mirroring Workflow

When implementing new features, you often need to mirror changes between the C++ bootstrap 
and the self-host Zuv compiler.

### When to Mirror
- **Phase 1 features** added to C++ bootstrap → Must mirror to Zuv sources
- **Phase 2 features** → Pure Zuv, no mirroring needed (or optional)

### Mirroring Targets (in `src/`)

| C++/Bootstrap Feature | Zuv Source to Update |
|----------------------|---------------------|
| New token kinds | `tokens.zv` — add classification functions |
| New lexer patterns | `lexer.zv` — scanner/tokenize rules |
| New grammar productions | `parser.zv` — expression/statement rules |
| New borrow checker rules | `checker.zv` — validation logic |
| New LLVM IR patterns | `codegen.zv` — code generation |
| New CLI commands | `main.zv` / `cli.zv` — command handlers |

### Mirroring Steps
1. **Add to C++ bootstrap** (if Phase 1 work)
2. **Add equivalent to Zuv source** in `sub_projects\zuv/src/`
3. **Run tests** to verify both work:
   ```powershell
   .\zuv.exe checkall       # C++ bootstrap
   .\zuv_selfhost.exe checkall  # Self-host
   ```
4. **Verify grammar consistency** — ensure both compilers parse the same constructs

---

## Common Self-Host Development Tasks

### Adding a New Zuv Language Feature

1. **Define the syntax** — Decide on the Zuv syntax for the new feature
2. **Update `tokens.zv`** — Add token classification if needed
3. **Update `parser.zv`** — Add parsing rules (precedence, infix/prefix, etc.)
4. **Update `checker.zv`** — Add semantic validation if needed
5. **Update `codegen.zv`** — Add LLVM IR generation if needed
6. **Add tests** — Create `.test.zv` file(s) in `tests/`
7. **Build and test:**
   ```powershell
   .\zuv_selfhost.exe checkall    # Semantic check
   .\zuv_selfhost.exe test        # Runtime tests
   ```

### Adding a New Standard Library Module

1. **Create the module file** — e.g., `std/math.zv` or add to existing std library
2. **Imp the module** in user code: `imp math frm "std/math"`
3. **Add implementations** — Functions, types, constants
4. **Export as `pub`** if external use is desired
5. **Add tests** — Create test file: `tests/math.test.zv`
6. **Run tests:**
   ```powershell
   .\zuv_selfhost.exe test
   ```

### Debugging Self-Host Compiler Issues

- **Use `prnt`** for console debug output (same as user-facing output)
- **Use `lg`/`wrn`/`err`** for diagnostic-level output
- **Run `zuv checkall`** to find semantic errors
- **Run `zuv test`** to find runtime test failures
- **Check LLVM IR** — Use `--emit-llvm` flag:
  ```powershell
  .\zuv_selfhost.exe build myfile.zv --emit-llvm -o myfile.ll
  ```
- **Inspect error format:** `error[EXXXX]: message --> file.zv:line:col`

### Working with the Bootstrap Compiler (`zuv.exe`)

Even when working primarily in the self-host folder, you'll interact with `zuv.exe`:

```powershell
# Build a Zuv source file with the bootstrap compiler
.\zuv.exe build src/main.zv -o myapp.exe

# Check it compiles correctly
.\zuv.exe checkall

# Run the compiled binary
.\myapp.exe

# Or use the self-host compiler after building it
.\zuv_selfhost.exe build src/main.zv -o myapp.exe
```

---

## Phase 1 vs Phase 2 Development Rules

### Phase 1 (Tier 1) — C++ Bootstrap Side
- Work happens in C++ sources (outside this folder, in `D:\rujs\`)
- Features: syntax primitives, parser, checker, LLVM codegen, FFI, linking
- Goal: Stable bootstrap that can compile the self-host compiler

### Phase 2 (Tier 2) — Pure Zuv Self-Host
- Work happens in `sub_projects\zuv/src/` and `sub_projects\zuv/tests/`
- Features: standard library modules, higher-level abstractions
- Goal: 100% pure Zuv self-hosting, eliminate C++ dependency

**Rule of thumb:** If you're modifying grammar/parser/checker → likely Phase 1 (mirror to Zuv). 
If you're adding standard library functionality → Phase 2 (no mirroring needed).

---

## Quick Start for Self-Host Agents

### 1. First Time Setup
```powershell
# Ensure C++ bootstrap is available
cd D:\rujs
# If zuv.exe doesn't exist, build it first:
# mkdir build && cd build
# cmake -G "Visual Studio 17 2022" ..
# cmake --build . --config Release

# Verify bootstrap compiler works
.\sub_projects\zuv\zuv.exe checkall

# Build self-host compiler from source
cd D:\rujs\sub_projects\zuv
zuv build src/main.zv -o zuv_selfhost.exe

# Verify self-host compiler works
.\zuv_selfhost.exe checkall
```

### 2. Daily Workflow
```powershell
# Make changes to .zv source files in src/ or tests/

# Run semantic check
.\zuv_selfhost.exe checkall

# Run runtime tests
.\zuv_selfhost.exe test

# If grammar changes, also verify with bootstrap compiler
..\..\zuv.exe checkall
```

### 3. Common Commands Reference

| Action | Command |
|--------|---------|
| Build self-host compiler | `zuv build src/main.zv -o zuv_selfhost.exe` |
| Check all semantic errors | `.\zuv_selfhost.exe checkall` |
| Run all runtime tests | `.\zuv_selfhost.exe test` |
| Build single file | `.\zuv_selfhost.exe build file.zv -o output.exe` |
| Emit LLVM IR | `.\zuv_selfhost.exe build file.zv --emit-llvm -o file.ll` |
| Check single file | `.\zuv_selfhost.exe check file.zv` |
| Run single test file | Build then execute the temp .exe, or use `zuv test` filter |

---

## File Conventions Specific to This Folder

- **Source files:** `src/*.zv` — Core compiler implementation
- **Test files:** `tests/*.test.zv` — Runtime tests (`.test.zv` extension)
- **Check files:** `tests/*.zv` — Semantic/check tests (without `.test` suffix)
- **Module files:** `std/*.zv` — Standard library modules
- **Configuration:** `zuv.yml` — Project settings
- **Build output:** `build/` — Compiler binaries and artifacts

---

*This guide is specific to D:\rujs\sub_projects\zuv and should be used alongside the top-level 
Agents.md for complete project context. Update as the self-host compiler evolves.*