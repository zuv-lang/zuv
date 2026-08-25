# Changelog

All notable changes to the **Zuv** programming language and self-hosting compiler are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### ⚡ Operators
- **Increment & Decrement (`++` / `--`)**: Prefix and postfix on variables (`++x`, `x++`, `--x`, `x--`). New test: `incr_decr.test.zv`.
- **Bitwise (`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`)**: Integer bitwise ops via `fptosi`/`sitofp`; prefix `&` remains borrow. New test: `bitwise.test.zv`.
- **Explicit Cast (`as`)**: `expr as int|num|bool|str|*byte|ptr` via `fptosi`/`sitofp`/`bitcast`. New test: `as_cast.test.zv`.
- **Implicit String + Number (`+`)**: `"Hello " + 5` / `5 + "x"` coerce the number via `snprintf`; chained concat and string vars tracked. New test: `str_num_plus.test.zv`.
- **Compound Assignment (`+=` … `^=`)**: Desugars to `x = x op rhs` (incl. `**=`, `str +=`). Also fixed `%` → LLVM `frem`. New test: `compound_assign.test.zv`.
- **Strict `==` / `!=`**: Different known concrete types never coerce (`"5"==5`, `nil==0` false). Single-digit `0`/`1` are bool-compatible (`yes==1`, `true==1`). New test: `strict_eq.test.zv`.
- **Ternary (`? :`)**: `cond ? then : else` with right-associative nesting and branch short-circuit via alloca merge. New test: `ternary.test.zv`.

---

## [0.6.0] - 2026-08-25

### 🧱 Data Types
- **Type Inspection Operator (`typ`)**: Added prefix `typ` operator returning type strings (`"str"`, `"num"`, `"bool"`, `"obj"`, `"arr"`, `"nil"`, `"und"`, `"sym"`, `"bigint"`). Implemented in C++ bootstrap and self-hosting compiler. `und` literal now distinct from `nil` for `typ und`. New test: `typ.test.zv`.
- **Enums (`enum`)**: Named variants as integer discriminants (`0..n-1`). `Status.Active` property access, `mch` arms with qualified (`Status.Pending`) or bare (`Pending`) patterns. C++ bootstrap keeps native enum match; self-host desugars enum `mch` to nested `if`/`els`. Fixed self-host match-target parsing so `ident {` is not swallowed as a struct literal. New test: `enum.test.zv`.
- **Undefined Literal (`und`)**: Runtime-distinct from `nil` (LLVM `bitcast i64 1 to double` sentinel). Equality/`!=`, falsy `if`/`wh`/`for`/`and`/`or`/`!` via truthiness helper, and `typ und` → `"und"`. Fixed C++ `TOK_UND` over-consume and self-host checker allowing `let x = nil|und`. New test: `und.test.zv`.
- **Unique Symbol Primitive (`sym`)**: `sym "key"` creates an interned identity (same key → equal, different from strings). `typ (sym "k")` → `"sym"`. Symbols are truthy. Renamed self-host locals that collided with the new `sym` keyword. New test: `sym.test.zv`.
- **BigInt Primitives (`BigInt` / `...n`)**: Digit-string bigint literals (`123n`) and `BigInt(...)` constructor. Equality via content (`strcmp`); distinct from numbers (`123n != 123`). `typ` → `"bigint"`. New test: `bigint.test.zv`.
- **Scientific Exponent Literals (`123e5`, `1.5E-2`)**: Lexer support for `e`/`E` with optional `+`/`-` in C++ bootstrap and self-host. New test: `sci_notation.test.zv`.
- **Number Bases (`0x` / `0b` / `0o`)**: Hex, binary, and octal integer literals in C++ bootstrap and self-host. New test: `bases.test.zv`.
- **Special Numbers (`nan`, `inf` / `Infinity`)**: IEEE-754 NaN/+Inf literals; statement `inf expr` remains info logging. Helpers `isNan` / `isFin`. New test: `nan_inf.test.zv`.

### ⚡ Operators
- **Exponentiation (`**`)**: Right-associative `**` via `llvm.pow.f64`. New test: `pow.test.zv`.

### 🔤 Self-Hosting Source Style
- **Template Literals**: Migrated self-hosting compiler sources (`src/*.zv`) from `"..." + expr` concatenation and escaped `\"` strings to backtick templates with `${...}` interpolation (e.g. `` `tests/${impPath}.zv` ``). Plain string literals left unchanged.
- **Shortform Methods**: Replaced long-form std method calls with shortforms used in the test suite — `.concat` → `.cat`, `.charAt` → `.chr`.

---

## [0.5.0] - 2026-08-25

