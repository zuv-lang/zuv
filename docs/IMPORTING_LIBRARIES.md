# Importing & Linking Libraries (`.lib` and `.dll`) in Zuv

Zuv provides first-class support for interoperating with native C and C++ libraries. You can consume native code via:
1. **Link-Time Static Import (`.lib`)**: Directly link static libraries or import libraries into your binary.
2. **Link-Time Dynamic Import (`.dll`)**: Link against a shared library using an import stub, binding calls at compile-time.
3. **Runtime Dynamic FFI (`ffi.*`)**: Load dynamic libraries at runtime without any compile-time `.lib` dependency.

---

## 1. Quick Comparison

| Approach | Syntax | Required Files at Build Time | Required Files at Runtime | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **Static `.lib`** | `extern "foo.lib"` | `foo.lib` | None (fully embedded) | Standalone binaries, custom C++ static libs |
| **Dynamic `.dll` (Link-Time)** | `extern "foo.dll"` | `foo.lib` (import stub) | `foo.dll` | System APIs, Zuv cdylibs, large shared libs |
| **Dynamic FFI (Runtime)** | `ffi.ld("foo.dll")` | None | `foo.dll` | Plugins, optional dependencies, no `.lib` available |

---

## 2. Link-Time Static Import (`.lib`)

Use `extern "<name>.lib"` to link pre-compiled C/C++ static libraries or system libraries directly.

### A. Single Function Declaration
```zuv
// Import individual C function from a .lib
extern "msvcrt.lib" puts s: str :: num
extern "user32.lib" GetSystemMetrics nIndex: num :: num

puts "Hello from C runtime!"
width = GetSystemMetrics 0
prnt ("Screen width: " + width)
```

### B. Grouped Block Declaration
Use block syntax to declare multiple symbols from the same library concisely:
```zuv
extern "kernel32.lib" {
    GetCurrentProcessId :: num
    Sleep ms: num :: void
}

pid = GetCurrentProcessId
Sleep 100
prnt ("Process ID: " + pid)
```

### C. Using Custom C/C++ Static Libraries
You can write and compile any C or C++ file into a `.lib` and call it directly from Zuv:

1. **Write C/C++ code (`my_math.c`)**:
   ```c
   // my_math.c
   double add(double a, double b) {
       return a + b;
   }
   double multiply(double a, double b) {
       return a * b;
   }
   ```

2. **Compile to `.lib` using MSVC or Clang**:
   ```powershell
   clang -c my_math.c -o my_math.obj
   llvm-lib my_math.obj /out:my_math.lib
   # Or using MSVC cl and lib:
   # cl /c my_math.c && lib my_math.obj /out:my_math.lib
   ```

3. **Consume in Zuv (`main.zv`)**:
   ```zuv
   // main.zv
   extern "my_math.lib" {
       add a: num, b: num :: num
       multiply a: num, b: num :: num
   }

   sum = add 12.5, 7.5
   prod = multiply 4, 5

   prnt ("Sum: " + sum)       // 20
   prnt ("Product: " + prod)   // 20
   ```

4. **Build with Zuv**:
   ```powershell
   zuv build main.zv -o app.exe
   .\app.exe
   ```

---

## 3. Link-Time Dynamic Import (`.dll`)

When referencing a `.dll`, the compiler emits an external symbol declaration and instructs `lld-link` to link against the corresponding import library (`<name>.lib`).

### A. Windows System DLLs
```zuv
extern "msvcrt.dll" puts s: str :: num
extern "kernel32.dll" GetCurrentProcessId :: num
extern "user32.dll" GetSystemMetrics nIndex: num :: num

puts "Invoking msvcrt puts"
pid = GetCurrentProcessId
```

### B. Custom Zuv or C Shared Libraries
When consuming a DLL built by Zuv (via `--cdylib`) or compiled by MSVC/Clang:

```zuv
// app.zv
extern "test_math.dll" {
    add a: num, b: num :: num
    multiply a: num, b: num :: num
}
extern "test_math.dll" calculateTax price: num, rate: num :: num

sum = add 10, 25
prod = multiply 7, 8
tax = calculateTax 100, 0.15

prnt ("Sum: " + sum)     // 35
prnt ("Prod: " + prod)   // 56
prnt ("Tax: " + tax)     // 15
```

### Build & Execution:
```powershell
# 1. Build DLL and import library
zuv build tests/cdylib_export.test.zv -o test_math.dll --cdylib
# Output: test_math.dll + test_math.lib

# 2. Build and run consumer
zuv tests/call_cdylib.test.zv
```

