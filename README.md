# brainfxxck

![](https://s2.radikal.cloud/2025/11/30/iPhone-16672b0b7bbcfbb076.png)

**Modern C++ brainfuck interpreter/compiler with LLVM 18**

brainfxxck is a high-performance brainfuck interpreter and compiler that leverages LLVM 18's JIT compilation capabilities to achieve significantly faster execution speeds compared to traditional interpreters.

## Features

- **JIT Compilation**: Default execution mode using LLVM's ORC JIT engine for fast runtime performance
- **Static Compilation**: Compile brainfuck programs to native executables
- **AST Optimizations**: Automatic optimization of brainfuck code at the Abstract Syntax Tree level:
  - Combines consecutive `+`/`-` operations into single instructions
  - Combines consecutive `>`/`<` operations into single instructions
  - Optimizes simple clear loops (`[-]` or `[+]`) into direct cell assignments
- **Multiple Output Formats**: Generate LLVM IR, assembly, bitcode, or object files
- **Configurable Optimization**: Choose optimization levels from 0-3 (default: 2)
- **Standard Brainfuck**: Full support for the standard brainfuck language (8 instructions)

## Performance

brainfxxck achieves significant performance improvements over traditional interpreters through LLVM JIT compilation and optimizations.

### Benchmark Results

Performance comparison on `mandelbrot-huge.bf`:

| Implementation | Real Time |
|----------------|-----------|
| **brainfxxck** | **0m28.230s** |
| brainfuck      | 1m24.638s |

**Result**: brainfxxck is approximately **3x faster** than the traditional brainfuck interpreter.

## Requirements

- **LLVM 18** or later (required)
- **CMake 3.20** or later
- **C++17** compatible compiler (GCC 7+, Clang 5+)
- Build tools (make, ninja, etc.)

## Building

### Step 1: Clone and Navigate

```bash
cd brainfxxck
mkdir build
cd build
```

### Step 2: Configure with CMake

```bash
cmake ..
```

CMake will automatically:
- Detect LLVM 18 installation
- Fetch PEGTL (Parsing Expression Grammar Template Library) if not found
- Configure the build system

### Step 3: Build

```bash
make
```

Or with ninja (faster):
```bash
cmake -GNinja ..
ninja
```

### Step 4: Install (Optional)

```bash
make install
```

This installs:
- `brainfxxck` executable to `bin/`
- `brainfxxck_lib` library to `lib/`
- Header files to `include/`

## Usage

### Basic JIT Execution (Default)

Run a brainfuck program with JIT compilation:

```bash
./brainfxxck program.bf
```

### Compile to Native Binary

Compile a brainfuck program to a standalone executable:

```bash
./brainfxxck --compile -o program program.bf
./program
```

### Optimization Levels

Control optimization level (0-3):

```bash
# No optimization
./brainfxxck -O0 program.bf

# Default optimization (O2)
./brainfxxck -O2 program.bf

# Maximum optimization
./brainfxxck -O3 program.bf
```

### Output Formats

Generate different intermediate representations:

```bash
# LLVM IR
./brainfxxck --emit-llvm -o program.ll program.bf

# Assembly
./brainfxxck --emit-asm -o program.s program.bf

# LLVM Bitcode
./brainfxxck --emit-bc -o program.bc program.bf

# Object file
./brainfxxck --compile -o program.o program.bf
```

### Disable AST Optimizations

Run without AST-level optimizations:

```bash
./brainfxxck --no-optimize program.bf
```

### Command-Line Options

```
Usage: brainfxxck [OPTIONS] <file>

Options:
  --jit              Run using JIT compilation (default)
  --compile          Compile to native binary
  -o <output>        Output file for compilation
  -O<level>          Optimization level (0-3, default: 2)
  --emit-llvm        Emit LLVM IR instead of binary
  --emit-asm         Emit assembly instead of binary
  --emit-bc          Emit LLVM bitcode instead of binary
  --no-optimize      Disable AST optimizations
  --help             Show this help message
```

## Architecture

brainfxxck consists of several key components:

### Parser ([`src/parser.cpp`](src/parser.cpp))

Uses [PEGTL](https://github.com/taocpp/PEGTL) (Parsing Expression Grammar Template Library) to parse brainfuck source code into an Abstract Syntax Tree. The parser:
- Handles all 8 brainfuck instructions (`+`, `-`, `>`, `<`, `.`, `,`, `[`, `]`)
- Validates bracket matching
- Ignores non-brainfuck characters

### AST ([`src/ast.cpp`](src/ast.cpp))

The Abstract Syntax Tree representation includes:
- **Add**: Increment/decrement operations
- **Move**: Pointer movement operations
- **Input/Output**: I/O operations
- **Loop**: Loop constructs with nested instructions
- **Set**: Direct cell assignment (from optimizations)

The AST optimizer performs:
- Combining consecutive `Add` operations
- Combining consecutive `Move` operations
- Converting simple clear loops to `Set` instructions

### Code Generator ([`src/codegen.cpp`](src/codegen.cpp))

Generates LLVM IR from the optimized AST:
- Creates a main function that allocates and initializes the tape
- Generates optimized LLVM IR for each instruction type
- Handles tape wrapping for pointer operations
- Links with standard C library functions (`putchar`, `getchar`, `malloc`, `free`)

### JIT Engine ([`src/jit.cpp`](src/jit.cpp))

Uses LLVM's ORC JIT framework to:
- Compile LLVM IR to machine code at runtime
- Apply LLVM optimization passes
- Execute the compiled code directly
- Manage memory and symbol resolution

### Compiler ([`src/compiler.cpp`](src/compiler.cpp))

For static compilation:
- Generates object files, assembly, or LLVM IR
- Applies target-specific optimizations
- Links with system linker to create executables

## Examples

The `examples/` directory contains a comprehensive collection of brainfuck programs:

- **Hello World**: `examples/hello.bf`
- **Mathematical Programs**: `examples/math/` (Fibonacci, prime numbers, π calculation, etc.)
- **Mandelbrot Set**: `examples/mandelbrot/` (including `mandelbrot-huge.bf` used in benchmarks)
- **Sorting Algorithms**: `examples/sort/`
- **Games**: `examples/gameoflife.bf`, `examples/lost-kingdom.bf`
- **Quines**: `examples/quine/` (self-replicating programs)
- **Compilers/Interpreters**: `examples/compiler/`, `examples/interpreter/`

### Running Examples

```bash
# Hello World
./brainfxxck examples/hello.bf

# Mandelbrot set (benchmark program)
./brainfxxck examples/mandelbrot/mandelbrot-huge.bf

# Compile and run
./brainfxxck --compile -o hello examples/hello.bf
./hello
```

## Project Structure

```
brainfxxck/
├── CMakeLists.txt          # Build configuration
├── LICENSE                  # MIT License
├── README.md               # This file
├── include/
│   └── brainfxxck/
│       ├── ast.hpp         # AST node definitions
│       ├── codegen.hpp     # LLVM IR code generator
│       ├── compiler.hpp    # Static compiler interface
│       ├── jit.hpp         # JIT engine interface
│       └── parser.hpp      # Parser interface
├── src/
│   ├── main.cpp            # Command-line interface
│   ├── parser.cpp          # PEGTL-based parser
│   ├── ast.cpp             # AST implementation and optimizer
│   ├── codegen.cpp         # LLVM IR generation
│   ├── jit.cpp             # JIT execution engine
│   └── compiler.cpp        # Static compilation
└── examples/               # Brainfuck example programs
    ├── hello.bf
    ├── mandelbrot/
    ├── math/
    └── ...
```

## License

See [LICENSE](LICENSE) for the full license text.
