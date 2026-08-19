# Changelog

All notable changes to the **Zuv** programming language and self-hosting compiler are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