> [!NOTE]
> **Windows DLL Runtime Resolution**:
> At runtime, Windows searches for the `.dll` in:
> 1. The directory containing the executable (`app.exe`).
> 2. The current working directory.
> 3. Directories listed in the `PATH` environment variable.
> Keep your `.dll` alongside your `.exe` when deploying.

---

## 4. Runtime Dynamic FFI (`ffi.*`)

If you do **not** have a `.lib` import library at compile time, or if you want to load libraries conditionally (e.g. plugins), use Zuv's dynamic FFI engine.

### Functions:
- `ffi.ld(path)` / `ffi.load(path)`: Loads dynamic library (`LoadLibraryA` / `dlopen`). Returns library handle `ptr`.
- `ffi.sym(handle, name)`: Resolves function symbol address (`GetProcAddress` / `dlsym`). Returns function pointer `ptr`.
- `ffi.call(fnPtr, ...args)`: Invokes function pointer with arguments using native C calling conventions.
- `ffi.cls(handle)` / `ffi.close(handle)`: Unloads dynamic library (`FreeLibrary` / `dlclose`).

### Complete Example:
```zuv
// Dynamically load kernel32.dll at runtime
hKernel = ffi.ld "kernel32.dll"
if hKernel != 0 {
    // Resolve GetCurrentProcessId symbol
    fnGetPid = ffi.sym hKernel, "GetCurrentProcessId"
    if fnGetPid != 0 {
        pid = ffi.call fnGetPid
        prnt ("PID from dynamic FFI: " + pid)
    }

    // Unload when finished
    ffi.cls hKernel
}
```

---

## 5. Type Mapping Reference

When declaring parameters and return types in `extern` definitions:

| Zuv Type | LLVM IR Type | C / Win32 Equivalent | Notes |
| :--- | :--- | :--- | :--- |
| `num` (custom DLL / math) | `double` | `double` (64-bit float) | Used by custom math functions and cdylibs |
| `num` (Win32 OS APIs) | `i32` | `int`, `DWORD`, `UINT` | Used by `kernel32`, `user32`, `msvcrt` |
| `str` | `ptr` | `const char*` | Null-terminated UTF-8 string pointer |
| `ptr` | `ptr` | `void*`, `HANDLE`, `HWND` | Opaque memory pointer or OS handle |
| `bool` | `i1` | `bool` / `BOOL` | Boolean value |
| `i64` / `u64` | `i64` | `int64_t`, `size_t` | 64-bit integer |
| `i32` / `u32` | `i32` | `int32_t`, `int`, `DWORD` | 32-bit integer |
| `i16` / `u16` | `i16` | `int16_t`, `short` | 16-bit integer |
| `i8` / `u8` / `byte` | `i8` | `int8_t`, `char`, `unsigned char` | 8-bit byte |
| `void` | `void` | `void` | No return value |

---

## 6. Library Search Paths & Resolution

The Zuv linker (`lld-link`) automatically resolves library paths in the following priority:
1. Current working directory (`.`)
2. Parent project directory (`..`)
3. `tests/` directory
4. MSVC and Windows SDK system library directories (configured via `LIB` environment variable).

### Standard Library Aliases
The C runtime is automatically resolved without manual path specification:
- `"c"`, `"C"`, `"libc"` -> Platform C Runtime (`libcmt.lib` on Windows, `-lc` on Linux, `-lSystem` on macOS).
- `"msvcrt"`, `"msvcrt.lib"`, `"msvcrt.dll"` -> Microsoft Visual C Runtime.
- `"kernel32"`, `"user32"`, `"ws2_32"` -> Standard Windows API libraries (automatically linked).

---

## 7. Troubleshooting

### 1. `lld-link: error: could not open 'foo.lib': no such file or directory`
- **Cause**: The compiler is trying to link against `foo.lib`, but it was not found in the search paths.
- **Solution**:
  - If `foo.lib` is a custom library, ensure `foo.lib` is located in the working directory, `tests/`, or the parent folder.
  - If compiling a DLL export test, build the DLL first:
    ```powershell
    zuv build foo.zv -o foo.dll --cdylib
    ```

### 2. `error: undefined symbol: myFunc`
- **Cause**: The function name declared in `extern` does not match the exported C symbol name (or C++ name mangling occurred).
- **Solution**: In C++ libraries, ensure exported functions are declared with `extern "C"` to disable C++ name mangling:
  ```cpp
  extern "C" __declspec(dllexport) double myFunc(double a);
  ```

### 3. Application terminates immediately when running `.exe`
- **Cause**: Windows cannot find a required `.dll` at runtime.
- **Solution**: Copy the required `.dll` file into the same directory as the generated `.exe`.