### 📦 Variables & Scope
- **`let` Keyword**: Added `let` keyword for mutable variable declarations, replacing/aliasing `mut`.
- **Duplicate Redeclaration Error**: Detects duplicate declarations in the same block scope while preserving child block shadowing (`{ let x = 2 }`).
- **Destructuring & Multi-Variable Assignments**: Added object (`{ name, age } = user`), array (`[x, y] = coords`), and multi-variable (`q, r = expr`) destructuring, plus multi-value returns (`-> a, b`), in both the C++ bootstrap and pure Zuv self-hosting compiler.
- **Project-Wide Global Object (`glb`)**: Added built-in `glb` object for cross-file runtime storage (`glb.appName = "rujs"`). Each property is registered as an LLVM module global (`@glb_<prop>`) with `load`/`store` in both the C++ bootstrap and self-hosting compilers.
- **Codebase Migration**: Migrated internal self-hosting compiler sources to use `let`.
- **New Tests**: Added `let_variables.test.zv`, `scope_redecl.test.zv`, `destructuring.test.zv`, and `glb.test.zv`.

### 💬 Comments
- **Multi-line Comments**: Added support for block comments (`/* ... */`) with newline tracking.
- **New Test**: Added `comments.test.zv` covering single-line, inline, multi-line, and expression comments.

---

## [0.4.0] - 2026-08-22

### 🔤 Strings & Lexer
- **Template Strings**: Added backticks (`` `...` ``) with multiline support and embedded quotes (`'`, `"`).
- **Interpolation**: Added `${...}` expression interpolation inside template strings.
- **String Indexing**: Added `s[i]` character indexing.
- **Tests**: Expanded `std_str.test.zv` to test template strings, `${...}`, and indexing.

### 🔌 C FFI (.lib)
- **Direct .lib Linking**: Added support for `.lib` in `extern` (e.g. `extern "kernel32.lib"`).
- **New Test**: Added `c_ffi_lib.test.zv`.

---

## [0.3.1] - 2026-08-22

### ⚡ Toolchain & Code Generation Improvements
- **Inline Pointer Dereference**: Replaced external `zuv_ptr_deref` helper with direct inline LLVM IR `load ptr, ptr %arg` instruction emission in both pure Zuv and C++ code generators.
- **Eliminated `Codegen.obj`**: Removed all intermediate C++ object dependencies from compiler builds, release packaging, and CI pipelines.
- **Bundled `lld-link.exe` Toolchain**: Packaged `lld-link.exe` alongside `zuv.exe` in release archives for portable standalone compilation.
- **Automatic Linker Discovery**: Added local `lld-link.exe` resolution in `cli.zv` to locate the bundled linker in the compiler directory before checking system `PATH`.

---

## [0.3.0] - 2026-08-21

### ⚡ Direct In-Process LLVM & LLD Linker
- **In-Memory Compilation & Linking**: Replaced external `clang` and `lld-link` subprocesses with direct LLVM C-API code emission and in-process `lld::coff::link`.
- **Zero-Dependency Monolithic Executable**: Statically linked `LLVM*.lib` and `lld*.lib` into a single standalone `zuv.exe` (~106 MB) with zero external DLLs.

### 🔌 C Foreign Function Interface (`extern` / `ffi`)
- **Native C ABI Declarations**: Added `extern "<lib>" <func> <params> -> <retType>` syntax in pure Zuv and C++ bootstrap compiler.
- **Direct LLVM C-API in Pure Zuv**: Declared and orchestrated target initialization, IR parsing, target machine creation, and object emission directly in `cli.zv`.
- **Dynamic Library Linking**: Automatically links Windows system libraries (`user32`, `kernel32`, `ntdll`, `ws2_32`) and custom `.lib` files.

### 🧪 Self-Hosting Verification
- **100% Pass Rate (28/28)**: Validated full test suite and type checker under the pure Zuv self-hosting compiler (`zuv test` & `zuv checkall`).

### 🚀 Pure Zuv Self-Hosting Compiler & 100% Test Suite Verification
- **100% Self-Hosting Test Suite Pass Rate (27/27 Tests)**:
  - Validated all 27 unit test suites in `tests/` running natively under the pure Zuv self-hosting compiler executable (`zuv test` / `C:\Users\rohit\zuv\bin\zuv.exe`).
  - Achieved seamless end-to-end self-hosting execution across all core language constructs, standard library modules, multithreading, concurrency, and error handling.

### 🛡️ Code Generation & Runtime Improvements
- **Directory Scanning Runtime (`sF` / `@zuv_scan_files`)**:
  - Implemented `@zuv_scan_files(ptr %dir, ptr %ext)` in pure Zuv code generator (`src/codegen.zv`) using Windows Win32 API (`FindFirstFileA`, `FindNextFileA`, `FindClose`, and `snprintf`), enabling zero-dependency directory scanning (`std_fs.test.zv`).
  - Added shortform standard library support for `sF`, `sD`, and `scanDir`.
