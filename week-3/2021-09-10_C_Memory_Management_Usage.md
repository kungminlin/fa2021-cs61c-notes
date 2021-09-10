# C Memory Management, Usage

## Table of Contents
- [C Memory Management, Usage](#c-memory-management-usage)
  - [Table of Contents](#table-of-contents)
  - [Variable Allocation](#variable-allocation)
    - [The Stack](#the-stack)
  - [Heaps](#heaps)
    - [`void *malloc(size_t n)`](#void-mallocsize_t-n)
    - [`void free(void *p)`](#void-freevoid-p)
    - [Using Dynamic Memory](#using-dynamic-memory)
    - [Summary](#summary)
    - [`malloc`/`free` Under the Hood](#mallocfree-under-the-hood)
    - [Fragmentation](#fragmentation)
  - [Common Memory Problems](#common-memory-problems)
  - [`realloc(p, size)`](#reallocp-size)

## Variable Allocation
- If declared outside a function, allocated in "static" storage
- If declared inside function, allocated on the "stack" and freed when the function returns

### The Stack
- Everytime a function is called, a new "stack frame" is allocated on the stack

**Stack frame**
- Return address
- Arguments
- Local variables
- Stack pointer indicates start of stack frame
- When function ends, stack pointer moves up and frees memory for future stack frames

## Heaps
- `malloc()`: Allocates a block of uninitialized memory
- `calloc()`: Allocates a block of zeroed memory
  - Recommended because more deterministic
- `free()`: Free previously allocated block of memory
- `realloc()`: Change size of previously allocated block
  - The memory location might move

### `void *malloc(size_t n)`
- Allocate a block of uninitialized memory
- `n` is an unsigned integer indicating size of the requested memory
- Returns `void*` pointer to block
- `NULL` return value indicates no more memory
```c
int *ip;
ip = (int *) malloc(sizeof(int));
```
```c
typedef struct { ... } TreeNode;
TreeNode *tp = (TreeNode *) malloc(sizeof(TreeNode));
```
- `calloc` takes `O(n)`, while `malloc` takes `O(1)`

### `void free(void *p)`
- `p` is pointer containing address originally returned by `malloc()`
```c
int *ip;
ip = (int *) malloc(sizeof(int));
...
free((void*) ip); /* Don't have to case, but better documentation */
```


### Using Dynamic Memory
```c
Node *create_node(int key, Node *left, Node *right) {
    Node *np;
    if (!(np = (Node*)))
}
```

### Summary
- Static storage is always constant
- Stack frames are created and destroyed last-in, first-out
- Heaps are tricker, because memory can be allocated/deallocated at any time

### `malloc`/`free` Under the Hood
- Operating system allows `malloc` library to ask for block of memory
  - e.g. `sbrk()` for UNIX

### Fragmentation
- Memory hierarchy creates continguous, small blocks of memory
- Scattered data

**Faster `malloc` Implementations**
- Power-of-2 "Buddy Allocator"

**Conservative Mark/Sweep Garbage Collectors**
- Stack and global variables are "live" memory
- Assume that every piece of memory in the starting set is a pointer
- Any block of memory that isn't live will be deallocated
- Java uses this approach
  - Java and Go uses incremental collector
- Problems: fragmentation and pauses


## Common Memory Problems
- Using uninitialized values
- Using memory that you don't own
  - Deallocated stack or heap variable
  - Out-of-bounds reference
  - Using `NULL` or garbage data as a pointer
  - Writing to a static string
- Memory Leaks
- Memory leaks are not as big a problem if program terminates quickly
  - Huge issue if running on small embedded system
- Solutions:
  - Diligently `free` memory
  - Use a "Conservative Garbage Collector" `malloc`
  - Quit and restart your program

## `realloc(p, size)`
- Resize a previously allocated block `p` to a new `size`
- If `p` is `NULL`, `realloc` behaves exactly like `malloc`

