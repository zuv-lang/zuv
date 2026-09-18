# Zuv Language Specification & Developer Reference (v0.5 / v1.0)

Welcome to the official language manual for **Zuv**, a fast, statically-typed, memory-safe compiled language featuring parenthesis-free syntax, modern borrowing & ownership semantics, C FFI import/export, native async/await, and Win32/POSIX multithreading.

---

## Table of Contents
1. [Syntax & Basics](#syntax--basics)
2. [Variables, Constants & Types](#variables-constants--types)
3. [Operators & Modern Expressions](#operators--modern-expressions)
4. [Ownership, Borrowing & Unsafe Pointers](#ownership-borrowing--unsafe-pointers)
5. [Control Flow & Pattern Matching](#control-flow--pattern-matching)
6. [Destructuring](#destructuring)
7. [Functions, Generics & Closures](#functions-generics--closures)
8. [Structs, Objects & Enums](#structs-objects--enums)
9. [Error Handling (`ok`/`err`, `try`/`cth`/`fin`)](#error-handling)
10. [C Foreign Function Interface (Inbound FFI)](#c-foreign-function-interface-inbound-ffi)
11. [Exporting C Shared Libraries (`pub extern "C"` / `--cdylib`)](#exporting-c-shared-libraries-pub-extern-c---cdylib)
12. [Module Imports (`imp`)](#module-imports-imp)
13. [Async / Await & Multithreading](#async--await--multithreading)
14. [Standard Library (`std/*`)](#standard-library-std)

---

## Syntax & Basics

Zuv uses a minimalist, modern, parenthesis-free calling syntax for functions and logging:

```zuv
main {
    prnt "Hello from Zuv!"
    lg "Logging a standard message"
    wrn "Warning message"
    inf "Informational message"
    err "Error message"
}
```

### Logging & Printing Statements
- `prnt <expr>`: Prints output formatted with a newline.
- `lg <expr>` / `inf <expr>` / `wrn <expr>` / `err <expr>`: Color-coded logging keywords for standard output streams.

---

## Variables, Constants & Types

### Variable Declaration & Mutability
Variables in Zuv are declared via direct assignment and are **mutable by default** without needing any keyword ceremony:

```zuv
counter = 20        // Mutable variable (default)
counter = counter + 1

name = "Zuv"
name = "Zuv Compiler" // Reassignment allowed
```

> [!NOTE]
> The `let` and `mut` keywords are deprecated. Standard assignment (`x = 1`) is mutable by default. `let` is retained only for backward compatibility.

### Block-Letter Constants (`UPPER_CASE`)
Constants in Zuv follow a zero-keyword naming convention based on **uppercase block lettering**:

- **Rule**: Any identifier consisting entirely of capital letters, numbers, and underscores (e.g. `ABC`, `MAX_RETRIES`, `PORT`, `API_URL`) is strictly **immutable** and enforced at compile time by the borrow checker (`error[E0203]`):
```zuv
MAX_BUFFER = 1024
PORT = 8080

MAX_BUFFER = 2048   // Compile Error [E0203]: Cannot mutate immutable constant 'MAX_BUFFER'
```

- **Scope**: Constants can be defined at module/global scope as well as within local function scopes:
```zuv
calculateTax price: num :: num {
    DEFAULT_TAX_RATE = 0.15
    -> price * DEFAULT_TAX_RATE
}
```

- **Objects & Arrays as Constants**: When an uppercase constant is bound to an object or array, the root reference itself is immutable (cannot be reassigned), while internal elements and fields remain accessible and mutable:
```zuv
CONFIG = { timeout: 3000 }
CONFIG.timeout = 5000     // Allowed (mutating field)
CONFIG = { timeout: 1000 } // Error [E0203]: Cannot reassign constant 'CONFIG'

LIST = [1, 2]
LIST.psh 3               // Allowed (appending to heap array)
LIST = [4, 5]            // Error [E0203]: Cannot reassign constant 'LIST'
```

- **Explicit Mutability Opt-Out**: In rare cases where a block-letter identifier needs to be mutable, prefixing it with `let` explicitly opts out of immutability:
```zuv
let MUTABLE_BLOCK = 10
MUTABLE_BLOCK = 20        // Allowed because 'let' explicitly opted out
```

### Primitive Types & Literal Formats
- **Floating Point Numbers (`num`)**: Standard IEEE 754 64-bit double precision (`10`, `3.14159`, `1.5e3`, `2.4E-2`).
- **BigInt Literals (`bigint`)**: 64-bit signed integer literals with `n` suffix (`100n`, `1234567890123456789n`).
- **Base Prefixes**:
  - Hexadecimal: `0xFF`, `0x1A3F`
  - Binary: `0b1010`, `0b11110000`
  - Octal: `0o755`, `0o644`
- **Special Values**: `nan` (NaN), `Infinity` (+Infinity), `nil` (null), `und` (undefined).
- **Booleans (`bool`)**: `true` / `yes`, `false` / `no`.
- **Strings (`str`)**: `"Hello World"` or string interpolation with template strings (e.g. `` `Count: ${counter}` ``).
- **Arrays**: `[1, 2, 3, 4]`.

---

## Operators & Modern Expressions

### Arithmetic & Power
- Standard arithmetic: `+`, `-`, `*`, `/`, `%`
- Exponentiation: `**` (e.g., `2 ** 8` yields `256`)
- Compound assignments: `+=`, `-=`, `*=`, `/=`, `%=`, `**=`, `&=`, `|=`, `^=`
- Increment & Decrement: `++`, `--`

### Logical & Boolean Operators (`!`, `not`, `&&`, `||`)
Zuv supports both symbol-based and natural word-based logical operators interchangeably:

- **Logical NOT (`!` / `not`)**:
  - Inverts truthiness of any boolean or expression:
  ```zuv
  isReady = false
  if !isReady {
      prnt "Not ready yet"
  }

  // Word-based 'not' is completely interchangeable with '!'
  if not isReady {
      prnt "Still not ready"
  }
  ```
  - Precedence: Prefix unary `!` binds tightly to the following operand or parenthesized expression:
  ```zuv
  if !(x > 10) {
      prnt "x is 10 or less"
  }
  ```

- **Logical AND (`&&` / `and`)**:
  ```zuv
  if isValid && hasPermission { ... }
  if isValid and hasPermission { ... }
  ```

- **Logical OR (`||` / `or`)**:
  ```zuv
  if isCached || fetchFromNetwork() { ... }
  if isCached or fetchFromNetwork() { ... }
  ```

- **Inequality (`!=`)**:
  ```zuv
  if count != 0 { ... }
  ```

### Bitwise Operators
- Bitwise AND: `&`
- Bitwise OR: `|`
- Bitwise XOR: `^`
- Bitwise NOT: `~`
- Left Shift: `<<`
- Logical / Arithmetic Right Shift: `>>`, `>>>`

### Modern Safety Operators
- **Ternary Conditional (`? :`)**:
  ```zuv
  status = score >= 50 ? "Pass" : "Fail"
  ```
- **Nullish Coalescing (`??`)**:
  ```zuv
  port = configPort ?? 8080
  ```
- **Optional Chaining (`?.`)**:
  ```zuv
  street = user?.address?.street
  ```
- **Type Casting (`as`)**:
  ```zuv
  val = rawPtr as num
  bytePtr = buffer as *byte
  ```
- **Type Inspection (`typ`)**:
  ```zuv
  t = typ 42 // "num"
  ```
- **Interned Symbols (`sym`)**:
  ```zuv
  s1 = sym "session_id"
  ```

---

## Ownership, Borrowing & Unsafe Pointers

### Ownership & Borrowing (`&` / `&mut`)
Zuv features a static borrow checker enforcing memory safety at compile time without a runtime garbage collector:

```zuv
// Immutable borrow (&)
val = 42
refVal = &val

// Mutable borrow (&mut)
mut counter = 100
mutRef = &mut counter
```

### Unsafe Blocks & Raw Pointer Dereferencing
For low-level OS interaction and hardware access:

```zuv
ptr = 0x1000 as *num
unsafe {
    *ptr = 42 // Raw memory dereference
}
```

---

## Control Flow & Pattern Matching

### Conditional (`if` / `els`)
```zuv
x = 15
if x > 10 {
    prnt "x is greater than 10"
} els if x == 10 {
    prnt "x is equal to 10"
} els {
    prnt "x is 10 or less"
}
```

### Loops (`wh`, `fr`, `fr of`, `fr in`)
- **While Loop (`wh`)**:
  ```zuv
  mut i = 0
  wh i < 5 {
      i = i + 1
      if i == 3 { cont }
      if i == 5 { brk }
  }
  ```
- **C-Style For Loop (`fr`)**:
  ```zuv
  fr i = 0; i < 10; i = i + 1 {
      prnt i
  }
  ```
- **For-Of Loop (Array Iteration)**:
  ```zuv
  fr item of ["apple", "banana", "cherry"] {
      prnt item
  }

  fr item, idx of items {
      prnt (`${idx}: ${item}`)
  }
  ```
- **For-In Loop (Object Property Iteration)**:
  ```zuv
  fr key in user {
      prnt key
  }

  fr key, val in user {
      prnt (`${key} => ${val}`)
  }
  ```

---

## Destructuring

Zuv supports ergonomic destructuring for objects, arrays, and multi-variable assignments:

```zuv
// 1. Object Destructuring
{ name, age } = user

// 2. Array Destructuring
[ first, second, ...rest ] = items

// 3. Multi-Variable Tuple Unpacking
a, b = getCoordinates()
```

---

## Functions, Generics & Closures

### Function Definition
Functions do not require parentheses around parameter lists or return statement parentheses:

```zuv
add a, b {
    -> a + b
}

greet name {
    prnt "Hello " + name
}
```

The return operator is `->`.

### Generic Functions
Generic parameters are specified within brackets `[T]`:

```zuv
identity[T] item {
    -> item
}
```

### Rest Parameters (`...params`)
```zuv
sumAll ...nums {
    mut total = 0
    fr n of nums {
        total = total + n
    }
    -> total
}
```

---

## Structs, Objects & Enums

### Enums
```zuv
enum Status {
    Pending,
    Active,
    Archived
}

mut current = Status.Active
if current == Status.Active {
    prnt "Account is active"
}
```

### Structs (`obj`) & Methods
```zuv
obj Point {
    x,
    y
}

obj Rectangle {
    width,
    height,

    area {
        -> width * height
    }
}
```

### Instantiation & Anonymous Objects
```zuv
// Named struct instantiation
p = Point { x: 10, y: 20 }

// Anonymous Object Literal
rect = { width: 50, height: 100 }
```

---

## Error Handling

### 1. `ok` / `err` Result Pattern
```zuv
divide a, b {
    if b == 0 {
        -> err "Division by zero"
    }
    -> ok a / b
}

res = divide 10, 2
mch res {
    ok v => prnt (`Result: ${v}`),
    err msg => prnt (`Error: ${msg}`)
}
```

### 2. Structured Exceptions (`try` / `cth` / `fin` / `thr`)
```zuv
try {
    thr "Something went wrong"
} cth ex {
    prnt "Caught: " + ex
} fin {
    prnt "Cleanup completed"
}
```

---

## C Foreign Function Interface (Inbound FFI)

Zuv can directly declare and call foreign C functions from Windows DLLs or POSIX libc:

```zuv
// Declare external Win32 API functions
extern "user32.dll" MessageBoxA hwnd: num, text: str, caption: str, type: num -> num
extern "kernel32.dll" GetTickCount -> num
extern "libc" exit code: num -> void

// Call foreign C functions with zero overhead
MessageBoxA 0, "Hello from native C FFI!", "Zuv Dialog", 0
```

---

## Exporting C Shared Libraries (`pub extern "C"` / `--cdylib`)

Zuv can compile into standard C dynamic libraries (**`.dll`** on Windows, **`.so`** on Linux, **`.dylib`** on macOS) that other languages (Node.js/Bun, Python `ctypes`, C/C++, Rust) can load:

```zuv
// math.zv
pub extern "C" add a: num, b: num -> num {
    -> a + b
}

pub extern "C" multiply a: num, b: num -> num {
    -> a * b
}
```

### Build Command
```powershell
zuv build math.zv --cdylib -o math.dll
```

### Calling from Python (`ctypes`)
```python
import ctypes
lib = ctypes.CDLL("./math.dll")
lib.add.restype = ctypes.c_double
lib.add.argtypes = [ctypes.c_double, ctypes.c_double]

print("10 + 25 =", lib.add(10.0, 25.0))  # 35.0
```

---

## Module Imports (`imp`)

Zuv supports granular and single-line module imports:

```zuv
// Import standard modules
imp str, arr, time, fs, thrd

// Selective function imports from local file
imp addNumbers, multiplyNumbers frm math_helper
```

---

## Async / Await & Multithreading

### Async / Await (`asc` / `awt`)
```zuv
asc fetchData url {
    sl 100 // Simulate async task delay
    -> "Data from " + url
}

main {
    res = awt fetchData "https://api.example.com"
    prnt res
}
```

### Native Multithreading (`wrk`, `spn`, `jn`)
```zuv
imp thrd

wrk workerTask {
    lg "Background thread running..."
}

main {
    hThread = spn workerTask
    jn hThread
    prnt "Worker thread finished execution"
}
```

---

## Standard Library (`std/*`)

| Module | Key Functions / Methods | Description |
| :--- | :--- | :--- |
| `std/str` | `.ln`, `.has sub`, `.idx sub`, `.slc start, end`, `.chr idx` | Native string methods and operations |
| `std/arr` | `.psh val`, `.pop`, `.ln`, `.slc start, end` | Dynamic array manipulation |
| `std/fs` | `rF path`, `wF path, text`, `fE path`, `rmF path`, `sF dir, ext` | File I/O (read, write, exists, remove, scan) |
| `std/time` | `nw`, `sl ms` | High-resolution timestamp (ms), thread sleep |
| `std/thrd` | `spn func`, `jn handle` | Native Win32 / POSIX OS thread spawning & joining |