- **Accurate Pointer & Float Conversions (`fptosi` vs `bitcast`)**:
  - Resolved floating-point bit pattern pointer corruption in `src/Codegen.cpp` across `getStrPtr`, `getStrPtrOrFormat`, `PrintStmt`, `PropertyAccess`, `IndexExpr`, `psh`, `pp`, and `ln` by replacing `bitcast double ... to i64` with `fptosi double ... to i64`.
- **Deterministic Struct Field Layout Synchronization**:
  - Pre-populated and synchronized `m_fieldOffsets` in `src/Codegen.h` and `fields` in `src/codegen.zv` to guarantee deterministic memory layouts between C++ bootstrap and self-hosting compilers.
- **String Variable Deduction (`isStrExpr`)**:
  - Extended string variable name heuristics in `src/Codegen.cpp` to recognize `val`, `arg`, and `lbl` identifiers for correct `@strcmp` comparison and `@printf` string formatting emission.

### 🧩 Parser & CLI Subsystem Fixes
- **Pattern Matching (`mch`) AST Parsing**:
  - Refactored `mch` statement parser in `src/parser.zv` to properly iterate match arm cases (`ok`, `err`) and block statements without token consumption desynchronization.
- **Parser Error Diagnostics & Exit Code Handling**:
  - Attached parser error collections (`p.errors`) directly to program AST in `parseProgramFull` (`src/parser.zv`).
  - Added compile-time syntax error diagnostic printing and failure exit code aborts in `src/cli.zv` (`semantic_errors.test.zv`).
- **Control Flow Loops & Generics**:
  - Fixed `ForStmt` increment code generator in `src/codegen.zv` to call `codegenStmt` and registered loop initialization variables in Pass 1.
  - Corrected generic function call bracket consumption and prefix array literal parsing in `src/parser.zv`.

### ⚙️ CI/CD & Release Workflow Automation (`.github/workflows/`)
- **Self-Hosting CI Pipeline (`.github/workflows/ci.yml`)**:
  - Upgraded CI to download the bootstrap compiler, compile the pure Zuv self-hosting compiler from source (`build src/main.zv`), and execute full `zuv check` and `zuv test` validation using the newly built self-hosting binary.
  - Added manual workflow dispatch and multi-branch triggers (`main`).
- **Automated Self-Hosting Release Packaging (`.github/workflows/release.yml`)**:
  - Fixed release artifact packaging to build the self-hosting compiler from source using the bootstrap compiler, validate the test suite, and bundle the genuine self-hosting binary into release archives (`.zip` / `.tar.gz`).

---

## [0.1.0] - 2026-08-19

### Added
- **Phase 1: Pure Zuv Lexer & Tokenizer Subsystem (`src/tokens.zv`, `src/lexer.zv`)**:
  - Complete Token definition with line and column tracking (`{ kind, val, line, col }`) and classification predicates (`isKeywordTok`, `isIdentTok`, `isNumberTok`, `isStringTok`, `isBoolTok`, `isSymbolTok`, `isEOFTok`).
  - Full tokenization support for all language keywords (`imp`, `frm`, `wrk`, `prnt`, `lg`, `wrn`, `inf`, `ret`, `els`, `wh`, `fr`, `brk`, `cont`, `dn`, `mch`, `ok`, `err`, `if`, `and`, `or`, `not`, `mut`, `asc`, `awt`, `obj`, `num`, `bool`, `str`, `main`).
  - Boolean literals (`yes`, `no`, `true`, `false`), Nil literals (`nil`, `und`), numbers (integer & float), and string literals with escape sequences (`\n`, `\t`, `\r`, `\"`, `\'`, `\\`, `\0`).
  - Single-line comment parsing (`// ... \n`) and multi-char/single-char operators (`==`, `=>`, `->`, `!=`, `<=`, `>=`, `&&`, `||`, `!`, `&`, `|`, `+`, `-`, `*`, `/`, `%`, `?`, `:`, `;`, `,`, `.`, `(`, `)`, `{`, `}`, `[`, `]`).
- **Phase 2: AST & Recursive Descent Parser Subsystem (`src/parser.zv`)**:
  - Formal AST Node constructor hierarchy for all expressions and statements (`NumberExpr`, `StringExpr`, `BoolExpr`, `NilExpr`, `VariableExpr`, `BinaryExpr`, `UnaryExpr`, `CallExpr`, `ArrayExpr`, `IndexExpr`, `PropertyAccess`, `StructLiteral`, `VarDecl`, `ReturnStmt`, `PrintStmt`, `IfStmt`, `WhileStmt`, `FunctionDecl`, `ImportStmt`, `BlockStmt`).
  - Pratt precedence climbing expression parser covering lowest, assignment, logical OR/AND, equality, relational, additive, multiplicative, unary, and call/index/dot access precedences.
  - Recursive descent statement parser for module imports (`imp a, b frm mod`), variable declarations (`mut` & immutable), `main` entry points with CLI argument parameters, struct declarations (`obj`), control flow (`if/els`, `wh`), returns (`ret`, `->`), and logging statements.
  - Parser syntax error collection and line-based diagnostic reporting.
