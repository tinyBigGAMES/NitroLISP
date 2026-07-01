<div align="center">

![NitroLISP](../media/logo.jpg)

</div>

<a id="what-is-nitrolisp"></a>

## 🚀 What is NitroLISP?

**NitroLISP** is a statically typed, ahead-of-time Lisp dialect with an S-expression surface that compiles to modern **C++23**. You write homoiconic Lisp; NitroLISP analyzes it, expands your macros, optimizes tail calls, and emits clean C++23 that a bundled zig/clang toolchain builds into a native executable.

Its defining idea is a **tiny compiler-known core plus macros that grow the language.** The compiler does not keep gaining features over time; the prelude, written in NitroLISP itself, does.

Write a source file. Run `NLC`. Get a native binary.

```lisp
// hello.nls
#include <cstdio>;

(module exe HelloWorld
  (println "Hello, World!")
  (printf "C++ passthrough: 2 + 3 = %d\n" (+ 2 3)))
```

```
> NLC -s hello.nls -r
Hello, World!
C++ passthrough: 2 + 3 = 5
```

> [!TIP]
> 💡 **Fast path:** read [Getting Started](#getting-started), skim the [Language Reference](#language-reference), then read the [Macro System](#macro-system) to see how the language grows itself.

### 🚦 Documentation Roadmap

| Reader Goal | Start Here | Why |
|-------------|------------|-----|
| 🚀 Run your first program | [Getting Started](#getting-started) | Install assumptions, your first `.nls` file, the `NLC` compiler, and build modes |
| 📘 Learn the language | [Language Reference](#language-reference) | Types, forms, modules, lambdas, control flow, directives, and C++ interop |
| 🧬 Grow the language | [Macro System](#macro-system) | `defmacro`, quasiquote, compile-time eval, the auto-loading prelude, threading |
| 🧾 Verify exact syntax | [BNF Grammar](#nitrolisp-language-grammar) | Formal EBNF grammar and lexical rules |
| 🛠️ Use the toolchain | [Tools](#tools) | Compiler CLI, DAP debugger, LSP server, and the zig/clang C++ build |
| 🔌 Embed NitroLISP | [API Reference](#api-reference) | Consuming a compiled NitroLISP `lib` / `dll` from a host (preview) |
| 🧪 Solve a task | [How-To Guide](#how-to-guide) | Practical recipes with complete examples |

### 💡 Core Idea

NitroLISP is designed around one direct workflow:

```text
write .nls  ->  run NLC  ->  C++23  ->  native executable
```

The core the compiler knows is deliberately small: lambdas and closures, the data and control forms, modules, and a macro engine. Everything ergonomic, `when` / `unless`, threading, and the rest, is a macro living in the auto-loaded prelude. New language features ship as NitroLISP code, not as compiler changes.

> [!IMPORTANT]
> 🧱 NitroLISP is statically typed. Every value has a known type at compile time, and `:` introduces a type annotation, as in `(n : int32)`. There is no dynamic typing and no runtime type tags in user code.

### ✨ Key Features

| Feature | What It Means |
|---------|---------------|
| **🧬 Tiny core, macros grow it** | A small set of compiler-known forms plus a real macro engine. `when`, `unless`, threading, and more are NitroLISP code in the prelude, not compiler features. |
| **🗒️ Homoiconic S-expressions** | Code is data. Cons-cell structure means macros transform the program with the same syntax you write programs in. |
| **⚙️ Compiles to C++23** | NitroLISP emits readable, modern C++23 and builds it with a bundled zig/clang toolchain into a native binary. |
| **🧠 Static typing** | Every value has a known type at compile time. Type annotations use `:` and expose full-width numeric types for clean C interop. |
| **🔁 First-class functions** | `lambda`, closures (compiler-driven free-variable capture), and an `fn` function type backed by `std::function`. |
| **📦 Modules** | `exe`, `dll`, and `lib` modules with `import`, exported declarations, and scope-resolved cross-module calls (`module.fn`). |
| **🌉 C++ interop is first-class** | Raw `#include`, calling `printf` and other C/C++ as S-expression forms. Interop is a designed-in feature, not an escape hatch. |
| **🪜 Tail-call optimization** | Self tail-calls are rewritten to loops, so recursion-as-iteration runs flat without overflowing the stack. |
| **🧰 Zero external dependencies** | The compiler and a bundled zig/clang toolchain ship in a single executable. No separate compiler, linker, or runtime to install. |

### 🏗️ Architecture

```
Source (.nls)
    |
    v
+-------------------------------------------+
|  NitroLISP Pipeline                       |
|                                           |
|  Tokenize --> Parse --> AST               |
|              |                            |
|              v                            |
|  Macro expansion (defmacro rewrite pass)  |
|              |                            |
|              v                            |
|  Semantics (types, scope, modules)        |
|              |                            |
|              v                            |
|  TCO (self tail-calls --> loops)          |
|              |                            |
|              v                            |
|  C++23 emit                               |
|              |                            |
|              v                            |
|  zig / clang build                        |
+-------------------------------------------+
    |
    v
Native executable
```

The pipeline is expressed as `.nld` language-definition files: the S-expression surface, the macro expansion pass, the TCO pass, and the C++23 emitters are all language definition, so the language is defined declaratively rather than hand-coded stage by stage.

> [!NOTE]
> 🧩 The macro pass runs as a rewrite over the program before semantics, and TCO runs as a separate rewrite after macro expansion. Both are isolated passes, which keeps the core forms small and the emitter untouched by either feature.

### 🎯 Who Is This For?

- **Game and application developers** who want to build the whole app in NitroLISP, or embed it in a host, as a native `lib` / `dll` component or a runtime-scriptable interpreter. NitroLISP's first-class C++ interop pairs naturally with engines and existing C/C++ libraries.
- **Lisp and language enthusiasts** who want a small, comprehensible core and a real macro engine, where the interesting language design happens in the prelude rather than buried in a compiler.
- **C++ developers** who want a homoiconic, macro-driven front end that emits readable C++23 they can inspect, build, and integrate.

### 📌 Current Status

The pipeline is working end-to-end, grounded in the language definition, with support for:

- Data types: signed and unsigned integers (`int8`..`int64`, `uint8`..`uint64`), floats (`float32`, `float64`), `boolean`, `char`, `wchar`, `string`, `wstring`, `pointer`, `set`, arrays (fixed and dynamic), records (with inheritance, packed layout, custom alignment, and bitfields), objects (with virtual methods, inheritance, and `self`/`parent`), overlays (unions), choices (enumerations), pointer types, set types, and routine types (function pointers)
- Control forms: `if`, `cond`, `let`, `begin` / `do`, `while`, `for`, `repeat`, `set!`, `return`, and `match`
- Exception handling: `guard`, `except`, and `finally`
- First-class functions: `lambda`, closures, and the `fn` function type
- Modules: `import`, exported declarations, and scope-resolved calls across `exe` / `dll` / `lib` modules
- Macros: `defmacro`, quasiquote / unquote / unquote-splicing templating, compile-time evaluation (list ops, arithmetic, `if` / `cond` / `let`), and `gensym`
- An auto-loading prelude with `when`, `unless`, and the `thread_first` / `thread_last` threading macros
- Tail-call optimization for self-recursive functions
- First-class C++ interop: raw `#include` and C/C++ calls such as `printf` as S-expression forms

> [!TIP]
> 💡 The exact, verified surface syntax always lives in the [BNF Grammar](#nitrolisp-language-grammar). When the prose and the grammar ever seem to disagree, the grammar is authoritative.

### 💻 System Requirements

| Area | Requirement |
|------|-------------|
| **Operating system** | Windows x64 |
| **Runtime dependencies** | None |
| **External toolchain** | None (zig / clang is bundled) |
| **Building from source** | Delphi 12 Athens or higher |

### 🗺️ Table of Contents

- 🚀 [Getting Started](#getting-started): install assumptions, your first `.nls` file, the `NLC` compiler, build modes, and project layout
- 📘 [Language Reference](#language-reference): types, `define` / `let` / `defn` / `lambda`, control flow, modules, directives, exceptions, and C++ interop
- 🧬 [Macro System](#macro-system): `defmacro`, quasiquote, compile-time eval, the auto-loading prelude, and threading
- 🧾 [BNF Grammar](#nitrolisp-language-grammar): formal EBNF grammar and lexical rules
- 🛠️ [Tools](#tools): compiler CLI, DAP debugger, LSP server, and the zig/clang C++ build
- 🔌 [API Reference](#api-reference): consuming a compiled NitroLISP `lib` / `dll` from a host (preview)
- 🧪 [How-To Guide](#how-to-guide): practical recipes for common tasks

<a id="getting-started"></a>

## 🚀 Getting Started

NitroLISP ships as a single compiler executable, **`NLC`** (the NitroLISP Compiler). It carries its own language definition, standard library, and a bundled zig/clang toolchain, so there is nothing else to install: no separate C++ compiler, no linker, no runtime.

> [!NOTE]
> 🪟 NitroLISP targets Windows x64 for the host compiler and can emit for `win64` or `linux64`. The zig/clang toolchain is bundled with `NLC` and resolved relative to the executable.

### ✍️ Your First Program

Create a file named `hello.nls`:

```lisp
// hello.nls
#include <cstdio>;

(module exe HelloWorld
  (println "Hello, World!")
  (printf "C++ passthrough: 2 + 3 = %d\n" (+ 2 3)))
```

Compile and run it in one step with `-r`:

```
> NLC -s hello.nls -r
Hello, World!
C++ passthrough: 2 + 3 = 5
```

That is the whole loop: a `.nls` file goes in, a native binary comes out. The `#include <cstdio>;` line is raw C++ passthrough, and `printf` is called directly as an S-expression form, which is how NitroLISP's first-class C++ interop looks in practice.

> [!TIP]
> 💡 `println` uses `{}` as its format placeholder, as in `(println "fib(30) = {}" (fib 30))`. `printf` uses C-style `%d` / `%lld` placeholders because it is the actual C `printf`.

### 🧭 The Compiler CLI

`NLC` takes one required flag, the source file, plus a few options:

| Flag | Long form | Meaning |
|------|-----------|---------|
| `-s` | `--source <file>` | **Required.** The `.nls` source file to compile. |
| `-o` | `--output <path>` | Output path. Defaults to `output`. |
| `-r` | `--autorun` | Build, then run the compiled binary. |
| `-d` | `--debug` | Build, then debug the compiled binary (DAP debugger). |
| `-h` | `--help` | Show usage. |

```
> NLC -s hello.nls            // compile to ./output
> NLC -s hello.nls -o build   // compile to ./build
> NLC -s hello.nls -r         // compile and run
```

`-r` and `-d` cannot be combined. See [Tools](#tools) for the debugger and the build pipeline in depth.

### ⚙️ Build Configuration

Build settings are **source directives**, written at the top of the file before the module form. They start with `@` and end with a semicolon. They are part of the source, not command-line flags, so a file always builds the same way regardless of how `NLC` is invoked.

```lisp
//@target win64 | linux64;
@target linux64;
@optimize debug;
@subsystem console;

(module exe Fib
  (defn fib ((n : int32)) : int32
    (return (if (< n 2)
                n
                (+ (fib (- n 1)) (fib (- n 2))))))
  (println "fib(30) = {}" (fib 30)))
```

| Directive | Valid values | Purpose |
|-----------|-------------|---------|
| `@target` | `win64`, `linux64` | The platform the emitted C++ is built for. |
| `@optimize` | `debug`, `releasesafe`, `releasefast` (alias: `release`), `releasesmall` | Optimization level for the zig/clang build. |
| `@subsystem` | `console`, `gui` | The subsystem of the produced binary (default: `console`). |

> [!IMPORTANT]
> 🧱 Directives are terminated with `;` and live above the `(module ...)` form. This is the same `;` terminator NitroLISP uses for C++ passthrough lines such as `#include <cstdio>;`, which is why `//` (not `;`) is the comment syntax.

> [!NOTE]
> 🧩 NitroLISP defines a broader directive family, including conditional compilation, version-info, resource, and path directives. See [Build Directives](#build-directives) for the complete reference with all validated values and predefined symbols.

### 🏗️ Build Modes

The output type is determined by the `module` declaration in the source file:

| Module declaration | Output | Description |
|-------------------|--------|-------------|
| `(module exe name ...)` | `name.exe` | Native executable |
| `(module dll name ...)` | `name.dll` | Dynamic link library with exported routines |
| `(module lib name ...)` | (linked inline) | Reusable library module, imported by other modules |

#### DLL: Dynamic Link Library

Use `module dll` and mark exported declarations with `export`:

```lisp
(module dll mylib
  (export (defn calculate ((x : int32) (y : int32)) : int32
    (return (+ (* x x) (* y y))))))
```

#### Library Module

Use `module lib` to produce a reusable library that other modules import. The module name must match the filename:

```lisp
// mathlib.nls
(module lib mathlib
  (export (defn triple ((n : int32)) : int32
    (return (* n 3)))))
```

Import it from another module with full qualification:

```lisp
(module exe main
  (import mathlib)
  (println "{}" (mathlib.triple 7)))   // 21
```

### 🧯 Common First-Run Issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Module not found | Library file is not in the current folder or `@modulepath` | Add `@modulepath "folder";` or move the library next to the main file |
| Unknown symbol | Imported symbol called without module qualification | Use `modulename.symbolname` |
| DLL routine not visible | Routine is not wrapped in `(export ...)` | Mark exported declarations with `export` |
| Build fails with clang errors | Raw C++ passthrough has a syntax error | Check that passthrough lines end with `;` and parens are balanced |

### 🗂️ Project Layout

A NitroLISP program is just one or more `.nls` files. A minimal project is a single source file:

```
my-project/
  hello.nls          // your source
  output/            // created by NLC
    zig-out/
      bin/
        HelloWorld.exe   // built binary, named after the module
```

When you run `NLC -s hello.nls`, the compiler emits C++23, builds it with the bundled toolchain, and writes the binary under the output path. The executable is named after the module (`(module exe HelloWorld ...)` produces `HelloWorld.exe`). With `-r`, `NLC` runs it for you immediately after the build.

Modules can also pull in other modules with `(import name)`, which resolves a `.nls` library module by name, compiles it, and links it. See the [Language Reference](#language-reference) and [Macro System](#macro-system) for how imports carry both functions and macros.

### 🧠 Editor Support

NitroLISP includes a Language Server, so editors that speak LSP can offer live diagnostics and navigation while you edit `.nls` files. The [Tools](#tools) section covers the LSP server and the DAP debugger in detail.

<a id="language-reference"></a>

## 📘 Language Reference

NitroLISP is an S-expression language: every construct is written `(head ...)` and the grammar dispatches on the head. There is no infix syntax and no operator precedence to memorize. `(+ 1 2)` is a call to `+`; `(if c a b)` is the conditional; `(defn f ...)` defines a function. Code is data, which is what makes the [macro system](#macro-system) possible.

Types are static. Every value has a type known at compile time, and `:` introduces a type annotation in the form `name : type`.

### 💬 Comments

```lisp
// line comment
/* block comment */
```

> [!IMPORTANT]
> 🧱 NitroLISP uses `//` for comments, not `;`. The semicolon is the terminator for C++ passthrough lines such as `#include <cstdio>;`, so it cannot also be a comment character.

### 📦 Modules

Every program is a module. The module form names a kind, `exe`, `dll`, or `lib`, then a name, then the body:

```lisp
(module exe HelloWorld
  (println "Hello, World!"))
```

| Kind | Produces |
|------|----------|
| `exe` | An executable program. |
| `dll` | A shared library. |
| `lib` | A reusable NitroLISP library module, imported by other modules. |

The body of a module is a sequence of statements: directives, imports, declarations, and top-level expressions all nest inside the form. The executable is named after the module, so `(module exe HelloWorld ...)` builds `HelloWorld.exe`.

### 🔢 Types

NitroLISP has a concrete type system that maps directly to machine reality. Every value has a known type at compile time, and every primitive has a fixed size.

#### Integer Types

| NitroLISP | C++ | Size | Range |
|-----------|-----|------|-------|
| `int8` | `int8_t` | 1 byte | -128 to 127 |
| `int16` | `int16_t` | 2 bytes | -32,768 to 32,767 |
| `int32` | `int32_t` | 4 bytes | -2,147,483,648 to 2,147,483,647 |
| `int64` | `int64_t` | 8 bytes | Full 64-bit signed range |
| `uint8` | `uint8_t` | 1 byte | 0 to 255 |
| `uint16` | `uint16_t` | 2 bytes | 0 to 65,535 |
| `uint32` | `uint32_t` | 4 bytes | 0 to 4,294,967,295 |
| `uint64` | `uint64_t` | 8 bytes | Full 64-bit unsigned range |

Integer literals default to `int32`. Hex literals use the `0x` or `0X` prefix: `0xFF00`. Append `u` to force an unsigned literal: `42u`.

#### Floating-Point Types

| NitroLISP | C++ | Size | Description |
|-----------|-----|------|-------------|
| `float32` | `float` | 4 bytes | 32-bit IEEE 754 |
| `float64` | `double` | 8 bytes | 64-bit IEEE 754 |

Float literals without a suffix default to `float64`. Append `f` or `F` to force `float32`.

#### Boolean, Character, String, and Pointer Types

| NitroLISP | C++ | Size | Description |
|-----------|-----|------|-------------|
| `boolean` | `bool` | 1 byte | Literals `true` / `false` |
| `char` | `char` | 1 byte | 8-bit character |
| `wchar` | `wchar_t` | 2 bytes | 16-bit wide character |
| `string` | `std::string` | 8 bytes (pointer) | Managed UTF-8 string |
| `wstring` | `std::wstring` | 8 bytes (pointer) | Managed UTF-16 string |
| `pointer` | `void*` | 8 bytes | Untyped pointer; `nil` is the null pointer |
| `set` | `RTSet` | varies | Bit-set type; see [Composite Types](#composite-types) |

> [!NOTE]
> 🔤 The boolean type keyword is `boolean` (not `bool`). Strings are `"..."` for `string` and `w"..."` for `wstring`.

#### Escape Sequences

String literals support these escape sequences: `\n` (newline), `\t` (tab), `\r` (carriage return), `\0` (null), `\\` (backslash), `\"` (quote), `\xNN` (hex byte).

#### Type Widening and Promotion

When two different numeric types appear in the same expression, the type checker automatically widens or promotes:

- **Integer widening:** a narrower signed integer widens to a wider one (`int8` with `int32` becomes `int32`). Same for unsigned (`uint8` with `uint32` becomes `uint32`).
- **Float widening:** `float32` with `float64` becomes `float64`.
- **Integer-to-float promotion:** any integer type with a float type promotes to the float type (`int32` with `float64` becomes `float64`).
- **Char-to-string:** `char` promotes to `string`; `wchar` promotes to `wstring`.

Composite types -- records, objects, overlays, choices, arrays, pointers, sets, and routine types -- are built with the `(type ...)` form and covered in [Composite Types](#composite-types) below.

### 📥 Values and Variables

`define` introduces a typed value with an optional initializer. `const` introduces a compile-time constant.

```lisp
(define total : int32 0)
(define name  : string "NitroLISP")
(const  limit : int32 100)
```

`define` can also bind to an external symbol with `external`:

```lisp
(define errno : int32 external)
```

Mutate a binding with the `set!` form (NitroLISP has no `=` assignment; `=` is the equality operator):

```lisp
(set! total (+ total 1))
```

### 🛠️ Functions

`defn` defines a function. Parameters are a list of `(name : type)` pairs, and the return type follows `:`. The body is a sequence of statements; use `(return expr)` to return a value.

```lisp
(defn fib ((n : int32)) : int32
  (return (if (< n 2)
              n
              (+ (fib (- n 1)) (fib (- n 2))))))
```

Parameters can carry a `var` or `const` modifier, a function can be variadic with `...`, and it can bind to an external or C-linkage symbol:

```lisp
(defn clink puts ((s : string)) : int32 external)  // C linkage + external
(defn sumInts (...) : int32                        // variadic
  (define total : int32 0)
  (for (i 0 to (- (varargs count) 1))
    (set! total (+ total (varargs next int32))))
  (return total))
```

> [!TIP]
> 💡 A leading `clink` or `cpplink` keyword before the function name is the linkage specifier (`cpplink`, C++ linkage, is the default). `external` (optionally followed by a symbol name) declares the function without a body, for binding to existing native code.

### λ Lambdas, the `fn` Type, and Closures

`lambda` is an anonymous function with the same parameter and return-type shape as `defn`. Its body is a sequence of expressions; the value of the last expression is the implicit return.

```lisp
(lambda ((n : int32)) : int32 (* n n))
```

The `fn` type names a function type, `(fn (arg-types) : ret)`, so functions are first-class values you can store and pass:

```lisp
// a routine that takes a function parameter
(defn apply1 ((f : (fn (int32) : int32)) (x : int32)) : int32
  (return (f x)))

// a lambda bound to a variable of fn type
(define square : (fn (int32) : int32)
  (lambda ((n : int32)) : int32 (* n n)))
```

Lambdas capture their environment by value, so a closure survives the scope it was created in:

```lisp
(define adder5 : (fn (int32) : int32)
  (let ((k : int32 5))
    (lambda ((n : int32)) : int32 (+ n k))))   // captures k
```

> [!NOTE]
> 🧠 Free-variable capture is handled by the compiler. At local scope a closure captures by value (C++ `[=]`), which is why an escaping closure like `adder5` keeps working after its `let` exits.

### 🔗 Local Bindings: `let`

`let` introduces local bindings, each `(name : type init)`, then a body. It works as a statement and as an expression (the value of the last body expression is the value of the `let`):

```lisp
(let ((z : int32 5))
  (+ z z))          // evaluates to 10
```

### 🔀 Control Flow

**`if`** takes a condition, a then-form, and an optional else-form. As an expression it is a ternary; as a statement, use `(begin ...)` for multi-statement arms.

```lisp
(if (< n 2) n (+ a b))
```

**`cond`** is a multi-branch conditional. Each branch is `(test body ...)`, with an optional final `(else body ...)`:

```lisp
(cond
  ((= (mod i 15) 0) (println "FizzBuzz"))
  ((= (mod i 3)  0) (println "Fizz"))
  ((= (mod i 5)  0) (println "Buzz"))
  (else             (println "{}" i)))
```

**`begin`** groups a sequence of forms; as an expression its value is the last form.

**`while`** loops while a condition holds. **`for`** counts a variable from a start to an end with `to` (or `downto`). **`repeat`** runs the body, then tests `(until cond)` (a do/while):

```lisp
(while (> n 0)
  (set! n (- n 1)))

(for (i 0 to 9)
  (println "{}" i))

(repeat (until (>= i 10))
  (set! i (+ i 1)))
```

**`match`** dispatches on a value. Each arm is `((labels ...) body ...)`, where a label is a single value or a `lo .. hi` range, with an optional `(else ...)`:

```lisp
(match grade
  ((90 .. 100) (println "A"))
  ((80 .. 89)  (println "B"))
  (else        (println "F")))
```

**`return`** returns from a function (`(return)` or `(return expr)`), **`leave`** breaks out of a loop, and **`skip`** continues to the next iteration.

> [!TIP]
> 💡 `if`, `let`, `cond`, and `begin` are all valid in expression position, so they produce values, not just effects. That is why `(println "{}" (let ((z : int32 5)) (+ z z)))` prints `10`.

### ➕ Operators

Operators are written head-first like any other form. There is no infix syntax and no operator precedence to memorize.

#### Arithmetic

| Form | Operation |
|------|-----------|
| `(+ a b)` | Addition |
| `(- a b)` | Subtraction |
| `(* a b)` | Multiplication |
| `(/ a b)` | Division (float) |
| `(div a b)` | Integer division |
| `(mod a b)` | Modulo (remainder) |
| `(- x)` | Unary negate |
| `(+ x)` | Unary plus |

#### Comparison

| Form | Operation |
|------|-----------|
| `(= a b)` | Equal (emits C++ `==`) |
| `(<> a b)` | Not equal |
| `(< a b)` | Less than |
| `(> a b)` | Greater than |
| `(<= a b)` | Less than or equal |
| `(>= a b)` | Greater than or equal |

#### Logical (short-circuit)

| Form | Operation |
|------|-----------|
| `(and a b)` | Logical AND |
| `(or a b)` | Logical OR |
| `(not x)` | Logical NOT |

#### Bitwise

| Form | Operation |
|------|-----------|
| `(and a b)` | Bitwise AND (context-dependent) |
| `(or a b)` | Bitwise OR (context-dependent) |
| `(xor a b)` | Bitwise XOR |
| `(shl x n)` | Shift left |
| `(shr x n)` | Shift right |

#### Set Membership

| Form | Operation |
|------|-----------|
| `(in elem set)` | True if `elem` is a member of `set` |

> [!IMPORTANT]
> 🧱 There is no `=` assignment in NitroLISP. `=` always means equality. All mutation goes through `(set! target value)`. The C operators `%` and `!=` are also accepted via C++ interop.

### 🖨️ Output

`print` and `println` map to `std::print` / `std::println` and use `{}` as the format placeholder:

```lisp
(println "fib(30) = {}" (fib 30))
(print "no newline")
```

### 🧩 Imports and Visibility

A library module exports declarations with `export`. Another module pulls it in with `import` (one import can name several modules), and calls across modules are scope-resolved with `module.function`:

```lisp
// testlib.nls
(module lib testlib
  (export (defn lib_triple ((n : int32)) : int32
    (return (* n 3)))))
```

```lisp
// caller
(module exe ImportTest
  (import testlib)
  (printf "lib_triple(7) = %d\n" (testlib.lib_triple 7)))
```

> [!NOTE]
> 🧷 A library module's declared name must match its filename, because a qualified call `(testlib.lib_triple ...)` emits `testlib::lib_triple(...)` and must resolve to the module's emitted `namespace testlib`. One `(import ...)` carries both the module's functions and its macros, see the [Macro System](#macro-system).

### 🛡️ Exceptions

`guard` wraps a protected body with an optional `except` handler and an optional `finally` block:

```lisp
(guard
  (println "before throw")
  (raise "boom")
  (except
    (println "caught: code={} msg={}" (exccode) (excmsg)))
  (finally
    (println "cleanup runs always")))
```

| Form | Purpose |
|------|---------|
| `(guard body... (except ...) (finally ...))` | Protected block. Requires `except`, `finally`, or both. |
| `(raise expr)` | Raise an exception with the default software code |
| `(raisecode code msg)` | Raise an exception with an explicit integer code and message |
| `(exccode)` | Get the exception code (inside an `except` block) |
| `(excmsg)` | Get the exception message (inside an `except` block) |

A `guard` block catches both software exceptions (`raise` / `raisecode`) and hardware exceptions (such as divide-by-zero). When both `except` and `finally` are present, `except` comes first.

### 🧠 Memory Management

NitroLISP provides direct memory-management intrinsics for typed allocation, raw blocks, and dynamic arrays:

| Form | Purpose |
|------|---------|
| `(create x)` | Allocate and initialize a typed instance (objects, records via pointer) |
| `(destroy x)` | Free a previously created instance |
| `(getmem n)` | Allocate `n` bytes of raw memory, returns a pointer |
| `(freemem p)` | Free a raw memory pointer |
| `(resizemem p n)` | Resize a raw allocation |
| `(setlength a n)` | Resize a dynamic array |

> [!WARNING]
> Every `(create x)` must have a matching `(destroy x)` to avoid memory leaks. These are general memory operations, not object-specific -- they work for any typed pointer allocation.

### 🧮 Intrinsics

| Form | Returns |
|------|---------|
| `(len x)` | Length of a string, wide string, or dynamic array |
| `(size <Type>)` | Size in bytes of a type (`sizeof`). Takes a type, not a value. |
| `(utf8 x)` | Convert a wide string to a UTF-8 string |
| `(paramcount)` | Number of command-line arguments |
| `(paramstr n)` | The nth command-line argument |
| `(exccode)` | Exception code (inside an `except` block) |
| `(excmsg)` | Exception message (inside an `except` block) |

### 🔁 Variadics

A variadic function declares `(...)` for its parameters and reads arguments with `varargs`:

```lisp
(defn sumInts (...) : int32
  (define total : int32 0)
  (for (i 0 to (- (varargs count) 1))
    (set! total (+ total (varargs next int32))))
  (return total))
```

| Form | Returns |
|------|---------|
| `(varargs count)` | Total number of variadic arguments |
| `(varargs next <Type>)` | Retrieve and consume the next argument as the given type |
| `(varargs get <Index> <Type>)` | Retrieve argument at index without advancing the cursor |
| `(varargs reset)` | Reset the cursor back to the first argument |
| `(varargs copy)` | Copy the current varargs cursor position |

<a id="composite-types"></a>

### 🧱 Composite Types

The `(type Name ...)` form names a new type. It is either a bare type (an alias) or a sub-form whose head names the kind:

```lisp
(type Celsius int32)                       // alias
(type Point (record (x : int32) (y : int32)))  // record (struct)
(type Color (choices Red Green Blue))      // enum
(type Buffer (array (range 0 255) uint8))  // fixed array
(type IntPtr (pointer int32))              // typed pointer
(type Flags (set of Color))                // set
```

#### Records

A record is a value type with named fields (emits a C++ `struct`). Fields are `(name : type)` with an optional bitfield width `(name : type : width)`:

```lisp
(type Point (record
  (x : float32)
  (y : float32)))

(define p : Point)
(set! p.x 10.0)
(set! p.y 20.0)
(println "({}, {})" p.x p.y)
```

**Record inheritance:** use `(parent Base)` to extend a record. The emitted C++ is `struct Derived : public Base { using Super = Base; ... };`:

```lisp
(type Shape (record (x : int32) (y : int32)))
(type Circle (record (parent Shape) (radius : float32)))

(define c : Circle)
(set! c.x 100)
(set! c.y 200)
(set! c.radius 50.0)
```

**Packed layout and alignment:** the bare word `packed` removes padding between fields (emits `#pragma pack(push, 1)`), and `(align N)` sets a custom alignment (emits `alignas(N)`):

```lisp
(type Header (record packed
  (magic   : uint16)
  (version : uint8)
  (flags   : uint8)))

(type AlignedVec (record (align 16)
  (x : float32) (y : float32) (z : float32) (w : float32)))
```

**Bitfields:** specify a bit width after a second `:` for compact binary layouts:

```lisp
(type Flags (record packed
  (visible  : uint8 : 1)
  (enabled  : uint8 : 1)
  (priority : uint8 : 3)
  (reserved : uint8 : 3)))
```

**Anonymous overlays and records** can nest inside a record for C-style union-in-struct patterns:

```lisp
(type Variant (record
  (kind : int32)
  (overlay
    (int_val   : int64)
    (float_val : float64)
    (str_val   : string))))
```

#### Objects

Objects are heap-allocated types with fields and `virtual` methods. They are NitroLISP's affordance for when the C++ boundary demands a real class with a vtable -- for example, implementing or overriding a C++ interface. Use `(create x)` / `(destroy x)` for allocation and deallocation; `self` accesses the current instance and `parent` calls the base:

```lisp
(type TCounter (object
  (value : int32)
  (method increment () : void
    (set! self.value (+ self.value 1)))
  (method get_value () : int32
    (return self.value))))

(define c : (pointer to TCounter))
(create c)
(set! c.value 0)
(c.increment)
(c.increment)
(println "count: {}" (c.get_value))   // count: 2
(destroy c)
```

**Object inheritance:** use `(parent Base)` to derive. Methods are `virtual`, so a derived method overrides the base. Call the base implementation with `parent`:

```lisp
(type TBase (object
  (x : int32)
  (method describe () : int32
    (return (* self.x 10)))))

(type TDerived (object (parent TBase)
  (y : int32)
  (method describe () : int32
    (return (+ (parent.describe) self.y)))))
```

> [!NOTE]
> 🧩 Object methods emit as `virtual`, so derived types can override them. Objects give you real C++ class semantics with a vtable, single inheritance, and `self` / `parent` dispatch.

#### Overlays (Unions)

Overlays share storage between fields (emit a C++ `union`):

```lisp
(type Value (overlay
  (as_int   : int32)
  (as_float : float32)
  (as_bytes : (array (range 0 3) uint8))))
```

#### Choices (Enumerations)

Choices declare named enumeration values. Values may be bare (auto-numbered) or given an explicit integer:

```lisp
(type Direction (choices North South East West))
(type Color (choices (Red 0) (Green 1) (Blue 2)))
```

#### Arrays

A fixed array uses a `(range lo hi)` sub-form; a dynamic array omits bounds:

```lisp
(type Grid (array (range 0 7) int32))          // fixed: std::array<int32_t, 8>
(type Items (array int32))                      // dynamic: std::vector<int32_t>
```

Use `(setlength arr n)` to resize a dynamic array and `(len arr)` to query its current length.

#### Pointer Types

A typed pointer names its target; `const` makes it read-only:

```lisp
(type PInt32     (pointer int32))               // int32_t*
(type PConstInt  (pointer to const int32))       // const int32_t*
```

A bare `(pointer)` with no target is `void*`.

#### Set Types

A set over a range is a `std::bitset`; a set over a type is a `std::set`:

```lisp
(type CharSet (set (range 0 255)))              // std::bitset<256>
(type IntSet  (set of int32))                    // std::set<int32_t>
```

Set literals use square brackets with optional `(range lo hi)` elements: `[1 3 (range 5 9)]`.

#### Routine Types (Function Pointers)

A routine type declares a callable with an optional linkage and parameter/return types:

```lisp
(type TCompare (routine ((a : int32) (b : int32)) : int32))
(type TCCallback (routine clink ((data : pointer)) : void))
```

Routine types use C calling convention by default. Add `clink` for explicit C linkage or `cpplink` for C++ ABI compatibility.

> [!NOTE]
> 🧠 Routine types (`routine`) are C function pointers. The `fn` type (`(fn (int32) : int32)`) is a `std::function` -- a first-class callable value that can hold a lambda or closure. Use `routine` for C interop callbacks; use `fn` for NitroLISP closures and higher-order functions.

### 📍 Pointers

Take an address with `(address x)` (the `of` word is optional: `(address of x)`). Dereference with the postfix `^` operator (`p^`). Cast a value to a typed pointer with `(pointer to <Type> expr)` (the `to` is optional, and `const` may follow it):

```lisp
(define p : pointer (address total))
(define v : int32 p^)
```

Postfix operators bind to the preceding operand: `a[i]` indexes an array, `a.field` accesses a field, and `a^` dereferences a pointer.

### 🎛️ Directives

Directives configure the build and are written `@name value;` at the top of a file. The common build directives, `@target`, `@optimize`, `@subsystem`, are covered in [Getting Started](#getting-started). NitroLISP also supports conditional compilation (`@define`, `@undef`, `@ifdef`, `@ifndef`, `@elseif`, `@else`, `@endif`) and a family of version-info and resource directives. The full directive reference with validated values lives in [Build Directives](#build-directives).

### 🧪 Unit Testing

NitroLISP has a built-in unit testing framework. Test blocks are written inside the module, gated by the `@unittestmode` directive. When `@unittestmode on;` is active, the compiler compiles test blocks and replaces the normal entry point with the test runner.

```lisp
@unittestmode on;

(module exe MathTests
  (defn add ((a : int32) (b : int32)) : int32
    (return (+ a b)))

  (test "addition"
    (testAssertEqualInt 5 (add 2 3))
    (testAssertEqualInt 0 (add -1 1)))

  (test "comparisons"
    (testAssertTrue (> 10 5))
    (testAssertFalse (< 10 5))))
```

#### Assertion Family

All assertions continue after failure -- failures accumulate and are reported per test. The compiler injects source file and line number automatically.

| Form | Description |
|------|-------------|
| `(testAssert expr)` | Fails if expression is false |
| `(testAssertTrue expr)` | Fails if expression is not true |
| `(testAssertFalse expr)` | Fails if expression is not false |
| `(testAssertEqualInt expected actual)` | Fails if signed integers are not equal |
| `(testAssertEqualUInt expected actual)` | Fails if unsigned integers are not equal |
| `(testAssertEqualFloat expected actual)` | Fails if floats are not equal |
| `(testAssertEqualStr expected actual)` | Fails if strings are not equal |
| `(testAssertEqualBool expected actual)` | Fails if booleans are not equal |
| `(testAssertEqualPtr expected actual)` | Fails if pointers are not equal |
| `(testAssertNil expr)` | Fails if expression is not nil |
| `(testAssertNotNil expr)` | Fails if expression is nil |
| `(testFail "message")` | Unconditional failure with a message |

> [!TIP]
> 💡 Test blocks have access to all module declarations. Use tests to verify routines, types, and module behavior without building a separate test harness.

### 🌉 C++ Interop

Interop is not a feature bolted onto NitroLISP. It is what NitroLISP *is*. The Lisp surface is compiled to modern **C++23**, and from that point your program is an ordinary C++ translation unit that a real C++ toolchain (the bundled zig/clang) compiles and links. There is no FFI, no binding layer, no marshalling boundary, because there is no boundary -- the code already *is* C++. Anything C++ can do with C and C++ code, at the source level and at the binary level, NitroLISP can do too.

That gives you the full interop matrix, in both directions:

| | Consume C / C++ | Produce for C / C++ |
|---|---|---|
| **Source level** | `#include` any header and call C/C++ functions as ordinary forms; declare `external` symbols with a typed signature | your NitroLISP-as-C++ shares one translation unit with any raw C++ you write inline |
| **Binary level** | link a `.lib` or load a `.dll` built by any modern C/C++ compiler and call it through `external` declarations | compile a `(module lib ...)` / `(module dll ...)` to a `.lib` / `.dll` that a modern C/C++ compiler links against, with `export` controlling visibility |

#### Source level: raw C/C++ in the same file

Pull in C/C++ headers with a raw `#include` passthrough line, terminated with `;`, then call the functions directly as ordinary S-expression forms:

```lisp
#include <cstdio>;

(module exe HelloWorld
  (println "Hello, World!")
  (printf "C++ passthrough: 2 + 3 = %d\n" (+ 2 3)))
```

Here `printf` is the real C `printf`, called as `(printf fmt args ...)` with C-style `%d` placeholders. The standard library and existing C/C++ code are available with no binding layer, because the surface is Lisp but the substance is C++. Verbatim C++ can go straight into the file through the same passthrough, so even a construct with no dedicated Lisp form is still reachable, all in one translation unit.

> [!IMPORTANT]
> 🧱 A passthrough line ends with `;`, the same terminator the lexer uses to know where raw C++ stops and S-expression forms resume. This is the reason `//`, not `;`, is the comment syntax in NitroLISP.

#### Binary level: ship and consume native libraries

Because the output is a normal native artifact, NitroLISP also interoperates with C and C++ across the linker. Compile a `lib` or `dll` module and a modern C/C++ compiler can link against it; `export` marks what is visible on the boundary:

```lisp
(module lib mathlib
  (export (defn triple ((n : int32)) : int32
    (return (* n 3)))))
```

Going the other way, declare a function that lives in a linked `.lib` / `.dll` with `external` and call it like any other form:

```lisp
#include <cmath>;

(defn clink c_pow ((base : float64) (exp : float64)) : float64 external "pow")
```

`clink` selects the C ABI (`extern "C"`, a plain unmangled symbol); `cpplink` selects C++ linkage and is the default, so a bare `(defn name ...)` is already `cpplink`. `external` marks the function as implemented elsewhere; `(define name : T external)` does the same for a global.

> [!NOTE]
> 🔗 The `clink` / `cpplink` choice is the one real interop decision, and it is inherited straight from C++, not a NitroLISP quirk. **`clink`** is a stable, universal ABI: `extern "C"` functions export under plain, unmangled names and link cleanly against MSVC, gcc, or clang. **`cpplink`** (the default) is richer -- overloads, namespaces, classes -- but toolchain-bound, since name mangling, the exception ABI, and STL layout are compiler- and version-specific. Overloading rides on mangling, so an overloaded routine must be `cpplink`; the compiler will not overload a `clink` symbol. Choose per export where on that spectrum to sit. The artifact-level detail lives in [Embedding and Scripting](#api-reference).

> [!TIP]
> 💡 Because NitroLISP *is* C++ underneath, "what can it interoperate with?" has the same answer as "what can C++ interoperate with?" -- the entire C and C++ ecosystem, at both source and binary level, in both directions.

<a id="build-directives"></a>

## 🎛️ Build Directives

Directives are compile-time instructions prefixed with `@` and terminated with `;`. They live at the top of a source file, before the `(module ...)` form. Directives are part of the source, not command-line flags, so a file always builds the same way regardless of how `NLC` is invoked.

### 📄 Build Configuration

| Directive | Value type | Valid values | Effect |
|-----------|-----------|--------------|--------|
| `@target` | identifier | `win64`, `linux64` | Target platform for the emitted C++ build |
| `@optimize` | identifier | `debug`, `releasesafe`, `releasefast` (alias: `release`), `releasesmall` | Optimization level passed to the zig/clang toolchain |
| `@subsystem` | identifier | `console`, `gui` | Application subsystem (default: `console`) |

```lisp
@target win64;
@optimize releasesafe;
@subsystem console;
```

### 🏷️ Version Information

Embed Windows version information in the output binary:

| Directive | Value type | Description |
|-----------|-----------|-------------|
| `@addverinfo` | identifier | Enable version info embedding (any non-empty value enables) |
| `@vimajor` | integer | Major version number |
| `@viminor` | integer | Minor version number |
| `@vipatch` | integer | Patch version number |
| `@viproductname` | string | Product name |
| `@videscription` | string | File description |
| `@vifilename` | string | Original filename |
| `@vicompanyname` | string | Company name |
| `@vicopyright` | string | Copyright string |

```lisp
@addverinfo true;
@vimajor 1;
@viminor 0;
@vipatch 0;
@viproductname "My Application";
@videscription "A NitroLISP application";
@vicompanyname "My Company";
@vicopyright "Copyright 2026";
```

### 📁 Resource and Path

| Directive | Value type | Description |
|-----------|-----------|-------------|
| `@exeicon` | string | Set the application icon (EXE only) |
| `@copydll` | string | Copy a DLL to the output directory during build |
| `@linklibrary` | string | Link against a native library |
| `@librarypath` | string | Add a library search path |
| `@modulepath` | string | Add a module (import) search path |
| `@includepath` | string | Add a C++ include path |

```lisp
@linklibrary "raylib";
@librarypath "libs/win64";
@includepath "include";
@modulepath "libs/std";
```

### 🐞 Debug and Diagnostic

| Directive | Value type | Description |
|-----------|-----------|-------------|
| `@breakpoint` | (none) | Insert a debugger breakpoint at this source location |
| `@unittestmode` | identifier | `on` or `off` (default: `off`). When `on`, test blocks are compiled and the test runner replaces the normal entry point. |
| `@message` | severity + string | Emit a compile-time diagnostic. Severity: `hint`, `info`, `note`, `warn`, `error`, `fatal`. |

### 🔀 Conditional Compilation

Conditional compilation lets a source file include or exclude code based on defined symbols. These directives are handled by the lexer preprocessor, not the grammar:

| Directive | Description |
|-----------|-------------|
| `@define SYM` | Define a symbol |
| `@undef SYM` | Undefine a symbol |
| `@ifdef SYM` | Compile the following block if symbol is defined |
| `@ifndef SYM` | Compile the following block if symbol is NOT defined |
| `@elseif SYM` | Alternative branch with condition |
| `@else` | Alternative branch |
| `@endif` | End conditional block |

```lisp
@define VERBOSE;

@ifdef VERBOSE
  (println "verbose mode is on")
@endif

@ifdef WIN64
  (println "running on 64-bit Windows")
@endif
```

#### 🏁 Predefined Symbols

| Symbol | Defined when |
|--------|-------------|
| `NITROLISP` | Always |
| `TARGET_WIN64` | Target is `win64` |
| `WIN64` | Target is `win64` |
| `MSWINDOWS` | Target is `win64` |
| `WINDOWS` | Target is `win64` |
| `CPUX64` | Always (x64-only target) |
| `TARGET_LINUX64` | Target is `linux64` |
| `LINUX` | Target is `linux64` |
| `POSIX` | Target is `linux64` |
| `UNIX` | Target is `linux64` |

<a id="macro-system"></a>

## 🧬 The Macro System

The macro system is the heart of NitroLISP. The compiler knows a deliberately small set of forms; everything else, `when`, `unless`, threading, and any language feature you add, is a macro written in NitroLISP. The compiler does not grow new features over time. The prelude does.

This is possible because NitroLISP is homoiconic: code is data. A macro is a compile-time function that receives the unevaluated argument forms, computes a replacement form, and the compiler substitutes it in place before semantics and code generation run.

> [!IMPORTANT]
> 🧱 Macros run at compile time, as a rewrite pass over the program before semantics. By the time your code is type-checked and emitted, every macro call has already been replaced by the code it produced.

### 🪄 `defmacro`

`defmacro` defines a macro: a name, a parameter list, and a body that is evaluated at expansion over the call's argument forms.

```lisp
(defmacro twice (x) (+ x x))
```

A macro can take a variadic body with `...`, which is how statement macros accept a block of forms (see `when` / `unless` below).

### 💬 Quote, Quasiquote, Unquote

`quote` yields a form as literal data. `quasiquote` builds a template, and `unquote` marks the holes in that template to be filled with the macro's arguments at expansion:

```lisp
(defmacro square (x)
  (quasiquote (* (unquote x) (unquote x))))

(defmacro shout (msg)
  (quasiquote (println (unquote msg))))

(println "{}" (square 5))          // expands to (* 5 5) -> 25
(println "{}" (square (+ 2 1)))    // expands to (* (+ 2 1) (+ 2 1))
```

> [!NOTE]
> 🔁 `square` uses its argument twice, so `(square (+ 2 1))` substitutes the whole `(+ 2 1)` form into both holes. The template machinery copies forms structurally, so the expanded code is `(* (+ 2 1) (+ 2 1))`, not a pre-computed `3`.

### 🪢 Unquote-Splicing: `(splice X)`

Inside a quasiquote, `(splice X)` splices the elements of a list `X` into the surrounding form, rather than inserting the list as a single element:

```lisp
(defmacro show3 (xs)
  (quasiquote (println "{} {} {}" (splice xs))))

(defmacro showWith (a rest)
  (quasiquote (println "{} {} {}" (unquote a) (splice rest))))
```

NitroLISP uses the word-forms `quote`, `quasiquote`, `unquote`, and `splice` rather than the `'` `` ` `` `,` `,@` reader sugar.

### ⚙️ Compile-Time Evaluation

A macro body is evaluated at expansion, so it can compute with real list operations, `list`, `cons`, `car`, `cdr`, and `append`, to build the form it returns:

```lisp
(defmacro firstof (a b)
  (car (list a b)))                       // -> a

(defmacro secondof (a b)
  (car (cdr (list a b))))                 // -> b

(defmacro appendsecond (a b)
  (car (cdr (append (list a) (list b))))) // -> b
```

The evaluator also supports arithmetic and the `if` / `cond` / `let` forms at expansion time, so macros can branch and bind while they compute their output.

### 🧼 Hygiene: `gensym`

`gensym` produces a fresh, unique name, so a macro can introduce a temporary binding without colliding with names in the calling code:

```lisp
(defmacro dbl (x)
  (let ((g : int32 (gensym)))
    (quasiquote
      (let (((unquote g) : int32 (unquote x)))
        (+ (unquote g) (unquote g))))))
```

### 📚 The Auto-Loading Prelude

The standard prelude is the always-on macro layer. It loads automatically, before your code is expanded, with no import required. This is where the language-growing standard macros live, so forms like `when`, `unless`, and the threading macros are available everywhere without ceremony.

> [!TIP]
> 💡 The prelude proves the central idea: these are not compiler features, they are ordinary NitroLISP macros that happen to ship with the language. You can write your own the same way.

### 🔀 `when` and `unless`

`when` runs its body when a condition is true; `unless` runs its body when a condition is false. Both take a multi-statement body:

```lisp
(when (> 5 3)
  (set! x 10)
  (println "when ran"))

(unless (> 1 2)
  (println "unless ran"))
```

Because expansion is statement-aware, a `when` in statement position emits a real `if` with a braced block, not a both-arms ternary.

### ➡️ Threading: `thread_first` and `thread_last`

The threading macros fold a value through a sequence of steps. `thread_first` inserts the running value as the FIRST operand of each step; `thread_last` inserts it as the LAST. Since subtraction is not commutative, the two differ:

```lisp
(thread_first 100 (sub 10) (sub 30))   // (sub (sub 100 10) 30) = 60
(thread_last  100 (sub 10) (sub 30))   // (sub 30 (sub 10 100))  = 120
(thread_first 7 dbl)                    // (dbl 7)               = 14
(thread_first 5 (+ 2) (* 3))            // (* (+ 5 2) 3)         = 21
```

> [!NOTE]
> 🧵 Clojure's `->` / `->>` names are not used: `->` is the C++ pointer-member operator and NitroLISP admits no C/C++ operator constructs at its surface. The heads are plain identifiers, `thread_first` and `thread_last`, instead.

### 📦 One Import Carries Functions and Macros

A library is one thing. A single `(import name)` delivers both the library's functions and its macros, there is no separate macro-import mechanism:

```lisp
(module exe ImportMacroTest
  (import logging)
  (logging.loginfo "import macro ok")   // a macro from the lib
  (println "{}" (logging.logtag 3)))    // a function from the lib
```

Macros are module-namespaced exactly like functions. A macro defined in module `logging` is used qualified as `logging.loginfo`, just as a function would be `logging.logtag`. Same-module and prelude macros are used bare; cross-module macros use the qualifier. Two modules can define a macro of the same name without colliding.

> [!IMPORTANT]
> 🧷 Imports are processed before macro expansion, so an imported library's macros are registered in time to expand the importing module. That is what lets a third party ship language extensions as an ordinary library.

# NitroLISP Language Grammar

Grammar in EBNF (ISO/IEC 14977).

Notation: `,` concatenation, `|` alternation, `{ }` zero or more,
`[ ]` optional, `( )` grouping, `"..."` literal source text. Non-terminals
are lowercase words. This file is the surface-syntax contract: it describes
what the programmer writes. Every construct is an S-expression `(head ...)`;
the grammar dispatches on the head. There are no infix operators -- `(` always
starts a new S-expression, never a grouping.

### Program

```ebnf
program         = { directive | module } ;
module          = "(" , "module" , module kind , symbol , { module element } , ")" ;
module kind     = "exe" | "dll" | "lib" ;
module element  = directive | import | export | declaration | statement ;
import          = "(" , "import" , symbol , { symbol } , ")" ;
export          = "(" , "export" , declaration , ")" ;
```

### Declarations

```ebnf
declaration     = function | variable | constant | type decl | method | macro ;

function        = "(" , "defn" , [ linkage ] , symbol , param list ,
                  [ ":" , type ] , ( body | external ) , ")" ;
linkage         = "clink" | "cpplink" ;
param list      = "(" , [ ( param , { param } , [ "..." ] ) | "..." ] , ")" ;
param           = "(" , [ "var" | "const" ] , symbol , ":" , type , ")" ;
external        = "external" , [ cstring | symbol ] ;

variable        = "(" , ( "define" | "var" ) , symbol , ":" , type ,
                  ( [ expression ] | external ) , ")" ;
constant        = "(" , "const" , symbol , [ ":" , type ] , expression , ")" ;
method          = "(" , "method" , symbol , param list , [ ":" , type ] , body , ")" ;
macro           = "(" , "defmacro" , symbol , "(" , { symbol } , [ "..." , symbol ] , ")" , body , ")" ;
lambda          = "(" , "lambda" , param list , [ ":" , type ] , body , ")" ;
```

- `linkage` is an optional linkage keyword, `clink` (C ABI) or `cpplink` (C++ linkage, the default), e.g. `(defn clink name ...)`.
- A trailing `...` in a param list marks the routine variadic; `(varargs ...)`
  reads the arguments.
- `external` replaces a body to bind an externally linked symbol.
- `lambda` is an anonymous function value (a first-class callable). The value
  of the last body expression is the implicit return.

### Type Declarations

```ebnf
type decl       = "(" , "type" , symbol , type def , ")" ;
type def        = alias | record | object | overlay | choices
                | array type | pointer type | set type | routine type ;

alias           = type ;

record          = "(" , "record" , { record item } , ")" ;
record item     = "packed" | "(" , "parent" , symbol , ")" | "(" , "align" , integer , ")"
                | field | anon overlay ;
object          = "(" , "object" , { "(" , "parent" , symbol , ")" | field | method } , ")" ;
overlay         = "(" , "overlay" , { field | anon record } , ")" ;
anon overlay    = "(" , "overlay" , { field | anon record } , ")" ;
anon record     = "(" , "record" , { field } , ")" ;
field           = "(" , symbol , ":" , type , [ ":" , integer ] , ")" ;
choices         = "(" , "choices" , { symbol | "(" , symbol , expression , ")" } , ")" ;

range           = "(" , "range" , integer , integer , ")" ;
array type      = "(" , "array" , [ range ] , type , ")" ;
pointer type    = "(" , "pointer" , [ "to" ] , [ "const" ] , type , ")" ;
set type        = "(" , "set" , [ range | type ] , ")" ;
routine type    = "(" , "routine" , [ linkage ] , param list , [ ":" , type ] , ")" ;
```

- Field bit width: `(name : type : width)`.
- Ranges are head-first: `(range lo hi)`. There is no infix `..`. A fixed array
  is `(array (range 0 9) int32)`, a dynamic one `(array int32)`; a bit-set over
  a range is `(set (range 0 255))`.
- `record` supports single inheritance `(parent Base)`, `packed`, `(align N)`,
  and nested anonymous `overlay`/`record` groups for C data interop.
- `object` is a class (fields + methods); `overlay` is a union; `choices` is
  an enum. `choices` values may be bare or given an explicit value.

### Statements

```ebnf
statement       = if | cond | let | begin | while | for | repeat
                | set | return | leave | skip | match | guard
                | raise | memory | output | test | expr stmt ;

if              = "(" , "if" , expression , statement , [ statement ] , ")" ;
cond            = "(" , "cond" , { "(" , ( expression | "else" ) , body , ")" } , ")" ;
let             = "(" , "let" , "(" , { binding } , ")" , body , ")" ;
binding         = "(" , symbol , ":" , type , expression , ")" ;
begin           = "(" , "begin" , body , ")" ;
while           = "(" , "while" , expression , body , ")" ;
for             = "(" , "for" , "(" , symbol , expression , ( "to" | "downto" ) , expression , ")" , body , ")" ;
repeat          = "(" , "repeat" , "(" , "until" , expression , ")" , body , ")" ;
set             = "(" , "set!" , place , expression , ")" ;
place           = symbol | postfix ;
return          = "(" , "return" , [ expression ] , ")" ;
leave           = "(" , "leave" , ")" ;
skip            = "(" , "skip" , ")" ;
match           = "(" , "match" , expression ,
                  { "(" , "(" , { match label } , ")" , body , ")" | "(" , "else" , body , ")" } , ")" ;
match label     = expression | range ;
guard           = "(" , "guard" , body ,
                  [ "(" , "except" , body , ")" ] , [ "(" , "finally" , body , ")" ] , ")" ;
raise           = "(" , "raise" , expression , ")"
                | "(" , "raisecode" , expression , expression , ")" ;
memory          = "(" , memory op , { expression } , ")" ;
memory op       = "getmem" | "freemem" | "resizemem" | "setlength" | "create" | "destroy" ;
output          = "(" , ( "print" | "println" ) , { expression } , ")" ;
test            = "(" , "test" , cstring , body , ")"
                | "(" , test assert , { expression } , ")" ;
test assert     = "testAssert" | "testAssertTrue" | "testAssertFalse"
                | "testAssertEqualInt" | "testAssertEqualUInt" | "testAssertEqualFloat"
                | "testAssertEqualStr" | "testAssertEqualBool" | "testAssertEqualPtr"
                | "testAssertNil" | "testAssertNotNil" | "testFail" ;
expr stmt       = "(" , expression , { expression } , ")" ;
```

- `if`, `while`, `for`, `cond`, `match`, `guard` arms take single statements;
  use `(begin ...)` to sequence several.
- `for` and `repeat` write their loop headers in parentheses:
  `(for (i 0 to 10) ...)`, `(repeat (until done) ...)`. `repeat` runs the body
  then tests (do/while semantics).
- `raise msg` raises with the default software code; `raisecode code msg`
  raises with an explicit integer code. The code and message are read in an
  `except` block via `(exccode)` and `(excmsg)`.

### Expressions

```ebnf
expression      = atom | quote | quasiquote | binary | unary | logical
                | membership | intrinsic | address | varargs | pointer cast
                | lambda | if | cond | let | begin | set | output
                | call | postfix ;
quote           = "(" , "quote" , datum , ")" ;
quasiquote      = "(" , "quasiquote" , template , ")" ;
template        = atom | "(" , "unquote" , expression , ")"
                | "(" , "unquote-splicing" , expression , ")"
                | "(" , { template } , ")" ;
binary          = "(" , binary op , expression , expression , ")" ;
binary op       = "+" | "-" | "*" | "/" | "%" | "=" | "<>" | "!=" | "<" | ">" | "<=" | ">="
                | "div" | "mod" | "shl" | "shr" | "xor" ;
unary           = "(" , ( "-" | "+" ) , expression , ")" ;
logical         = "(" , "and" , expression , expression , ")"
                | "(" , "or" , expression , expression , ")"
                | "(" , "not" , expression , ")" ;
membership      = "(" , "in" , expression , expression , ")" ;
intrinsic       = "(" , "len" , expression , ")"
                | "(" , "size" , type , ")"
                | "(" , "utf8" , expression , ")"
                | "(" , "paramcount" , ")"
                | "(" , "paramstr" , expression , ")"
                | "(" , "exccode" , ")"
                | "(" , "excmsg" , ")"
                | "(" , "getmem" , expression , ")"
                | "(" , "resizemem" , expression , expression , ")" ;
address         = "(" , "address" , [ "of" ] , expression , ")" ;
varargs         = "(" , "varargs" , ( "count" | "reset" | "copy" | "next" , type ) , ")" ;
pointer cast    = "(" , "pointer" , [ "to" ] , [ "const" ] , type , expression , ")" ;
call            = "(" , expression , { expression } , ")" ;
postfix         = expression , { field access | index | deref } ;
field access    = "." , symbol ;
index           = "[" , expression , "]" ;
deref           = "^" ;
body            = { statement } ;
datum           = atom | "(" , { datum } , ")" ;
```

- The same unary form `(- x)` / `(+ x)` is distinguished from binary by
  operand count.
- `size` takes a type and yields its byte size (sizeof); every other intrinsic
  takes value expressions.
- `if`, `cond`, `let`, `begin`, `set!`, `print`/`println` may appear in
  expression position and yield a value (the print forms yield 0).
- `%` and `!=` are the C spellings of `mod` and `<>`, available via C++ interop.

### Types

```ebnf
type            = scalar | "string" | "wstring" | "char" | "wchar" | "pointer"
                | array type | pointer type | set type | routine type | fn type
                | qualified name ;
scalar          = "int8" | "int16" | "int32" | "int64"
                | "uint8" | "uint16" | "uint32" | "uint64"
                | "float32" | "float64" | "boolean" ;
fn type         = "(" , "fn" , "(" , { type } , ")" , ":" , type , ")" ;
qualified name  = symbol , { "." , symbol } ;
```

- `fn` is a first-class callable value type: `(fn (int32 int32) : int32)` maps
  to a `std::function`. It is distinct from `routine` (a C function pointer).
- `qualified name` covers user types and module-qualified types (`Module.Type`).

### Atoms

```ebnf
atom            = symbol | integer | float | cstring | wstring | character
                | boolean | "nil" | "self" | "parent" | set literal ;
boolean         = "true" | "false" ;
set literal     = "[" , { set element } , "]" ;
set element     = expression | range ;
```

### Lexical

```ebnf
symbol          = ( letter | "_" ) , { letter | digit | "_" } ;
integer         = ( decimal | hexadecimal ) , [ "u" ] ;
decimal         = digit , { digit } ;
hexadecimal     = ( "0x" | "0X" ) , hex digit , { hex digit } ;
float           = digit , { digit } , "." , digit , { digit } , [ ( "e" | "E" ) , [ "+" | "-" ] , digit , { digit } ] , [ "f" | "F" ] ;
cstring         = '"' , { character | escape } , '"' ;
wstring         = "w" , cstring ;
character       = "'" , ? one character ? , "'" ;
escape          = "\" , ( "n" | "t" | "r" | "0" | "\" | "'" | '"' | "x" , hex digit , hex digit ) ;
letter          = ? "A" .. "Z" | "a" .. "z" ? ;
digit           = ? "0" .. "9" ? ;
hex digit       = ? "0" .. "9" | "a" .. "f" | "A" .. "F" ? ;
```

- Identifiers and keywords are case-sensitive.
- `42u` is an unsigned integer literal; a `f`/`F` float suffix forces `float32`.
- `character` uses the C char literal `'x'` (provided via C++ interop).

### Reserved Words

Keywords are reserved and cannot be used as identifiers. Every word below is
consumed by a current form: `var` and `const` are parameter modifiers (by-ref
and by-value-const) and declaration heads; `routine` is a function-pointer
type constructor; `until` is the
`repeat` header test. The Pascal scaffolding words `do`, `then`, `end` and the
type-test `is` are NOT reserved -- NitroLISP is committed S-expression, so they
carry no meaning and are free for use as identifiers and C++ interop names.

```
address   align     and       array     begin     boolean   char
choices   const     cond      create    define    defmacro  defn
destroy   div       downto    else      except    export    external
false     finally   for       freemem   getmem    guard     if
import    in        lambda    leave     len       let       match
method    mod       module    nil       not       object    of
or        overlay   packed    parent    pointer   print     println
quasiquote quote    raise     raisecode range     record    repeat
resizemem return    routine   self      set       setlength shl
shr       size      skip      to        true      type      until
utf8      var       varargs   while     xor
exccode   excmsg    paramcount paramstr
```

Test keywords (used inside `test` blocks): `test`, `testAssert`,
`testAssertTrue`, `testAssertFalse`, `testAssertEqualInt`,
`testAssertEqualUInt`, `testAssertEqualFloat`, `testAssertEqualStr`,
`testAssertEqualBool`, `testAssertEqualPtr`, `testAssertNil`,
`testAssertNotNil`, `testFail`.

### Operators and Delimiters

```ebnf
operator        = ":=" | "+=" | "-=" | "*=" | "/=" | "<>" | "<=" | ">="
                | "..." | "=" | "<" | ">" | "+" | "-" | "*" | "/"
                | "^" | "|" | "&" ;
delimiter       = "(" | ")" | "[" | "]" | "," | ":" | ";" | "." ;
```

- `(` `)` carry S-expression structure. `:` separates a name from its type in
  typed bindings `(name : type)`.
- `:=` `+=` `-=` `*=` `/=` are reserved (Myra heritage); mutation is written
  `(set! ...)`. `^` is pointer dereference (postfix), `&` is address-of.

### Comments

```ebnf
comment         = line comment | block comment ;
line comment    = "//" , { ? any character except newline ? } ;
block comment   = "/*" , { ? any character ? } , "*/" ;
```

### Directives

```ebnf
directive       = "@" , directive name , { symbol | cstring | integer } , [ ";" ] ;
directive name  = "define" | "undef" | "ifdef" | "ifndef" | "elseif" | "else" | "endif"
                | "exeicon" | "copydll" | "linklibrary" | "librarypath" | "modulepath"
                | "includepath" | "subsystem" | "target" | "optimize" | "addverinfo"
                | "vimajor" | "viminor" | "vipatch" | "viproductname" | "videscription"
                | "vifilename" | "vicompanyname" | "vicopyright" | "breakpoint"
                | "message" | "unitTestMode" ;
```

- Conditional compilation: `@define` `@undef` `@ifdef` `@ifndef` `@elseif`
  `@else` `@endif`.
- Build config and version info directives take a following string, integer,
  or identifier value and are terminated by `;`.

#### Known Directive Values

| Directive | Value type | Valid values |
|-----------|-----------|--------------|
| `@target` | identifier | `win64`, `linux64` |
| `@optimize` | identifier | `debug`, `releasesafe`, `releasefast` (alias: `release`), `releasesmall` |
| `@subsystem` | identifier | `console`, `gui` |
| `@addverinfo` | identifier | any non-empty identifier enables |
| `@unittestmode` | identifier | `on`, `off` |
| `@message` | severity + string | `hint`, `info`, `note`, `warn`, `error`, `fatal` |
| `@breakpoint` | (none) | (no value) |
| `@vimajor` / `@viminor` / `@vipatch` | integer | version numbers |
| `@viproductname` / `@videscription` / `@vifilename` / `@vicompanyname` / `@vicopyright` | string | version info strings |
| `@exeicon` / `@copydll` / `@linklibrary` / `@librarypath` / `@modulepath` / `@includepath` | string | file/directory paths |

#### Predefined Conditional Symbols

| Symbol | Defined when |
|--------|-------------|
| `NITROLISP` | Always |
| `TARGET_WIN64`, `WIN64`, `MSWINDOWS`, `WINDOWS` | Target is `win64` |
| `TARGET_LINUX64`, `LINUX`, `POSIX`, `UNIX` | Target is `linux64` |
| `CPUX64` | Always (x64-only target) |

#### Numeric Literal Type Rules

| Literal | Suffix | Type |
|---------|--------|------|
| `42` | (none) | `int32` |
| `42u` | `u` | unsigned (context-resolved) |
| `0xFF` | (none) | `int32` (hex) |
| `1.5` | (none) | `float64` (contextual) |
| `1.5f` | `f` / `F` | `float32` |

<a id="tools"></a>

## 🛠️ Tools

NitroLISP ships as one executable, `NLC`, that carries the whole toolchain: the compiler, the C++ build pipeline, a debugger, and a Language Server. There is nothing else to download.

### 🧰 The Compiler CLI

`NLC` compiles a `.nls` source file. The full flag set:

| Flag | Long form | Meaning |
|------|-----------|---------|
| `-s` | `--source <file>` | **Required.** The `.nls` source file. |
| `-o` | `--output <path>` | Output path. Defaults to `output`. |
| `-r` | `--autorun` | Build, then run the compiled binary. |
| `-d` | `--debug` | Build, then debug the compiled binary. |
| `-h` | `--help` | Show usage. |

```
> NLC -s hello.nls -r
```

`-r` and `-d` are mutually exclusive. With no arguments, `NLC` prints its help.

### 🏗️ The C++ Build Pipeline

After NitroLISP emits C++23, it builds that C++ with a bundled zig toolchain, no external compiler or linker required. The build is driven entirely by source [directives](#getting-started), so a file always builds the same way.

| Capability | Controlled by | Values |
|------------|---------------|--------|
| Output kind | module form | `exe`, `lib`, `dll` |
| Target platform | `@target` | `win64`, `linux64` |
| Optimization | `@optimize` | `debug`, `releasesafe`, `releasefast`, `releasesmall` |
| Subsystem | `@subsystem` | `console`, `gui` |
| Version info | `@vimajor`, `@viproductname`, ... | embedded version resource |
| Executable icon | `@exeicon` | icon resource |
| Native libraries | `@linklibrary`, `@librarypath` | link against C/C++ libraries |
| Headers | `@includepath` | additional C++ include paths |
| Preprocessor defines | `@define`, `@undef` | C++ `-D` defines |

> [!NOTE]
> 🧱 Because the target can be `win64` or `linux64`, NitroLISP cross-compiles: the zig toolchain produces a Linux binary from Windows. A cross-built binary is built but not auto-run on the host.

### 🐞 The Debugger

`NLC -s file.nls -d` builds the program and drops into a debugging session. NitroLISP also exposes a Debug Adapter Protocol (DAP) server, so DAP-aware editors can drive the same debugger with a familiar UI.

The debugger supports:

- Breakpoints and exception breakpoints
- Step over, step in, and step out
- Continue and pause
- Call stack inspection (stack trace)
- Scopes and variable inspection
- Expression evaluation in the current frame

> [!IMPORTANT]
> 🪟 Source-level debugging targets `win64`. Build for the host platform when you intend to step through code.

### 🧠 The Language Server (LSP)

NitroLISP includes a Language Server so any LSP-capable editor gets first-class support while editing `.nls` files. It implements:

| Feature | LSP request |
|---------|-------------|
| Live diagnostics | `publishDiagnostics` |
| Document sync | `didOpen` / `didChange` / `didClose` |
| Completion | `completion` |
| Hover | `hover` |
| Go to definition | `definition` |
| Find references | `references` |
| Document symbols | `documentSymbol` |
| Signature help | `signatureHelp` |
| Folding ranges | `foldingRange` |
| Semantic tokens | `semanticTokens/full` |
| Inlay hints | `inlayHint` |
| Rename | `rename` |
| Code actions | `codeAction` |
| Formatting | `formatting` |

> [!TIP]
> 💡 Diagnostics carry severity (error, warning, info, hint), a code, and a source range, so editors can squiggle the exact span and surface NitroLISP's structured error codes inline.

<a id="api-reference"></a>

## 🔌 Embedding and Scripting

> [!NOTE]
> 🚧 The detailed C-ABI reference is under construction. This section explains the embedding and scripting model so you can choose the right approach today; exact function signatures will be documented in a later revision.

NitroLISP compiles ahead-of-time to C++23, which a bundled zig/clang toolchain builds into a native artifact. There are three ways to put it to work, from shipping a native component to scripting a host application at runtime.

### 🧩 1. AOT Component (lib / dll)

Compile NitroLISP to a `lib` and link it at the host's build time, or compile to a `dll` and load it at runtime, calling its exported functions across a C ABI. This is what `module lib`, `module dll`, `export`, and `defn clink ... external` are for. The host gets a fast, native component built from NitroLISP source, a clean plugin / FFI boundary.

### 📜 2. An Interpreter Written in NitroLISP

For runtime scripting, write a reader, an evaluator, and an environment as a NitroLISP program, and compile it once to a small `lib` or `dll`. The host embeds that artifact and, at runtime, hands it script text and calls its `eval` entry point. The interpreter parses and evaluates the script in-process, so scripts are hot-reloadable and sandboxable, and the host registers its own functions as primitives so scripts can drive the application.

The scripted language is whatever that interpreter implements, naturally a Lisp given NitroLISP's S-expression DNA. This is how you script a game at runtime while the engine itself is compiled NitroLISP: both share the same S-expression surface, one AOT-compiled, one interpreted.

> [!TIP]
> 💡 You can build your entire game in NitroLISP and still expose a runtime-scriptable Lisp to players or designers, by shipping an interpreter written in NitroLISP alongside the compiled engine.

### 🎮 3. NitroLISP as the Primary Language

Build the whole application in NitroLISP, a full systems language with first-class C++ interop, and compile it to an executable. NitroLISP is the language you ship, top to bottom.

<a id="how-to-guide"></a>

## 🧪 How-To Guide

Practical recipes built from NitroLISP's own test programs. Each compiles and runs as shown.

### 🗺️ Recipe Map

| Need | Recipe |
|------|--------|
| 🖨️ Output text | [Hello World](#-hello-world) |
| 📦 Store values | [Variables and Constants](#-variables-and-constants) |
| 🚦 Branch or loop | [Control Flow](#-control-flow) |
| 🔧 Reuse logic | [Functions](#-functions) |
| 🔁 Accept variable args | [Variadics](#-variadics) |
| λ Use anonymous functions | [Lambdas and Closures](#-lambdas-and-closures) |
| 📋 Model data | [Records](#-records), [Arrays](#-arrays), [Choices](#-choices), [Sets](#-sets), [Overlays](#-overlays), [Objects](#-objects) |
| 📍 Work close to memory | [Pointers](#-pointers), [Memory Management](#-memory-management) |
| 🌉 Call C/C++ | [C++ Interop](#-c-interop) |
| 📥 Import a library | [Modules and Imports](#-modules-and-imports) |
| 🛡️ Recover from failures | [Exceptions](#-exceptions) |
| 🪄 Transform code | [Macros](#-macros) |
| ➡️ Thread values | [Threading Macros](#-threading-macros) |
| 🔀 Conditional compilation | [Conditional Compilation](#-conditional-compilation) |
| 🧪 Verify behavior | [Unit Testing](#-unit-testing) |

> [!TIP]
> 💡 Run any program straight from source with `NLC -s file.nls -r`. Add `-d` instead to build and step through it in the debugger.

### 👋 Hello World

```lisp
#include <cstdio>;

(module exe HelloWorld
  (println "Hello, World!")
  (printf "C++ passthrough: 2 + 3 = %d\n" (+ 2 3)))
```

### 📦 Variables and Constants

```lisp
(module exe Vars
  (const MAX : int32 100)
  (define count : int32 0)
  (define name : string "NitroLISP")
  (set! count (+ count 1))
  (println "name={} count={} max={}" name count MAX))
```

### 🚦 Control Flow

#### If/Else

```lisp
(module exe FlowIf
  (define x : int32 42)
  (if (> x 0)
    (println "positive")
    (println "non-positive")))
```

#### While Loop

```lisp
(module exe FlowWhile
  (define i : int32 0)
  (while (< i 5)
    (println "i = {}" i)
    (set! i (+ i 1))))
```

#### For Loop

```lisp
(module exe FlowFor
  (for (i 0 to 4)
    (println "up: {}" i))
  (for (i 4 downto 0)
    (println "down: {}" i)))
```

#### Repeat/Until

```lisp
(module exe FlowRepeat
  (define n : int32 1)
  (repeat (until (> n 100))
    (println "{}" n)
    (set! n (* n 2))))
```

#### Match

```lisp
(module exe FlowMatch
  (define day : int32 3)
  (match day
    ((1)         (println "Monday"))
    ((2)         (println "Tuesday"))
    ((3)         (println "Wednesday"))
    (((range 4 5)) (println "Thu/Fri"))
    (((range 6 7)) (println "Weekend"))
    (else        (println "Unknown"))))
```

### 🔧 Functions

```lisp
(module exe Routines
  (defn add ((a : int32) (b : int32)) : int32
    (return (+ a b)))

  (defn factorial ((n : int32)) : int32
    (define result : int32 1)
    (for (i 2 to n)
      (set! result (* result i)))
    (return result))

  (println "3 + 4 = {}" (add 3 4))
  (println "5! = {}" (factorial 5)))
```

### 🔁 Variadics

```lisp
(module exe Variadics
  (defn sumInts (...) : int32
    (define total : int32 0)
    (define i : int32 0)
    (for (i 0 to (- (varargs count) 1))
      (set! total (+ total (varargs next int32))))
    (return total))
  (println "{}" (sumInts 10 20 30)))   // 60
```

### λ Lambdas and Closures

```lisp
(module exe Lambdas
  (defn apply1 ((f : (fn (int32) : int32)) (x : int32)) : int32
    (return (f x)))

  // lambda bound to a variable
  (define square : (fn (int32) : int32)
    (lambda ((n : int32)) : int32 (* n n)))

  (println "square(5) = {}" (apply1 square 5))

  // closure that captures a local
  (define adder5 : (fn (int32) : int32)
    (let ((k : int32 5))
      (lambda ((n : int32)) : int32 (+ n k))))

  (println "adder5(10) = {}" (adder5 10)))   // 15
```

### 📋 Records

```lisp
(module exe Records
  (type Customer (record
    (name    : string)
    (email   : string)
    (balance : float64)
    (active  : boolean)))

  (define c : Customer)
  (set! c.name "Ada Lovelace")
  (set! c.email "ada@example.com")
  (set! c.balance 250.0)
  (set! c.active true)
  (println "name    = {}" c.name)
  (println "balance = {}" c.balance))
```

#### Record Inheritance

```lisp
(module exe RecInherit
  (type Shape (record (x : int32) (y : int32)))
  (type Circle (record (parent Shape) (radius : float32)))

  (define c : Circle)
  (set! c.x 100)
  (set! c.y 200)
  (set! c.radius 50.0)
  (println "Circle at ({}, {}) radius {}" c.x c.y c.radius))
```

#### Packed Records and Bitfields

```lisp
(module exe Packed
  (type Flags (record packed
    (visible  : uint8 : 1)
    (enabled  : uint8 : 1)
    (priority : uint8 : 3)
    (reserved : uint8 : 3)))

  (println "Flags size: {}" (size Flags)))   // 1 byte
```

### 📚 Arrays

```lisp
(module exe Arrays
  (type Grid (array (range 0 4) int32))
  (define nums : Grid)
  (for (i 0 to 4)
    (set! nums[i] (* i i)))
  (for (i 0 to 4)
    (println "nums[{}] = {}" i nums[i])))
```

### 🎛️ Choices

```lisp
(module exe Choices
  (type Color (choices (Red 0) (Green 1) (Blue 2)))
  (define c : Color)
  (set! c Color.Green)
  (println "color value: {}" c))
```

### 🧮 Sets

```lisp
(module exe Sets
  (define s1 : set [1 3 (range 5 9)])
  (if (in 7 s1) (println "7 in set") (println "7 NOT in set"))
  (if (in 4 s1) (println "4 in set") (println "4 NOT in set")))
```

### 🧊 Overlays

```lisp
(module exe Overlays
  (type Value (overlay
    (as_int   : int32)
    (as_float : float32)))

  (define v : Value)
  (set! v.as_int 42)
  (println "as int: {}" v.as_int)
  (println "size: {} bytes" (size Value)))
```

### 🏛️ Objects

```lisp
(module exe Objects
  (type TCounter (object
    (value : int32)
    (method increment () : void
      (set! self.value (+ self.value 1)))
    (method get_value () : int32
      (return self.value))))

  (define c : (pointer to TCounter))
  (create c)
  (set! c.value 0)
  (c.increment)
  (c.increment)
  (c.increment)
  (println "count: {}" (c.get_value))   // 3
  (destroy c))
```

#### Object Inheritance

```lisp
(module exe ObjInherit
  (type TBase (object
    (x : int32)
    (method describe () : int32
      (return (* self.x 10)))))

  (type TDerived (object (parent TBase)
    (y : int32)
    (method describe () : int32
      (return (+ (parent.describe) self.y)))))

  (define d : (pointer to TDerived))
  (create d)
  (set! d.x 7)
  (set! d.y 3)
  (println "describe: {}" (d.describe))   // 73
  (destroy d))
```

### 📍 Pointers

```lisp
(module exe Pointers
  (define x : int32 42)
  (define p : pointer (address x))
  (println "value: {}" p^)    // 42
  (set! p^ 100)
  (println "x is now: {}" x)) // 100
```

### 🧠 Memory Management

```lisp
(module exe Memory
  (type TData (record (value : int32)))
  (define p : (pointer to TData))
  (create p)
  (set! p^.value 99)
  (println "value: {}" p^.value)
  (destroy p))
```

### 🌉 C++ Interop

```lisp
#include <cstdio>;

(module exe InteropDemo
  (printf "factorial says hi: %d\n" (* 6 7)))
```

### 📥 Modules and Imports

A library is a `lib` module whose exported declarations are visible to importers. Its declared name must match its filename.

```lisp
// testlib.nls
(module lib testlib
  (export (defn lib_triple ((n : int32)) : int32
    (return (* n 3)))))
```

```lisp
// main.nls
(module exe ImportTest
  (import testlib)
  (printf "lib_triple(7) = %d\n" (testlib.lib_triple 7)))
```

### 🛡️ Exceptions

```lisp
(module exe Exceptions
  // guard/except with raise
  (guard
    (raise "plain boom")
    (except
      (println "caught: code={} msg={}" (exccode) (excmsg))))

  // raisecode with explicit code
  (guard
    (raisecode 42 "coded boom")
    (except
      (println "caught: code={} msg={}" (exccode) (excmsg))))

  // guard/finally (cleanup runs always)
  (guard
    (println "body runs")
    (finally
      (println "cleanup runs always"))))
```

### 🪄 Macros

```lisp
(module exe MacroDemo
  (defmacro square (x)
    (quasiquote (* (unquote x) (unquote x))))

  (println "{}" (square 5))          // 25
  (println "{}" (square (+ 2 1))))   // (* (+ 2 1) (+ 2 1))
```

### ➡️ Threading Macros

```lisp
(module exe ThreadDemo
  (defn sub ((a : int32) (b : int32)) : int32
    (return (- a b)))

  (println "{}" (thread_first 100 (sub 10) (sub 30)))  // 60
  (println "{}" (thread_last  100 (sub 10) (sub 30)))) // 120
```

### 🔀 Conditional Compilation

```lisp
@define VERBOSE;

(module exe Conditional
  @ifdef VERBOSE
    (println "verbose mode is on")
  @endif

  @ifdef WIN64
    (println "running on 64-bit Windows")
  @endif)
```

### 🧪 Unit Testing

```lisp
@unittestmode on;

(module exe Tests
  (defn add ((a : int32) (b : int32)) : int32
    (return (+ a b)))

  (defn is_even ((n : int32)) : boolean
    (return (= (mod n 2) 0)))

  (test "addition"
    (testAssertEqualInt 5 (add 2 3))
    (testAssertEqualInt 0 (add -1 1)))

  (test "even check"
    (testAssertTrue (is_even 0))
    (testAssertTrue (is_even 42))
    (testAssertFalse (is_even 7)))

  (test "comparisons"
    (testAssertTrue (> 10 5))
    (testAssertTrue (<= 5 5))
    (testAssertTrue (= 42 42))
    (testAssertTrue (<> 42 99))))
```

### ⚙️ Build Modes

Build settings are directives at the top of the file:

```lisp
@target win64;
@optimize debug;
@subsystem console;

(module exe Fib
  (defn fib ((n : int32)) : int32
    (return (if (< n 2)
                n
                (+ (fib (- n 1)) (fib (- n 2))))))
  (println "fib(30) = {}" (fib 30)))
```

Switch `@target` to `linux64` to cross-compile. Use `releasesafe`, `releasefast`, or `releasesmall` for optimized builds.

<a id="contributing"></a>

## 🤝 Contributing

NitroLISP is developed by tinyBigGAMES. Whether you are fixing a bug, improving documentation, sharpening examples, or proposing a feature, contributions are welcome.

| Contribution | Best Way to Help |
|--------------|------------------|
| 🐞 Bug report | Open an issue with a minimal reproduction and the exact command used |
| 💡 Feature idea | Describe the real use case first, then the proposed syntax or behavior |
| 🧾 Documentation fix | Point to the section and explain what was unclear or missing |
| 🧪 Test case | Include the smallest `.nls` file that proves the behavior |
| 🔧 Pull request | Keep the change focused and explain the before/after behavior |

> [!TIP]
> 🚀 Small, focused contributions are the easiest to review and the fastest to land.

## 💖 Support the Project

If NitroLISP saves you time, helps you learn, or sparks something useful:

- ⭐ **Star the repo**: it costs nothing and helps others find the project
- 🗣️ **Spread the word**: write a post, mention it in a community, or share a screenshot
- 💬 **Join the community**: show what you are building and help shape what comes next
- 🧪 **Try examples**: real usage finds issues that synthetic tests miss
- 💖 **[Become a sponsor](https://github.com/sponsors/tinyBigGAMES)**: sponsorship directly funds development, examples, and documentation

## 📜 License

NitroLISP is licensed under the **Apache License, Version 2.0**. See [LICENSE](https://github.com/tinyBigGAMES/NitroLISP?tab=License-1-ov-file#) for details.

Apache 2.0 is a permissive open source license that lets you use, modify, and distribute NitroLISP freely in both open source and commercial projects. You are not required to release your own source code. Attribution is required: keep the copyright notice and license file in place.

## 🔗 Links

- 🌐 [nitrolisp.com](https://nitrolisp.com)
- 🧑‍💻 [GitHub](https://github.com/tinyBigGAMES/NitroLISP)
- 💬 [Discord](https://discord.gg/Wb6z8Wam7p)
- 🦋 [Bluesky](https://bsky.app/profile/tinybiggames.com)
- 🎮 [tinyBigGAMES](https://tinybiggames.com)

<div align="center">

**🚀 NitroLISP&trade;** - Lisp at the speed of thought.

Copyright &copy; 2026-present tinyBigGAMES&trade; LLC<br/>All Rights Reserved.

</div>