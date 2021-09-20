# Assembly, RISC-V Intro

## Assembly Language

### Instruction Set Architecture (ISA)
- Job of the CPU is to execute instructions
- Instructions are executed in sequence
  - Advanced processors can execute in parallel
- CPU belong to families, each with their own ISA
  - Examples of ISA: ARM, Intel x86, MIPS, RISC-V, x86_64
  - These are still *abstractions*. Could be different hardware, but same instruction set
- Early trend was to add more and more complex instruction to new CPU's: Complex Instruction Set Computer (CISC)
  - e.g. VAX architecture that could evaluate polynomials
- Reduced Instruction Set Computer (RISC)
  - Keep instruction set small and simple; easier to build fast hardware
  - Abstract away complicated operations to software

### Architectures
- Servers: x86/64
  - Software: Windows/Linux
- Phones/embedded: ARM
  - Software: Linux/Android
  - Efficient for small chips, small hardware (e.g. doorbells)
- Each assembly language is tied to a particular ISA


## RISC-V Architecture
- Comes in 32-bit, 64-bit and 128-bit
- Textbook uses 64-bit
- We use 32-bit
- Efficient concurrent operations

## Registers vs. Variables

### Registers
- Assembly does not have variables
- Registers: Assembly language operands
  - Limited number of places to hold values built directly into the hardware
  - Arithmetic operations can only be performed on registers
- Registers are directly in hardware; fast to access

![Registers](../images/2021-09-17_1.png)

- Registers are 100-500 times faster than DRAM (Dynamic Random Access Memory)
- There are 32 registers/words (4-bytes each) in RISC-V
  - 128 bytes in total
- x0 is special: always hold zero value; so there is really only 31 available registers

### Memory Alignment
- Memory operations address 32-bit words
- Word-aligned: Integers start on even 4-byte boundaries (address is even multiple of 4; last 2 bits of address is 00)
  - e.g. `0xa04` -> `0xa08` -> `0xa0c` -> ...
- RISC-V does not require integers be word-aligned, but ***in practice***, it is required

## RISC-V Instructions
- Instructions are fixed, 32 bytes long word-aligned
- Each instruction uses predefined formats

![RISC-V Instructions](../images/2021-09-17_2.png)

### Syntax
- Instructions have opcode and operands
  - e.g. `add x1, x2, x3 # x1 = x2 + x3`
  - Opcode: `add`
  - Operands: `x1, x2, x3`
- `#` is assembly comment (`# comment`)

### Addition and Subtraction (`add`, `sub`)
- Addition: `add x1, x2, x3`
  - x1 = x2 + x3
- Subtraction: `sub x3, x4, x5`
  - x3 = x4 - x5


Example: `a = b + c + d - e;`
```
add x1, x2, x3 # temp (x1) = b + c
add x1, x1, x4 # temp = temp + d
add x1, x1, x5 # a = temp - e
```

### Register x0
- Move a value from one register to another
  - `add x3, x4, x0`: Moves `x4` to `x3`
- Throw away a value

### Immediates (`addi`)
- Immediates are used to provide numerical constants
- e.g.: `addi x3, x4, -10`
  - x3 = x4 - 10
- Moving with `addi`: `addi x3, x4, 0`
- "Sign-extended"
  - Bits 31:12 gets the same value as bit 11, which indicates the sign

## Memory
- "Load" and "store" instructions are used to access data from the memory (which is slower)
- Data typically smaller than 32 bits, but rarely smaller than 8 bits
- Word size is 4 bytes
- Memory is addressable to individual bytes
- Word addresses are 4 bytes apart

### Transfer from Memory to Register
```c
int A[100];
g = h + A[3];
```
- Assume `x13` holds the base register pointing to `A[0]`, `x12` holds `h`
- 12 is offset in bytes (3 * 4)
- Use load word (`lw`) in RISC-V:
  ```
  lw    x10, 12(x13)    # register x10 gets A[3], which is 12 + base address (x13)
  add   x11, x12, x10   # g = h + A[3]
  ```

```c
int A[100];
A[10] = h + A[3];
```
- Assume `x13` holds base register, `x12` holds `h`
- A[3] is at offset 12, A[10] is at offset 40 (4 * 10)
- Using store word (`sw`) in RISC-V:
    ```
    lw  x10, 12(x13)    # temp reg x10 gets A[3]
    add x10, x12, x10   # temp reg x10 gets h + A[3]
    sw  x10, 40(x13)    # A[10] = h + A[3]
    ```

### Loading and Storing Bytes
- Load byte: `lb`
- Store byte: `sb`
- e.g. `lb x10, 3(x11)`
  - Memory location with address = 3 + `x11` is copied to the low byte position of register `x10`
- `lbu` is used to load and zero extend to fill register for unsigned byte
- There is no `sbu`

## Assembly Conventions

**Symbolic Register Names**
- a0 - a7 for argument registers (x10 - x17)
- x0 for zero

**Pseudo-instructions**
- Instructions that don't exist in machine instruction but assembler automatically parses
- `mv rd, rs` = `addi rd, rs, 0`

### Register Conventions
![Register Conventions](../images/2021-09-20_3.png)

## Functions
1. Put parameters in a place where function can access
2. Transfer control to function (jump)
3. Acquire storage needed for function (e.g. local variables)
4. Perform desired task
5. Put result value in a place where calling code can access
6. Return control to point of origin

### Calling Convention
- Caller-saved
  - The function invoked (the callee) can do whatever it wants to them
  - Caller cannot count on their contents
- Callee-saved
  - The function invoked must restore them before returning

| Register | Purpose |
| - | - |
| a0 - a7 | Parameters |
| a0 - a1 | Return values |
| ra | Return address |
| s0 - s11 | Saved registers (callee-saved) |
| fp | s0; Frame pointer: Points to the top of the call frame |
| sp | x2; Stack pointer; Points to the last used place on the stack |
| t0 - t6 | Temporaries; caller-saved |

### Example
```c
int Leaf(int g, int h, int i, int j) {
  int f;
  f = (g + h) - (i + j);
  return f;
}
```
- Parameter variables `g`, `h`, `i`, `j` in argument registers `a0`, `a1`, `a2`, `a3`

```
leaf: addi  sp, sp, -8    # adjust stack for 2 items
      sw    s1, 4(sp)     # save s1 for use afterwards
      sw    s0, 0(sp)     # save s0 for use afterwards
      add   s0, a0, a1    # s0 = g + h
      add   s1, a2, a3    # s1 = i + j
      sub   a0, s0, s1    # (g + h) - (i + j)
      lw    s0, 0(sp)     # restore register s0 for caller
      lw    s1, 4(sp)     # restore register s1 for caller
      addi  sp, sp, 8     # adjust stack to delete 2 items
      jr    ra            # save ra
```
- We didn't need to save `ra` because it doesn't call another function