- **Phase 3: Static Semantic & Borrow Checker Subsystem (`src/checker.zv`)**:
  - Lexical scope management with scope stack (`pushCheckerScope`, `popCheckerScope`) and symbol resolution.
  - Variable mutability enforcement (`mut` vs immutable), flagging illegal mutations on immutable variables.
  - Rust-grade ownership and borrowing validation rules:
    - Rejection of use or borrow of moved values (`Cannot borrow moved value`).
    - Immutable variables cannot be borrowed as mutable (`&mut`).
    - Mutual exclusion of simultaneous mutable and immutable borrows.
    - Single unique mutable borrow constraint enforcement.
  - Recursive semantic traversal and validation of statements and expressions.
- **Phase 4: LLVM IR Code Generator Subsystem (`src/codegen.zv`)**:
  - Module header emitting full C and Win32 runtime external declarations (`printf`, `malloc`, `free`, `memcpy`, `strcmp`, `strlen`, `fopen`, `fclose`, `CreateThread`, `WaitForSingleObject`).
  - In-process LLVM-C API declarations (`LLVMInitializeX86Target`, `LLVMParseIRInContext`, `LLVMTargetMachineEmitToFile`, `LLVMDisposeModule`).
  - Dynamic global string constant pool generation (`@str_N`).
  - Expression code generation: numbers, booleans, string pointer encoding, local register loading, arithmetic (`fadd`, `fsub`, `fmul`, `fdiv`), comparisons (`fcmp` with `uitofp`), assignments, and function call invocations.
  - Statement code generation: local allocations (`alloca`), store instructions, control flow branching (`br i1`), while loop headers/latches, return statements, and print formats (`@fmt_str`, `@fmt_num`).
  - Function code generator and `@main(i32 %argc, ptr %argv)` entry point assembler.
- **Phase 5: CLI Driver & Native Test Runner (`src/cli.zv`, `src/main.zv`)**:
  - Implemented pure Zuv CLI handlers: `handleBuild` (self-compilation to LLVM IR and executable linking), `handleCheck` (fast lex-parse-semantic check pipeline), `handleRun` (build and run), `handleFmt` (code formatting), and `handleCheckAll` (native self-hosting project-wide static & type checker).
  - Dynamic discovery and execution across all 26 positive unit test suites and 1 expected syntax/semantic error test suite (`tests/semantic_errors.test.zv`) with pass/fail summary statistics.
  - Complete CLI subcommand dispatcher in `src/main.zv` (`checkall`, `check`, `build`, `run`, `fmt`).
  - Enhanced runtime string and number formatting across property accesses and binary concatenations.
- **Phase 6: Self-Hosting Verification & Stage 2 Bootstrapping Validation**:
  - Validated pure Zuv self-hosting compiler compilation via Stage 1 compiler (`zuv build src/main.zv`).
  - Implemented native `std/fs` directory scanner `sF "dir", "ext"` for zero-configuration, dynamic test suite auto-discovery.
  - Verified 100% test pass rate across all 27 unit tests running inside `output.exe checkall`.
  - Completed stage 2 bootstrapping validation producing and verifying `output_stage2.exe`.
- **Phase 7: Native AOT Test Runner Subcommand (`test`)**:
  - Implemented `handleTest` native AOT test runner in pure Zuv matching `zuv test` CLI ergonomics.
  - Automatically compiles, executes, measures elapsed duration, and verifies exit codes for all 27 unit test suites in `tests/`.
  - Added support for negative test assertions (e.g. `tests/semantic_errors.test.zv`) checking expected compiler errors.

### Changed
- Replaced stubbed placeholder functions in `sub_projects/zuv` with genuine, modular compiler frontend components.
- Formatted AST constructors and function blocks to clean multi-line representations.

### Fixed
- Upgraded `isFunctionDeclaration()` in `src/Parser.cpp` to perform token-level lookahead, preventing string literals with braces (e.g., `"{"`) from triggering false function declaration parses.
- Implemented chained property access call and mutation support in `src/Parser.cpp` (`a.b.c = d`, `p.errors.psh(...)`).
- Fixed dynamic array literal memory allocation in `src/Codegen.cpp` with heap growth capacity buffer, preventing out-of-bounds corruption during `.psh` / `.push` operations (`std/arr`).
- Standardized uniform 64-bit (`i64`) function calling convention across LLVM codegen, fixing float truncation bugs in object methods, generic monomorphizations, string methods, thread wrappers, and async/await expressions.

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
