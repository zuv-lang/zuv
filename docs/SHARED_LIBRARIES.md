# Building & Exporting Shared Libraries (`.dll` / `.so` / `.dylib`)

Zuv supports compiling native shared dynamic libraries (**`.dll`** on Windows, **`.so`** on Linux, **`.dylib`** on macOS) with standard C ABI exports via the `pub extern "C"` syntax and the `--cdylib` / `--lib` compiler flag.

---

## 1. Compilation Architecture

```
┌────────────────────────────────────────────────────────┐
│  1. Zuv Source Code (e.g. math.zv)                     │
│     pub extern "C" add a: num, b: num -> num { ... }   │
└───────────────────────────┬────────────────────────────┘
                            │  1. Lexer & Parser
                            ▼
┌────────────────────────────────────────────────────────┐
│  2. AST Representation (FunctionDeclAST)               │
│     isExported: true, isExternC: true                  │
└───────────────────────────┬────────────────────────────┘
                            │  2. In-Process LLVM Codegen
                            ▼
┌────────────────────────────────────────────────────────┐
│  3. LLVM IR Code Generation                            │
│     define dllexport double @add(double %a, double %b) │
└───────────────────────────┬────────────────────────────┘
                            │  3. LLVM Target Machine (AOT)
                            ▼
┌────────────────────────────────────────────────────────┐
│  4. Native Object Code File (.obj / .o)                │
│     Contains compiled machine code + export table      │
└───────────────────────────┬────────────────────────────┘
                            │  4. LLD Linker (DLL / Shared Mode)
                            ▼
┌────────────────────────────────────────────────────────┐
│  5. Final Shared Dynamic Library Output                │
│     • Windows: math.dll (+ math.lib import lib)        │
│     • Linux:   libmath.so                              │
│     • macOS:   libmath.dylib                           │
└───────────────────────────┘
```

---

## 2. Writing Exportable Zuv Code

Use `pub extern "C"` to declare and define functions with unmangled C calling conventions:

```zuv
// math.zv

pub extern "C" add a: num, b: num -> num {
    -> a + b
}

pub extern "C" multiply a: num, b: num -> num {
    -> a * b
}

pub extern "C" calculateTax price: num, rate: num -> num {
    -> price * rate
}
```

- **`pub`**: Marks the function for public symbol export.
- **`extern "C"`**: Enforces the standard C ABI (no name mangling, standard parameter register passing).
- **`-> type`**: Explicit return type annotation.

---

## 3. Building the Shared Library

Use the `zuv build` command with `--cdylib` or `--lib`:

### Windows (`.dll`)
```powershell
zuv build math.zv --cdylib -o math.dll
```
*Generates `math.dll` (runtime shared library) and `math.lib` (import library for C/C++ build systems).*

### Linux (`.so`)
```bash
zuv build math.zv --cdylib -o libmath.so
```

### macOS (`.dylib`)
```bash
zuv build math.zv --cdylib -o libmath.dylib
```

---

## 4. Platform Linker Drivers & Flags

| Platform | Output Format | Linker | Flags Passed by Compiler |
| :--- | :--- | :--- | :--- |
| **Windows** | `math.dll` | `lld-link.exe` | `-dll -noentry -out:math.dll -implib:math.lib -defaultlib:libcmt -defaultlib:ucrt -defaultlib:vcruntime` |
| **Linux** | `libmath.so` | `ld.lld` | `-shared -soname libmath.so -o libmath.so -lc -lm` |
| **macOS** | `libmath.dylib` | `ld64.lld` | `-dylib -o libmath.dylib -lSystem` |

- **`-dll` / `-shared` / `-dylib`**: Configures the linker for dynamic shared library output instead of a console executable.
- **`-noentry`**: Relaxes the requirement for a `main()` entrypoint.
- **`-implib`**: Generates import stub libraries on Windows.

---

## 5. Interoperability & Consumption Examples

### A. JavaScript / TypeScript (Node.js & Bun)

Using **Bun FFI** or Node.js (`ffi-napi` / `koffi`):

```javascript
// bun_ffi.js
import { dlopen, FFIType } from "bun:ffi";

const lib = dlopen("./math.dll", {
    add: {
        args: [FFIType.f64, FFIType.f64],
        returns: FFIType.f64,
    },
    multiply: {
        args: [FFIType.f64, FFIType.f64],
        returns: FFIType.f64,
    },
    calculateTax: {
        args: [FFIType.f64, FFIType.f64],
        returns: FFIType.f64,
    }
});

console.log("10 + 25 =", lib.symbols.add(10, 25));              // 35
console.log("7 * 8 =", lib.symbols.multiply(7, 8));            // 56
console.log("Tax(100, 0.15) =", lib.symbols.calculateTax(100, 0.15)); // 15
```

### B. Python (`ctypes`)

```python
# test_math.py
import ctypes

lib = ctypes.CDLL("./math.dll")

lib.add.restype = ctypes.c_double
lib.add.argtypes = [ctypes.c_double, ctypes.c_double]

lib.multiply.restype = ctypes.c_double
lib.multiply.argtypes = [ctypes.c_double, ctypes.c_double]

lib.calculateTax.restype = ctypes.c_double
lib.calculateTax.argtypes = [ctypes.c_double, ctypes.c_double]

print("10 + 25 =", lib.add(10.0, 25.0))                # 35.0
print("7 * 8 =", lib.multiply(7.0, 8.0))               # 56.0
print("Tax(100, 0.15) =", lib.calculateTax(100.0, 0.15))  # 15.0
```

### C. C / C++

```c
// main.c
#include <stdio.h>

__declspec(dllimport) double add(double a, double b);
__declspec(dllimport) double multiply(double a, double b);
__declspec(dllimport) double calculateTax(double price, double rate);

int main() {
    printf("Add: %f\n", add(10.0, 25.0));
    printf("Multiply: %f\n", multiply(7.0, 8.0));
    printf("Tax: %f\n", calculateTax(100.0, 0.15));
    return 0;
}
```

### D. Zuv Inbound C FFI

Another Zuv program can link and consume the generated DLL directly:

```zuv
// call_math.zv
extern "math.dll" add a: num, b: num -> num
extern "math.dll" multiply a: num, b: num -> num
extern "math.dll" calculateTax price: num, rate: num -> num

sum = add 10, 25
prod = multiply 7, 8
tax = calculateTax 100, 0.15

prnt (`Sum: ${sum}, Prod: ${prod}, Tax: ${tax}`)
```
