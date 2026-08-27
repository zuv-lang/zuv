# Using Dynamic (`.dll`) and Static (`.lib`) Libraries in Zuv

This guide covers how to declare, import, link, and export both dynamic (`.dll`, `.so`, `.dylib`) and static (`.lib`, `.a`) native libraries in Zuv using the Foreign Function Interface (FFI).

---

## 1. Overview of FFI Syntax

Zuv supports both **Single-Line** (individual) and **Grouped / Block** syntax for:
1. **Importing** symbols from dynamic libraries (`.dll`) and static libraries (`.lib`).
2. **Exporting** symbols with C ABI conventions for compilation into dynamic shared libraries (`--cdylib`).

| Syntax Form | Import Syntax | Export Syntax (`--cdylib`) |
| :--- | :--- | :--- |
| **Single-Line** | `extern "<lib>" fnName arg1: type -> retType` | `pub extern "C" fnName arg1: type -> retType { ... }` |
| **Grouped / Block** | `extern "<lib>" { fn1 ...; fn2 ... }` | `pub extern "C" { fn1 ... { ... }; fn2 ... { ... } }` |

---

## 2. Using Dynamic Libraries (`.dll`)

### 2.1 Importing from a DLL

To consume a `.dll`, specify the DLL filename in the `extern` declaration:

#### Grouped / Block Declaration
```zuv
extern "test_math.dll" {
    add a: num, b: num -> num
    multiply a: num, b: num -> num
}
```

#### Single-Line Declaration
```zuv
extern "test_math.dll" calculateTax price: num, rate: num -> num
```

#### Complete Consumer Example
```zuv
// main.zv

// 1. Grouped import
extern "test_math.dll" {
    add a: num, b: num -> num
    multiply a: num, b: num -> num
}

// 2. Single-line import
extern "test_math.dll" calculateTax price: num, rate: num -> num

sum = add 10, 20
mul = multiply 5, 6
tax = calculateTax 100, 0.15

prnt (`Sum: ${sum}`)
prnt (`Product: ${mul}`)
prnt (`Tax: ${tax}`)
```

### 2.2 Exporting a DLL from Zuv (`--cdylib`)

You can create a `.dll` from Zuv using `pub extern "C"` and compiling with `--cdylib`:

```zuv
// math_export.zv

// Grouped block export
pub extern "C" {
    add a: num, b: num -> num {
        -> a + b
    }

    multiply a: num, b: num -> num {
        -> a * b
    }
}

// Single-line export
pub extern "C" calculateTax price: num, rate: num -> num {
    -> price * rate
}
```

#### Build Command
```powershell
zuv build math_export.zv --cdylib -o test_math.dll
```
*Emits `test_math.dll` and its companion import library `test_math.lib`.*

---

## 3. Using Static Libraries (`.lib`)

### 3.1 Importing from a Static Library

When linking against a static library (`.lib` on Windows or `.a` on Linux/macOS), specify the library name with the `.lib` extension:

#### Grouped / Block Declaration
```zuv
extern "test_static_math.lib" {
    static_add a: num, b: num -> num
    static_multiply a: num, b: num -> num
}
```

#### Single-Line Declaration
```zuv
extern "test_static_math.lib" static_calculateTax price: num, rate: num -> num
```

#### Complete Consumer Example
```zuv
// call_static.zv

// 1. Grouped import from static library
extern "test_static_math.lib" {
    static_add a: num, b: num -> num
    static_multiply a: num, b: num -> num
}

// 2. Single import from static library
extern "test_static_math.lib" static_calculateTax price: num, rate: num -> num

sum = static_add 10, 20
mul = static_multiply 5, 6
tax = static_calculateTax 100, 0.15

if sum == 30 {
    prnt "static_add ok"
}
if mul == 30 {
    prnt "static_multiply ok"
}
if tax == 15 {
    prnt "static_calculateTax ok"
}
```

### 3.2 Creating a Static `.lib` (C/C++ Source)

If you have native C code:

```c
// static_math.c
double static_add(double a, double b) {
    return a + b;
}

double static_multiply(double a, double b) {
    return a * b;
}

double static_calculateTax(double price, double rate) {
    return price * rate;
}
```

#### Build the Static Library
```powershell
# Compile C to object file
clang -c static_math.c -o static_math.obj

# Archive into .lib
llvm-lib /OUT:test_static_math.lib static_math.obj
```

#### Build and Run Zuv Program
```powershell
zuv build call_static.zv -o call_static.exe
.\call_static.exe
```
*The Zuv compiler automatically invokes the linker with `-defaultlib:test_static_math.lib` or direct archive inclusion.*

---

## 4. Type Mapping Reference

When interfacing with C libraries via `.dll` or `.lib`, parameter and return types map as follows:

| Zuv Type | C Type | LLVM IR Type | Register Passing Convention |
| :--- | :--- | :--- | :--- |
| **`num`** | `double` | `double` | Floating point register (`XMM0`-`XMM3`) |
| **`i32`** / **`int`** | `int` / `int32_t` | `i32` | Integer register (`RCX`, `RDX`, `R8`, `R9`) |
| **`i64`** | `int64_t` / `size_t` | `i64` | Integer register (`RCX`, `RDX`, `R8`, `R9`) |
| **`str`** | `const char*` | `ptr` | Pointer in integer register |
| **`ptr`** | `void*` / `uintptr_t` | `ptr` | Pointer in integer register |
| **`bool`** | `bool` / `int8_t` | `i1` | Integer register |
| **`void`** | `void` | `void` | None |

---

## 5. Summary of Build Commands

| Target Type | Command | Resulting Artifacts |
| :--- | :--- | :--- |
| **Build Executable consuming `.dll`** | `zuv build app.zv -o app.exe` | `app.exe` (requires `.dll` at runtime) |
| **Build Executable consuming `.lib`** | `zuv build app.zv -o app.exe` | `app.exe` (statically linked, standalone) |
| **Build Dynamic Library (`.dll`)** | `zuv build lib.zv --cdylib -o lib.dll` | `lib.dll` + `lib.lib` (import library) |
| **Self-Host Compiler Equivalent** | `zuv_selfhost build lib.zv --cdylib -o lib.dll` | Same native binary output |
