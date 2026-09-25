# Zuv Compiler Error Codes & Diagnostics Reference

This document provides a comprehensive reference of compiler error codes, diagnostic formatting conventions, and error series categories in the **Zuv Compiler System**, with a primary focus on error codes implemented and active in the **Self-Hosting Compiler** (`src/bootstrap/`).

---

## 1. Diagnostic Architecture & Output Format

The Zuv compiler uses a Rust/Clang-grade diagnostic formatting engine defined in [`diagnostics.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/diagnostics.zv) (and bootstrapped by `src/Diagnostics.cpp`).

Every error reported by the compiler includes:
1. **Error Code**: A standardized alphanumeric code (e.g., `E0101`, `E0204`, `E0306`).
2. **Descriptive Message**: Clear explanation of the semantic or syntax violation.
3. **Source Location**: Exact file path, 1-based line, and 1-based column (`--> file.zv:line:col`).
4. **Code Snippet & Caret Span**: The offending line of source code rendered with precise caret indicators (`^` or `^^^^^`) spanning the exact token or expression.
5. **Actionable Help / Note**: An optional suggestion line (`= help: ...` or `= note: ...`) guiding the developer on how to fix the problem.

### Standard Diagnostic Example

```text
error[E0204]: Cannot mutate 'count' while it is borrowed
  --> src/main.zv:5:5
   |
 5 |     count = count + 1
   |     ^^^^^
   = help: cannot assign to borrowed variable; release active reference first
```

---

## 2. Error Code Series Taxonomy (`E0001` – `E9999`)

Error codes in Zuv are partitioned into dedicated numerical series by compiler subsystem. This structure guarantees immediate identification of which compilation phase caught the error:

| Error Series | Compiler Subsystem / Phase | Scope & Description |
| :--- | :--- | :--- |
| **`E0001` – `E0099`** | **Lexer & Scanner** | Tokenization failures, illegal characters, unclosed string/template literals, malformed numeric/hex/binary literals, invalid escape sequences. |
| **`E0100` – `E0199`** | **Parser & Syntax** | Grammar violations, unexpected tokens, missing semicolons/delimiters (`{`, `}`, `(`, `)`, `[`, `]`), malformed expressions and declarations. |
| **`E0200` – `E0299`** | **Scope, Ownership & Borrow Checker** | Duplicate variable declarations, use of moved values, mutation of immutable bindings/constants, borrow conflicts, lifetime violations, unsafe operations. |
| **`E0300` – `E0399`** | **Semantic & Type Checker** | Type mismatches, operator operand incompatibilities, unknown struct fields/properties, function signature mismatches, argument count errors, module export-visibility violations. |
| **`E0400` – `E0499`** | **Metaprogramming & Macros** | Macro expansion failures, malformed attributes/decorators (`@deprecated`, `@inline`, `@test`), invalid compile-time evaluation. |
| **`E0500` – `E0599`** | **Code Generation (LLVM IR)** | Target lowering errors, invalid LLVM type conversions, unsupported calling conventions, unresolved extern symbols. |
| **`E0600` – `E0699`** | **Linker & Binary Emission** | In-process LLD linking failures, missing static/dynamic libraries, corrupted object files (`.obj`), target machine creation errors. |
| **`E0700` – `E0799`** | **CLI & Package Management** | Missing source files, invalid command-line flags, invalid target triples, corrupted manifest (`zuv.yml`) or lockfile (`zuv.sum`). |

---

## 3. Error Codes Available in Self-Host Compiler

The table below lists all error codes currently implemented, checked, and emitted by the self-hosting compiler (`src/bootstrap/`):

| Error Code | Series | Phase / File | Summary | Status |
| :--- | :--- | :--- | :--- | :--- |
| [`E0101`](#e0101-expected-valid-expression-or-declaration) | `E01xx` | [`parser/expr.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/parser/expr.zv)<br>[`parser/decl.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/parser/decl.zv) | Expected valid expression or top-level declaration statement | **Active** |
| [`E0102`](#e0102-expected-specific-token-or-delimiter) | `E01xx` | [`parser/parser.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/parser/parser.zv)<br>[`parser/decl.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/parser/decl.zv)<br>[`parser/expr.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/parser/expr.zv) | Expected specific token, delimiter, or identifier | **Active** |
| [`E0201`](#e0201-duplicate-declaration-in-scope) | `E02xx` | [`checker/borrow.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/borrow.zv) | Variable or symbol already declared in the current scope | **Active** |
| [`E0202`](#e0202-undefined-identifier-or-use-of-moved-value) | `E02xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv)<br>[`checker/borrow.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/borrow.zv) | Undefined identifier, or use / borrow of a moved value | **Active** |
| [`E0203`](#e0203-cannot-mutate-immutable-variable-or-constant) | `E02xx` | [`checker/borrow.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/borrow.zv)<br>`src/BorrowChecker.cpp` | Cannot mutate immutable variable, constant, or immutable borrow | **Active** |
| [`E0204`](#e0204-borrow-conflict-and-lifetime-violation) | `E02xx` | [`checker/borrow.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/borrow.zv) | Cannot mutate/move while borrowed, or value does not live long enough | **Active** |
| [`E0205`](#e0205-raw-pointer-operation-outside-unsafe-block) | `E02xx` | `src/BorrowChecker.cpp`<br>[`checker/borrow.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/borrow.zv) | Raw pointer dereference or memory intrinsic requires `unsafe` block | **Active** |
| [`E0301`](#e0301-module-export-visibility-violation) | `E03xx` | [`checker/checker.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/checker.zv)<br>`src/BorrowChecker.cpp` | Cannot import unexported (lowercase) symbol from external module | **Active** |
| [`E0302`](#e0302-property-does-not-exist-on-type) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Property / field does not exist on object/struct type | **Active** |
| [`E0303`](#e0303-unary-operator-type-incompatibility) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Unary operator requires numeric operand | **Active** |
| [`E0304`](#e0304-arithmetic-operator-type-incompatibility) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Arithmetic operator requires numeric operands | **Active** |
| [`E0305`](#e0305-comparison-operator-type-incompatibility) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Comparison operator requires comparable types (numbers or strings) | **Active** |
| [`E0306`](#e0306-type-assignment--parameter-mismatch) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Type mismatch in assignment, variable initialization, or call argument | **Active** |
| [`E0307`](#e0307-invalid-array-index-expression-type) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Index expression must be numeric | **Active** |
| [`E0308`](#e0308-function-call-argument-count-mismatch) | `E03xx` | [`checker/infer.zv`](file:///D:/rujs/sub_projects/zuv/src/bootstrap/checker/infer.zv) | Function call argument count does not match declared parameter count | **Active** |

---

## 4. Detailed Error Code Reference

### `E0101`: Expected Valid Expression or Declaration

- **Compiler Phase**: Parser (`src/bootstrap/parser/expr.zv`, `src/bootstrap/parser/decl.zv`)
- **Trigger**: Occurs when the parser encounters a token that cannot start an expression or statement, or encounters an unexpected end-of-file (EOF).

#### Incorrect Code
```zuv
badVar = 
```

#### Compiler Output
```text
error[E0101]: Expected valid expression, got EOF
  --> tests/semantic_errors.test.zv:3:1
   |
 3 | 
   | ^ expected expression
   |
   = help: provide a valid identifier, literal, or expression
```

#### Solution
Supply a valid expression, literal, or identifier following the assignment operator.

---

### `E0102`: Expected Specific Token or Delimiter

- **Compiler Phase**: Parser (`src/bootstrap/parser/parser.zv`, `decl.zv`, `expr.zv`)
- **Trigger**: A mandatory syntactic delimiter (such as `{`, `}`, `(`, `)`, `[`, `]`, `=>`, or type identifier) was expected according to Zuv grammar rules, but a different token was encountered.

#### Incorrect Code
```zuv
fn square x
    -> x * x
}
```

#### Compiler Output
```text
error[E0102]: Expected '{' to begin block, got '->'
  --> src/main.zv:2:5
   |
 2 |     -> x * x
   |     ^^ expected token
   |
   = help: add '{' here
```

#### Solution
Ensure all blocks, parameter lists, array literals, and object bodies are properly delimited with matching braces and parentheses.

---

### `E0201`: Duplicate Declaration in Scope

- **Compiler Phase**: Borrow & Safety Checker (`src/bootstrap/checker/borrow.zv`)
- **Trigger**: Declaring a variable or symbol with `let` or `mut` using an identifier that already exists in the current block scope.

#### Incorrect Code
```zuv
let item = "initial declaration"
let item = "duplicate declaration in the same scope"
```

#### Compiler Output
```text
error[E0201]: Variable 'item' is already declared in this scope
  --> tests/redecl_error.test.zv:5:5
   |
 5 | let item = "duplicate declaration in the same scope"
   |     ^^^^ duplicate declaration
   |
   = help: remove the duplicate 'let' / 'mut' or rename the variable
```

#### Solution
Reassign the existing variable without `let`, rename the variable, or wrap the second declaration in a new nested scope block `{ ... }`.

---

### `E0202`: Undefined Identifier or Use of Moved Value

- **Compiler Phase**: Type Checker & Borrow Checker (`src/bootstrap/checker/infer.zv`, `borrow.zv`)
- **Trigger**: Referencing a variable or symbol that has not been defined in scope, or attempting to read, use, or borrow a variable whose ownership has already been transferred (moved).

#### Incorrect Code (Undefined Identifier)
```zuv
prnt unknownVariable
```

#### Compiler Output
```text
error[E0202]: Undefined identifier 'unknownVariable'
  --> src/main.zv:1:6
   |
 1 | prnt unknownVariable
   |      ^^^^^^^^^^^^^^^
   = help: check variable spelling or define variable
```

#### Incorrect Code (Use of Moved Value)
```zuv
let a = [1, 2, 3]
let b = a           // Ownership of array is moved from 'a' to 'b'
prnt a              // Error: 'a' was moved
```

#### Compiler Output
```text
error[E0202]: Use of moved value: 'a'
  --> src/main.zv:3:6
   |
 3 | prnt a
   |      ^
   = help: value was moved earlier
```

#### Solution
- Declare identifiers before using them.
- If transferring ownership, do not use the original variable; or borrow the value with `&let` instead of moving it.

---

### `E0203`: Cannot Mutate Immutable Variable or Constant

- **Compiler Phase**: Borrow Checker (`src/bootstrap/checker/borrow.zv`, `src/BorrowChecker.cpp`)
- **Trigger**: Attempting to mutate or reassign an immutable variable declared with `let`, an all-uppercase constant (e.g. `MAX_COUNT`), or an immutable reference.

#### Incorrect Code
```zuv
MAX_BUFFER = 2048
MAX_BUFFER = 4096   // Error: UPPERCASE identifiers are strictly immutable constants
```

#### Compiler Output
```text
error[E0203]: Cannot mutate immutable variable: 'MAX_BUFFER'
  --> src/main.zv:2:1
   |
 2 | MAX_BUFFER = 4096
   | ^^^^^^^^^^
   = help: declare variable with 'mut' if reassignment is intended
```

#### Solution
Use lowercase variable names declared with `mut` for mutable bindings. Constants with all-uppercase names cannot be reassigned.

---

### `E0204`: Borrow Conflict and Lifetime Violation

- **Compiler Phase**: Borrow Checker (`src/bootstrap/checker/borrow.zv`)
- **Trigger**: 
  1. Mutating or moving a variable while active borrow references exist.
  2. Borrowing a local value into a reference whose lifetime outlasts the value (scope escape).

#### Incorrect Code (Mutation While Borrowed)
```zuv
mut count = 10
&let r = count      // Active borrow created
count = 20          // Error: mutating 'count' while borrowed by 'r'
prnt r
```

#### Compiler Output
```text
error[E0204]: Cannot mutate 'count' while it is borrowed
  --> src/main.zv:3:1
   |
 3 | count = 20
   | ^^^^^
   = help: cannot assign to borrowed variable
```

#### Solution
End the borrow scope before mutating the original variable, or avoid mutating values while they have live reference aliases.

---

### `E0205`: Raw Pointer Operation Outside `unsafe` Block

- **Compiler Phase**: Borrow Checker (`src/BorrowChecker.cpp`, `checker/borrow.zv`)
- **Trigger**: Dereferencing a raw pointer (`*ptr`) or invoking low-level memory allocation functions without wrapping in an `unsafe { ... }` block.

#### Incorrect Code
```zuv
let p: ptr = alloc(64)
let val = *p        // Error: raw pointer dereference
```

#### Compiler Output
```text
error[E0205]: Dereference of raw pointer requires unsafe block
  --> src/main.zv:2:11
   |
 2 | let val = *p
   |           ^^
   = help: wrap pointer dereference in an 'unsafe { ... }' block
```

#### Solution
Wrap all raw pointer dereferences and unchecked memory calls in an `unsafe { ... }` block.

---

### `E0301`: Module Export-Visibility Violation

- **Compiler Phase**: Type Checker & Module Loader (`src/bootstrap/checker/checker.zv`, `src/BorrowChecker.cpp`)
- **Trigger**: Attempting to import an unexported (lowercase) symbol across module boundaries. In Zuv, only symbols starting with an **uppercase letter** or marked with `pub` are exported from a module.

#### Incorrect Code
```zuv
// tests/export_error.test.zv
imp internalDiff frm tests/my_helper

prnt (internalDiff 100, 50)
```

#### Compiler Output
```text
error[E0301]: Cannot import unexported symbol 'internalDiff' from module 'tests/my_helper'
  --> tests/export_error.test.zv:2:5
   |
 2 | imp internalDiff frm tests/my_helper
   |     ^^^^^^^^^^^^
   = help: export symbol with uppercase first letter or 'pub' keyword in module
```

#### Solution
In the exporting module, capitalize the symbol name (e.g. `InternalDiff`) or export it as `pub`.

---

### `E0302`: Property Does Not Exist on Type

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**: Accessing a struct field or object property that is not defined on the inferred type schema.

#### Incorrect Code
```zuv
obj Point { x: num, y: num }
pt = Point { x: 10, y: 20 }
prnt pt.z           // Error: 'z' is not a member of Point
```

#### Compiler Output
```text
error[E0302]: Property 'z' does not exist on type 'Point'
  --> src/main.zv:3:9
   |
 3 | prnt pt.z
   |         ^
   = help: check member spelling or add field to Point
```

#### Solution
Check the field name spelling, or add the field to the object/type declaration.

---

### `E0303`: Unary Operator Type Incompatibility

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**: Applying a unary operator (`-`, `+`, `~`) to an operand that is not numeric.

#### Incorrect Code
```zuv
let s = "hello"
let neg = -s        // Error: cannot negate a string
```

#### Compiler Output
```text
error[E0303]: Unary operator '-' requires numeric operand, got str
  --> src/main.zv:2:11
   |
 2 | let neg = -s
   |           ^
   = help: provide a numeric expression
```

#### Solution
Ensure the operand evaluates to a numeric type (`num`, `i32`, `i64`).

---

### `E0304`: Arithmetic Operator Type Incompatibility

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**: Using arithmetic operators (`-`, `*`, `/`, `%`) with non-numeric operand types (or string concat with invalid types).

#### Incorrect Code
```zuv
let result = "hello" * 5    // Error: cannot multiply string
```

#### Compiler Output
```text
error[E0304]: Arithmetic operator '*' requires numeric operands, got str and num
  --> src/main.zv:1:22
   |
 1 | let result = "hello" * 5
   |                      ^
   = help: convert operands to numbers
```

#### Solution
Ensure both operands are numeric, or use string repetition helper functions instead of `*`.

---

### `E0305`: Comparison Operator Type Incompatibility

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**: Comparing two expressions of incompatible types using relational operators (`<`, `<=`, `>`, `>=`).

#### Incorrect Code
```zuv
let obj1 = { id: 1 }
if obj1 <= 10 {     // Error: cannot compare object with number
    prnt "matched"
}
```

#### Compiler Output
```text
error[E0305]: Comparison operator '<=' requires comparable types, got obj and num
  --> src/main.zv:2:9
   |
 2 | if obj1 <= 10 {
   |         ^^
   = help: ensure operands are both numeric or strings
```

#### Solution
Compare numeric or string fields directly (e.g., `obj1.id <= 10`).

---

### `E0306`: Type Assignment & Parameter Mismatch

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**:
  1. Assigning a value to a variable of an incompatible type.
  2. Passing an argument to a function whose type does not match the parameter type annotation.

#### Incorrect Code
```zuv
fn greet name: str :: void {
    prnt "Hello, " + name
}

greet 12345         // Error: expected str, got num
```

#### Compiler Output
```text
error[E0306]: Argument 1 of 'greet' expects str, got num
  --> src/main.zv:5:7
   |
 5 | greet 12345
   |       ^^^^^
   = help: pass a compatible argument type
```

#### Solution
Pass an argument matching the expected type annotation, or convert it explicitly (e.g. `"" + 12345`).

---

### `E0307`: Invalid Array Index Expression Type

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**: Indexing an array using an expression that does not evaluate to a numeric type.

#### Incorrect Code
```zuv
let items = ["apple", "banana", "cherry"]
let item = items["first"]   // Error: index must be numeric
```

#### Compiler Output
```text
error[E0307]: Index expression must be numeric, got str
  --> src/main.zv:2:18
   |
 2 | let item = items["first"]
   |                  ^^^^^^^
   = help: use an integer index
```

#### Solution
Use an integer numeric index (`0`, `1`, `2`, ...) when accessing array elements.

---

### `E0308`: Function Call Argument Count Mismatch

- **Compiler Phase**: Semantic Checker (`src/bootstrap/checker/infer.zv`)
- **Trigger**: Calling a function with fewer or more arguments than declared in its parameter signature.

#### Incorrect Code
```zuv
fn add a: num, b: num :: num {
    -> a + b
}

let sum = add 42    // Error: missing second argument
```

#### Compiler Output
```text
error[E0308]: Function 'add' expects 2 arguments, but got 1
  --> src/main.zv:5:11
   |
 5 | let sum = add 42
   |           ^^^
   = help: provide the expected number of arguments
```

#### Solution
Provide the exact number of required arguments according to the function's parameter signature.

---

## 5. Testing & Verifying Diagnostics

### 1. Check a Source File for Errors
You can verify compiler diagnostics without running the full code generation pipeline using the `check` subcommand:
```powershell
zuv check path/to/file.zv
# Or using the bootstrap binary:
.\zuv_bootstrap.exe check path/to/file.zv
```

### 2. Run the Verification Suite
The self-hosting compiler includes an automated 19-stage pipeline verification suite testing diagnostics, parser recovery, and borrow checker errors:
```powershell
zuv verify
# Or:
.\zuv_bootstrap.exe verify
```

### 3. Dedicated Negative Test Suites
The test suite includes dedicated negative test cases that validate accurate diagnostic emission:
- `tests/export_error.test.zv` — Tests `E0301` (unexported lowercase module symbol import rejection).
- `tests/redecl_error.test.zv` — Tests `E0201` (duplicate variable declaration detection).
- `tests/semantic_errors.test.zv` — Tests `E0101` (invalid expression / unexpected EOF syntax detection).
