# 🏷️ Attributes & Metaprogramming Annotations Guide

This guide documents the **Attribute & Annotation System** (also referred to as **Decorators**) in the Zuv programming language. Attributes provide compile-time directives, LLVM optimization hints, test discovery metadata, and code-generation directives attached directly to declarations.

---

## 1. Syntax Forms

Zuv supports two equivalent, clean syntactic styles for attaching metadata:

### A. Prefix `@` Syntax (Standard / Python & TypeScript Style)
```zuv
@inline
fastAdd a, b {
    -> a + b
}

@deprecated("Use calculateNew instead")
calculateOld x {
    -> x * 2
}
```

### B. Bracket `[...]` Syntax (Rust & C# Style)
```zuv
[inline(always)]
fastMultiply a, b {
    -> a * b
}

[noinline]
coldErrorHandler errCode {
    prnt "Error encountered: " + toStr errCode
}
```

---

## 2. Inbuilt Annotations Catalog

| Annotation | Syntaxes | Target | Description / Effect |
| :--- | :--- | :--- | :--- |
| **`inline`** | `@inline`, `[inline]`, `[inline(always)]` | Function | Directs LLVM to inline the function body into calling sites for zero call overhead. |
| **`noinline`** | `@noinline`, `[noinline]`, `[inline(never)]` | Function | Directs LLVM to never inline the function, keeping instruction caches compact for heavy routines. |
| **`test`** | `@test`, `[test]` | Function | Registers the function as an automated unit test discoverable by `zuv test`. |
| **`export`** | `@export`, `[export]` | Function | Emits `dllexport` on Windows (`.dll`) or default symbol visibility on Linux/macOS for dynamic linking. |
| **`deprecated`** | `@deprecated("msg")` | Function, Struct, Var | Emits a compiler deprecation warning when the target symbol is referenced. |
| **`derive`** | `@derive(Trait, ...)` | Struct (`obj`) | Instructs the metaprogramming code generator to automatically synthesize methods (e.g. `Clone`, `Debug`, `Hash`). |
| **`cold`** | `@cold`, `[cold]` | Function | Instructs branch predictors that this function is rarely executed (ideal for fatal panic/error paths). |

---

## 3. Detailed Annotation Usage & Examples

### ⚡ `@inline` / `[inline(always)]`
Inlining replaces a function call directly with its body, removing stack frame setup, argument passing, and `ret` instructions:

```zuv
@inline
distanceSquared x1, y1, x2, y2 {
    dx = x2 - x1
    dy = y2 - y1
    -> dx * dx + dy * dy
}

[inline(always)]
clamp val, minVal, maxVal {
    if val < minVal { -> minVal }
    if val > maxVal { -> maxVal }
    -> val
}
```

---

### 🛑 `@noinline` / `[noinline]`
Prevents the compiler from duplicating large function bodies into multiple callers, reducing binary footprint and instruction cache misses:

```zuv
@noinline
parseHeavyAstNode inputStr {
    // Large parsing state machine...
    -> 1
}

[inline(never)]
logStackTraceToDisk logPath {
    // Rarely called diagnostic handler
}
```

---

### 🧪 `@test`
Marks functions for automated unit testing:

```zuv
@inline
square x {
    -> x * x
}

@test
testSquareCalculation {
    res = square 5
    if res != 25 {
        prnt "FAILED: expected 25, got " + toStr res
        exit 1
    }
    prnt "PASS: square(5) == 25"
}

// Running the test
testSquareCalculation
```

---

### 🔌 `@export`
Exports a function symbol for native C ABI consumption by external runtimes (Node.js N-API / FFI, Python `ctypes`, C/C++ host applications):

```zuv
@export
pub extern "C" addNumbers a: num, b: num -> num {
    -> a + b
}

@export
pub extern "C" multiplyNumbers a: num, b: num -> num {
    -> a * b
}
```
* **Build Shared Library**:
  ```powershell
  zuv build my_lib.zv --cdylib -o my_lib.dll
  ```

---

### ⚠️ `@deprecated`
Communicates API migration paths to developers:

```zuv
@deprecated("Use Vector3::new instead")
obj LegacyVec3 {
    x: num,
    y: num,
    z: num
}

@deprecated("Use secureHash instead")
legacyMd5 text {
    // ...
}
```

---

### 🧬 `@derive` (Metaprogramming & Struct Traits)
Enables compile-time synthesis of common boilerplate methods for structs:

```zuv
@derive(Clone, Debug, Hash)
obj Player {
    id: num,
    username: str,
    score: num
}
```

---

## 4. Attaching Multiple Annotations

Multiple attributes can be stacked on a single declaration:

```zuv
@export
@inline
pub extern "C" fastVectorDot x1: num, y1: num, x2: num, y2: num -> num {
    -> x1 * x2 + y1 * y2
}
```

---

## 5. Verification Commands

* **Compile and run test file**:
  ```powershell
  zuv build tests/attributes.test.zv -o test_attr.exe
  .\test_attr.exe
  ```
* **Inspect emitted LLVM function attributes**:
  ```powershell
  zuv build tests/attributes.test.zv --emit-llvm -o test_attr.ll
  ```