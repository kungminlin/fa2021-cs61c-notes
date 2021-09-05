# Brian Harvey's Intro to C
## Table of Contents
  - [61C from a 61A Perspective](#61c-from-a-61a-perspective)
    - [Abstraction](#abstraction)
    - [The Stored Program Computer](#the-stored-program-computer)
  - [C as a Low-Level Language](#c-as-a-low-level-language)
  - [C Pitfalls](#c-pitfalls)
    - [15 Levels of Precedence](#15-levels-of-precedence)
    - [Type Casting](#type-casting)
    - [`switch` and `break`](#switch-and-break)
  - [Storage Classes](#storage-classes)
    - [`extern`](#extern)
    - [`static`](#static)
  - [The `main()` Procedure](#the-main-procedure)
  - [Arrays and Pointers](#arrays-and-pointers)
    - [Memory Addresses](#memory-addresses)
    - [Endianness](#endianness)
    - [C Pointer Types](#c-pointer-types)
    - [Arrays](#arrays)
## 61C from a 61A Perspective

### Abstraction
**Maximum "fanout"**: The number of circuits that can be allowed to read a given circuit's output before the voltage is interpreted as low voltage (1 bit appears to be 0 bit instead)

**Number Representation**
- Current computers use two's complement library, but earlier computers used BCD (Binary Coded Decimal)
    - In BCD, each decimal digit is hard-coded with 4-bits
    - e.g. `2479` = `0010 0100 0111 1001`
    - BCD was this way because there were *business* computers with direct encoding of format (e.g. `$******2,479.10`)

### The Stored Program Computer
- **Metacircular evaluator** (*universal* program): A single program used to turn input into a particular computation desired
- Instead of representing a problem in the circuitry, you represent a problem as *data* in the computer's memory
  - The circuitry is an algorithm by which the representation is interpreted
  - The modern CPU is exactly analogous to the metacircular evaluator

## C as a Low-Level Language
**Approximating powers of two**

$
2^{10} = 1024 \approx 1000 \\
2^{15} = 2^5 \cdot 2^{10} \approx 32 \ \textup{thousand} \\
2^{32} = 2^2 \cdot 2^{10} \cdot 2^{10} \approx 4 \ \textup{billion}
$
- K (kilo), M (mega) in the computer industry is sometimes 1024 and 1024 squared

**Integers**
- Today signed integer range from plus to minus 2 billion
- In C, `int` is represented in a form reflecting the natural size of the integer on the host machine, so 16-bit integers and 32-bit integers are not the same
  - Not portable

## C Pitfalls
### 15 Levels of Precedence
- We use a two-level system for infix operator precedence: Multiplication and division before addition and subtraction
- C uses a chart with 15 levels of precedence for its 45 operators
- "just use parentheses" to avoid confusion of precedence
- `*p++` both dereferences and increments a pointer
  - Operators at this level associate right to left, so it is `(* (p++))`
    - The incrementing happens *before* the dereferencing, but the result of the increment is its prior value
    - `*p++` and `*--p` are very common because they compile into a single instruction on the PDP-11 computer

### Type Casting
```c
int x, y;
int *p;
...
y = *p;             /* legal */
y = *x;             /* illegal */
y = *((int *)x);    /* legal!! */
```
- The third statement: "Pretend that x does contian a pointer to an integer, dereference that pointer, and set y to the value found at that location in memory."
  - This will likely result in runtime error
- C allows this because one functionality it was designed for was to communicate with *registers* that contain I/O device info
  - Registers *look* like memory — each device has its own register addresses. C has to enable processor reads and writes to these registers just like actual memory.

### `switch` and `break`
- Using `switch` without `break` will cause C to evaluate all `case` until it hits `default`

## Storage Classes
- Syntactically similar to Java `public` modifier
- `extern`, `static`, `auto`, `register`

### `extern`
- Similar to "global", but not entirely the same

***Declaration* vs. *Definition***
- A *declaration* tells the compiler the type of the variable, but does not allocate storage for it.
- A *definition* serves both purposes.
- `extern` is explicitly a declaration
  ```c
  int x;
  int x = 10;           /* Definition */
  extern int x;         /* Explicitly Declaration */
  extern int x = 0;     /* Illegal */
  ```
- `int x;` is actually a special case
  - Considered a *definition*; allocates 1-bit memory (0) for `x`

**Usage**
- `extern` is normally used in a header (`.h`) file
- Common for large programs divided into several files
- `extern` is allowed inside a procedure
  - Tells compiler that the procedure will use the declared variable from elsewhere (similar to Python `global`)

### `static`

***Scope* of Variables**
- The *scope* of a variable means where the variable is available for use
- *Lexical scope* means the variable is available anywhere inside the procedure including internally defined procedures (e.g. Scheme)
- *Dynamic scope* means that variable is available within any procedure invoked by the defining procedure
- Both C and Java use a *degenerate* form of *lexical scope*
  - No internal procedure definitions
- C has two kinds of global scope
  - "external linkage": Available throughout the program
  - "internal linkage": Available only in the source file

***Extent* of Variables**
- The *extent* of a variable means how long it exists
- Most global variables have infinite extent (entire duration of the program)

|               | Inside Procedure              | Global |
| ------------- | ----------------------------- | ------- |
| `static`      | Local scope, infinite extent  | Same-file scope, infinite extent |
| not `static`  | Local scope, local extent     | Program scope, infinite extent |

## The `main()` Procedure
```c
int main(int argc, char **argv) { ... }
```
- There's an optional third argument which allows access to shell variables set with the `setenv` shell command
- Example: `foo hello 87`
  ```c
  argv[0] = "foo"
  argv[1] = "hello"
  argv[2] = "87"
  ```
- The shell turns `*` into a list of all files in the current directory
    - e.g. `rm *`

## Arrays and Pointers

### Memory Addresses
- Always use `double`, not `float`
  - 32-bit floats don't have enough precision
  - Any `int` can be represented by `double`, but not with `float`
- The *de facto* standard is that the smallest addressable unit of memory is 8-bit (byte)
  - C uses `int` instead to represent boolean because booleans are 1-bit and C does not support that
  - It is not possible to dereference a pointer with 1-bit
- On most current architectures (including MIPS), `int` must have an address that's divisible by 4
  - The reason why most computer memories are distinct by a factor of 4

### Endianness
- *Big-endian*: Machines that start counting addresses from the left.
- *Little-endian*: Machines that start counting addresses from the right.

### C Pointer Types
- `*p`: Dereference operator
  - "Get me the contents of the memory location whose address is in variable `p`"
  - "Store something into the memory location whose address is in variable `p`"
- There is no pointer type for `int` type (no `*int`)
  - A variable can't just be a general pointer; has to be pointer to a particular data type
  - This also explains why `char **argv` is same as `char *argv`
    - `char **argv` means that `argv` is pointer to data of type `char *`, a pointer to a pointer to a character

**`void` Pointers**
```c
void *p;        /* "I'll tell you what kind of thing it points to" */
*p              /* illegal */
*((int *)p)     /* legal */
```

### Arrays
Example: An array of 10 integers starting at address 1000:
```c
1000:   a[0]
1004:   a[1]
1008:   a[2]
    ...
1036:   a[9]
```
- To do something to an element: `int *p = &a[0];`
- To get the next element: `p++` or `p+1`
  - It's not `p+4` because C takes the size of a pointer's target into account
  - This only works on `*argv` and not `**argv`, because a pointer (`*argv`) is 4 bytes wide, whereas `char` (`argv` itself) is 1 byte.