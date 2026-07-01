<div align="center">

![NitroLISP](media/logo.jpg)

[![Discord](https://img.shields.io/discord/1457450179254026250?style=for-the-badge&logo=discord&label=Discord)](https://discord.gg/Wb6z8Wam7p) [![Follow on Bluesky](https://img.shields.io/badge/Bluesky-tinyBigGAMES-blue?style=for-the-badge&logo=bluesky)](https://bsky.app/profile/tinybiggames.com)

</div>

## What is NitroLISP?

**A statically typed Lisp that compiles to modern C++23.** Write homoiconic S-expression code and get a native binary. NitroLISP carries its own language definition, standard library, and a bundled zig/clang toolchain, so there is no separate C++ compiler, linker, or runtime to install.

```lisp
// hello.nls
#include <cstdio>;

(module exe HelloWorld
  (println "Hello, World!")
  (printf "C++ passthrough: 2 + 3 = %d\n" (+ 2 3)))
```

Compile and run:

```bash
NLC -s hello.nls -r
```

NitroLISP compiles ahead-of-time through a complete pipeline -- tokenize, parse, macro expansion, semantic analysis, tail-call optimization, and C++ 23 emission -- after which a bundled zig/clang toolchain builds the emitted C++. The entire toolchain runs from a single executable. Ship a standalone program, embed NitroLISP as a native component, or script a host application at runtime with an interpreter written in NitroLISP.

Its defining idea is a **tiny compiler-known core plus macros that grow the language.** The compiler knows a small set of forms, lambdas and closures, the data and control forms, modules, and a macro engine. Everything else, `when`, `unless`, threading, and anything you add, is a macro in the auto-loading prelude, written in NitroLISP itself.

Because the target is C++ 23, **interop is first-class.** Drop in a raw `#include` and call C/C++ functions directly as S-expression forms. The surface is Lisp; the standard library and existing C/C++ code come along with no binding layer.

## 🎯 Who is NitroLISP For?

- **Game and application developers**: build the whole app in NitroLISP, or embed it in a host -- as a native `lib` / `dll` component, or as a runtime-scriptable interpreter. First-class C++ interop pairs naturally with engines and existing C/C++ libraries.
- **C++ developers**: a homoiconic, macro-driven front end that emits readable C++23 you can inspect, build, and integrate.
- **Lisp and language enthusiasts**: a small, comprehensible core and a real macro engine, where the interesting design lives in the prelude rather than buried in a compiler.

## ✨ Key Features

- **Tiny core, macros grow it**: a small set of compiler-known forms plus a real macro engine. `when`, `unless`, threading, and more are NitroLISP code in the prelude, not compiler features.
- **Homoiconic S-expressions**: code is data. Macros transform the program with the same syntax you write it in.
- **Compiles to C++23**: emits readable, modern C++23 and builds it with a bundled zig toolchain into a native binary.
- **Static typing**: every value has a compile-time type. Full-width numeric types (`int8`..`int64`, `uint8`..`uint64`, `float32`/`float64`) for clean C interop.
- **First-class functions**: `lambda`, closures with compiler-driven capture, and an `fn` function type backed by `std::function`.
- **Modules**: `exe`, `dll`, and `lib` modules with `import`, exported declarations, and scope-resolved cross-module calls (`module.fn`). One import carries a library's functions and macros together.
- **First-class C++ interop**: raw `#include` passthrough and calling C/C++ as S-expression forms, by design rather than as an escape hatch.
- **Tail-call optimization**: self tail-calls are rewritten to loops, so recursion-as-iteration runs flat.
- **Structured exceptions**: `guard` / `except` / `finally` with `raiseexception` and exception context intrinsics.
- **Variadics, intrinsics, memory**: `varargs`, `len` / `size` / `utf8` / `paramcount` / `paramstr`, and explicit memory management.
- **Built-in debugger**: Debug Adapter Protocol (DAP) support with breakpoints, stepping, call stacks, scopes, variable inspection, and expression evaluation.
- **Language Server Protocol**: diagnostics, completion, hover, go-to-definition, references, document symbols, signature help, folding, semantic tokens, inlay hints, rename, code actions, and formatting.
- **Conditional compilation and resources**: `@define` / `@ifdef` / `@ifndef` / `@elseif` / `@else` / `@endif`, plus version info and icon directives.
- **Zero external dependencies**: the compiler and a bundled zig/clang toolchain ship in a single executable. No separate compiler, linker, or runtime to install.

## 🚀 Getting Started

Every NitroLISP program is a **module**. The module kind, `exe`, `dll`, or `lib`, is declared at the top of the file and determines the artifact. Build settings are directives, written above the module form.

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

Compile and run:

```bash
NLC -s fib.nls -r
```

Output:

```
fib(30) = 832040
```

The output kind is determined by the `module` declaration:

| Module Declaration | Output | Description |
|--------------------|--------|-------------|
| `module exe name` | `name.exe` | Native executable |
| `module dll name` | `name.dll` | Shared library |
| `module lib name` | (imported) | Reusable NitroLISP library module, imported by other modules |

## 🔌 Embedding and Scripting

NitroLISP fits a host application in three ways:

- **AOT component**: compile to a `lib` and link it, or to a `dll` and load it at runtime, calling exported functions across a C ABI (`module lib` / `module dll`, `exported`, `defn "C" ... external`). A fast native component with a clean plugin / FFI boundary.
- **Runtime scripting**: write an interpreter -- reader, evaluator, environment -- in NitroLISP and compile it once; the host hands it script text at runtime and calls its `eval` entry point. Scripts run in-process, hot-reloadable and sandboxable, with host functions registered as primitives. This is how you script a game at runtime while the engine itself is compiled NitroLISP.
- **Primary language**: build and ship the whole application in NitroLISP, a full systems language with first-class C++ interop.

See the [embedding and scripting reference](docs/NitroLISP.md#api-reference) for details.

## 📖 Documentation

The full language reference, macro system, BNF grammar, toolchain guide, and how-to recipes are in a single document:

| Document | Description |
|----------|-------------|
| **[NitroLISP Documentation](docs/NitroLISP.md)** | Complete tour: types, `define` / `defn` / `lambda` / `fn`, control flow, modules and imports, exceptions, memory, intrinsics, varargs, C++ interop, the macro system (`defmacro`, quasiquote, compile-time eval, `gensym`, the prelude, threading), the full BNF grammar, and the toolchain (compiler, DAP debugger, LSP, zig/clang build). |

## 🔨 Getting NitroLISP

### Download the Latest Release

**[Download the latest release](https://github.com/tinyBigGAMES/NitroLISP/releases/latest)**

The release package contains the compiler (`NLC.exe`), the standard library, and the bundled zig/clang toolchain. No external toolchain is required.

### CLI Reference

```bash
NLC -s <source.nls> [options]
```

| Flag | Description |
|------|-------------|
| `-s <file>` | Source file to compile (required) |
| `-o <path>` | Output path (default: `output`) |
| `-r` | Compile and run the resulting binary |
| `-d` | Compile and launch the debugger |
| `-h` | Show help |

```bash
NLC -s hello.nls              # compile
NLC -s hello.nls -o build     # compile to a specific output path
NLC -s hello.nls -r           # compile and run
NLC -s hello.nls -d           # compile and debug
```

### System Requirements

| | Requirement |
|---|---|
| **Host OS** | Windows x64 |
| **Runtime dependencies** | None |
| **External toolchain** | None (zig/clang is bundled) |

## 🔧 Building the Compiler from Source

This section is for contributors who want to modify NitroLISP itself. To just write NitroLISP programs, use the release download above.

### Prerequisites

| | Requirement |
|---|---|
| **Host OS** | Windows x64 |
| **Compiler** | Delphi 12 Athens or higher |

### Get the Source

```bash
git clone https://github.com/tinyBigGAMES/NitroLISP.git
```

### Compile

1. Open the NitroLISP project group in Delphi 12 Athens or higher
2. Build all projects in the group (Win64 target)
3. The Testbed project runs the test programs and reports results

## 🤝 Contributing

NitroLISP is an open project and contributions are welcome at every level:

- **Report bugs**: open an issue with a minimal `.nls` reproduction case.
- **Suggest features**: describe the use case first, then the syntax you have in mind.
- **Submit pull requests**: bug fixes, documentation improvements, new test cases, and well-scoped features.

Join our [Discord](https://discord.gg/Wb6z8Wam7p) to discuss development, ask questions, or share what you are building with NitroLISP.

## 💙 Support the Project

If NitroLISP saves you time, sparks an idea, or becomes part of something you ship:

- **Star the repo** -- helps others find the project
- **Spread the word** -- write a post, mention it on social media
- **Join us on [Discord](https://discord.gg/Wb6z8Wam7p)** -- share what you are building
- **Become a sponsor** via [GitHub Sponsors](https://github.com/sponsors/tinyBigGAMES) -- directly funds development

## 📄 License

NitroLISP is licensed under the **Apache License 2.0**. See [LICENSE](https://github.com/tinyBigGAMES/NitroLISP?tab=Apache-2.0-1-ov-file#License-1-ov-file) for details.

## 🔗 Links

- [Website](https://nitrolisp.com)
- [Discord](https://discord.gg/Wb6z8Wam7p)
- [Bluesky](https://bsky.app/profile/tinybiggames.com)
- [tinyBigGAMES](https://tinybiggames.com)

<div align="center">

**NitroLISP**&#8482; - Lisp at the speed of thought.

Copyright &copy; 2026-present tinyBigGAMES&#8482; LLC
All Rights Reserved.

</div>
