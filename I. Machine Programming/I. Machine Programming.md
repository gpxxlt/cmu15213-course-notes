#Stack #Array #Struct
>[!quote] Procedure, *CSAPP Chapter 3.7*
Procedures are a key abstraction in software. They provide a way to package code that implements some functionality with a designated set of arguments and an optional return value. This function can then be invoked from different points in a program.
#### Outline
1. Procedures (Code Logistics)
- Stack Structure & Convention
- Passing Control: going to another function frame and returning
- Passing Data: arguments and return value
- Memory Management
2. Data
- Array: nested vs multilevel
- Structs: access patterns, alignment
3. Attack
- Return oriented attacks
- Defending schema
---
>[!quote] Procedure Mechanism
>Machine instructions implement the mechanisms, but the choices are determined by designers. These choices make up the **Application Binary Interface (ABI).** 


**stack vs heap? 
why stack?

#### Stack Conventions
1. Stack grows toward lower addresses `0x00...00`.
2. Stack Pointer `%rsp` contains lowest stack address. 
3. Push and pop are managed by `pushq` and `popq` instructions.
4. `popq` does not eliminate content; content remains there, only address change.
#### Control Transfer
Control transfer is done by two instructions `call` and `ret`. Procedure `P` calls procedure `Q`. 
1. `call Q` ***pushes*** an address `A` onto the stack and sets PC to beginning of `Q`. The pushed address `A` is referred to as the **return address** (address of some instruction in `P` that follows calling `Q`).
2. `ret` pops an address `A` off the stack and sets the PC to `A`.
#### Data Transfer
- When `P` calls `Q`, the code for `P` must first copy the arguments into the proper registers. Similarly, when `Q` returns back to `P`, the code for `P` can access the returned value in register `%rax`.
- First 6 arguments are stored in registers, 7th and beyond are stored on the stack. 
#### Data Management
>[!note] Local Storage in Registers
>**Caller-Saved** (aka “Call-Clobbered”) registers save temporary values in its frame before the call. Example registers include `%rax, %rdi, %rsi, %rdx, %rcx`. 
>
>**Callee-Saved** (aka “Call-Preserved”) registers saves temporary values in its frame before using. Callee restores them before returning to caller. Examples include `%rbx, %rbp, %rsp`.

The motivation is that, we must make sure that when one procedure (the caller) calls another (the callee), the callee ***does not overwrite*** some register value that the caller planned to use later.
#### Recursion
Recursion instances are stored as individual stack frames. This means that each function call has private storage where registers, local variables, and return address are saved.

--- 
#### Machine Data: Array

##### 1. Array Allocation
data stored using little endian

```c
int A1[3];     // Size: 12, Memory layout: [int][int][int]
int *A2[3];    // Size: 24, Memory layout: [int*][int*][int*]
int (*A3)[3];  // Size: 8,  Memory layout: [int*]->[int][int][int]
```
Sometimes we might encounter very complex expression:
```c

```

>[!note] Parsing Complex Declaration
>tbd
##### 2. Array Access
Two different approaches for implementing high dimensional arrays. 
![[Nested vs Multi-level Array.png]]
#### Machine Data: Structure
##### Alignment Motivation
why do we need to align

 Aligned Data
 Primitive data type requires B bytes
 Address must be multiple of B
 Required on some machines; advised on x86-64
 Motivation for Aligning Data
 Memory accessed by (aligned) chunks of 4 or 8 bytes (system dependent)
 Inefficient to load or store datum that spans cache lines (64 bytes).
Intel states should avoid crossing 16 byte boundaries.
 Virtual memory trickier when datum spans 2 pages (4 KB pages)
 Compiler
 Inserts gaps in structure to ensure correct alignment of fields

##### Alignment Requirement
internal padding & external padding


---
##### References
1. CSAPP Chapter 3.7