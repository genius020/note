# Accessing Information
<img width="723" height="824" alt="image" src="https://github.com/user-attachments/assets/8c050164-0447-4c0b-9219-30ea45cd6a2f"/> <br>

## What Are These Registers?

An x86-64 CPU has **16 general-purpose registers** that each hold **64 bits** (8 bytes) of data. Think of them as 16 super-fast storage compartments inside the CPU that can hold numbers or memory addresses (pointers).

## Historical Evolution and Naming

The naming seems weird because it evolved over time: 

1. **Original 8086 (1978)**: Had 8 registers, each 16-bit
   - Named:  `%ax`, `%bx`, `%cx`, `%dx`, `%si`, `%di`, `%bp`, `%sp`
   - Each had specific purposes (like `%ax` for accumulator, `%sp` for stack pointer)

2. **IA32 expansion (1985)**: Extended to 32-bit
   - Names changed to:  `%eax`, `%ebx`, `%ecx`, `%edx`, `%esi`, `%edi`, `%ebp`, `%esp`
   - The "e" prefix means "extended"

3. **x86-64 (2003)**: Extended to 64-bit and added 8 more registers
   - Original 8 became:  `%rax`, `%rbx`, `%rcx`, `%rdx`, `%rsi`, `%rdi`, `%rbp`, `%rsp`
   - 8 new registers:  `%r8`, `%r9`, `%r10`, `%r11`, `%r12`, `%r13`, `%r14`, `%r15`

## Accessing Different Sizes Within Registers

This is really clever:  each 64-bit register can be accessed in smaller chunks.  Looking at the diagram for `%rax`:

- **64-bit (full register)**: `%rax` - all 64 bits (bits 0-63)
- **32-bit (lower half)**: `%eax` - bits 0-31
- **16-bit (lower quarter)**: `%ax` - bits 0-15
- **8-bit (lowest byte)**: `%al` - bits 0-7

**Example:** If `%rax` contains the value `0x123456789ABCDEF0`:
- `%rax` = `0x123456789ABCDEF0` (64 bits)
- `%eax` = `0x9ABCDEF0` (lower 32 bits)
- `%ax` = `0xDEF0` (lower 16 bits)
- `%al` = `0xF0` (lower 8 bits)

## Important Rule About Setting Values

When you write to these registers, what happens to the rest of the bits depends on the size: 

**Rule 1:** Operations that write **1 or 2 bytes** leave the other bytes unchanged. 

**Example:**
```
Initial:   %rax = 0x123456789ABCDEF0
Write:    %al = 0x22 (1 byte)
Result:   %rax = 0x123456789ABCDE22  (only lowest byte changed)
```

**Rule 2:** Operations that write **4 bytes** (32-bit) automatically clear the upper 4 bytes to zero.

**Example:**
```
Initial:  %rax = 0x123456789ABCDEF0
Write:    %eax = 0x11111111 (4 bytes)
Result:   %rax = 0x0000000011111111  (upper 32 bits zeroed)
```

This second rule was introduced when moving from 32-bit to 64-bit to make the transition smoother.

## Special Roles of Registers

While most registers are flexible, they have conventional uses (shown on the right side of your diagram):

### **%rsp - Stack Pointer (Very Special)**
This is the most rigid register. It always points to the top of the run-time stack.  Many instructions automatically read from or write to `%rsp`.

**Example:** When you call a function, the CPU automatically decreases `%rsp` to make room on the stack for the return address.

### **Function Argument Registers (Fixed Convention)**
When calling a function, arguments are passed in these registers in order: 
- 1st argument → `%rdi`
- 2nd argument → `%rsi`
- 3rd argument → `%rdx`
- 4th argument → `%rcx`
- 5th argument → `%r8`
- 6th argument → `%r9`

**Example:** Calling `add(5, 10, 20)`:
```
%rdi = 5   (first argument)
%rsi = 10  (second argument)
%rdx = 20  (third argument)
```

### **%rax - Return Value**
Functions return their result in `%rax`.

**Example:** If a function computes `5 + 10` and returns `15`, the value `15` will be in `%rax` when the function finishes.

### **Callee-Saved vs Caller-Saved**
- **Callee-saved** (`%rbx`, `%rbp`, `%r12`-`%r15`): If a function uses these, it must restore their original values before returning.
- **Caller-saved** (`%r10`, `%r11`): A function can freely modify these without restoring them.

**Example:** 
```
Function A calls Function B
- Function A stores important data in %rbx (callee-saved)
- Function B can use %rbx, but MUST restore it before returning
- Function A knows %rbx will still have its data after calling B
- But if A stored data in %r10 (caller-saved), it might be lost after calling B
```

## Practical Example: Simple Addition

Let's say we want to compute `result = a + b + c`:

```assembly
movq $5, %rdi      # a = 5 (put in first argument register)
movq $10, %rsi     # b = 10 (put in second argument register)
movq $20, %rdx     # c = 20 (put in third argument register)

addq %rsi, %rdi    # %rdi = %rdi + %rsi = 5 + 10 = 15
addq %rdx, %rdi    # %rdi = %rdi + %rdx = 15 + 20 = 35

movq %rdi, %rax    # Put result in %rax (return value register)
```

After this code runs, `%rax` contains `35`.
- These conventions are covered in detail in Section 3.7 when discussing procedure implementation

The flexibility of these registers combined with established conventions allows efficient function calls, data manipulation, and memory management in x86-64 assembly code. 

# Operand Specifiers
<img width="723" height="426" alt="image" src="https://github.com/user-attachments/assets/eb790671-ce77-42f5-a9b6-48ccacf1f2ea" /> <br>
# x86-64 Operand Types:  A Detailed Explanation

In x86-64 assembly, instructions need operands - these tell the CPU where to get data (source) and where to put results (destination). Let me explain the three main types of operands with clear examples.

## The Three Operand Types

### **Type 1: Immediate Operands (Constants)**

These are literal constant values written directly in the instruction.  In AT&T syntax (used in this text), they're written with a `$` prefix followed by the number. 

**Examples:**
```assembly
movq $5, %rax          # Put the constant 5 into %rax
addq $100, %rbx        # Add the constant 100 to %rbx
movq $-577, %rcx       # Negative numbers work too
movq $0x1F, %rdx       # Hexadecimal:  0x1F = 31 in decimal
```

The value is literally baked into the instruction itself.  If you write `$100`, the CPU knows you mean exactly the number 100, not a memory address or register. 

**Important detail:** Different instructions support different ranges of immediate values. For example, some might only accept values that fit in 1 byte (-128 to 127), while others accept larger values.  The assembler is smart - it automatically chooses the most compact encoding for whatever value you provide.

### **Type 2: Register Operands**

These refer to the contents of one of the 16 registers we discussed earlier. The notation `R[ra]` means "the value stored in register ra."

**Examples:**
```assembly
movq %rax, %rbx        # Copy the value from %rax to %rbx
                       # Source:  R[%rax], Destination: R[%rbx]

addq %rcx, %rdx        # Add R[%rcx] to R[%rdx], store result in %rdx
                       # %rdx = %rdx + %rcx

movl %eax, %ebx        # 32-bit operation using lower halves
```

**Concrete example:**
```
If %rax contains 50 and %rbx contains 20:
movq %rax, %rbx    →  Now %rbx contains 50

If %rcx contains 7 and %rdx contains 10:
addq %rcx, %rdx    →  Now %rdx contains 17 (10 + 7)
```

The register can be any of the 8-, 16-, 32-, or 64-bit versions depending on the instruction: 
- 64-bit: `%rax, %rbx, %rcx... `
- 32-bit: `%eax, %ebx, %ecx...`
- 16-bit: `%ax, %bx, %cx... `
- 8-bit: `%al, %bl, %cl...`

### **Type 3: Memory Operands**

These access data stored in RAM at a calculated memory address (called the "effective address"). The notation `M[Addr]` means "the value in memory at address Addr." Memory is viewed as a giant array of bytes. 

This is the most complex type because there are multiple ways to calculate which memory address to access.  Let's go through each addressing mode:

---

## Memory Addressing Modes

### **1. Absolute Addressing:  `Imm`**

The simplest form - just a direct memory address. 

**Form:** `Imm`  
**Effective Address:** `Imm`  
**Meaning:** Access memory at the exact address `Imm`

**Example:**
```assembly
movq 0x1000, %rax      # Read 8 bytes from memory address 0x1000
```

If memory at address 0x1000 contains the value 42, then after this instruction `%rax` will contain 42.

---

### **2. Indirect Addressing: `(rb)`**

Use the value in a register as the memory address.

**Form:** `(rb)`  
**Effective Address:** `R[rb]`  
**Meaning:** Access memory at the address stored in register `rb`

**Example:**
```assembly
movq (%rax), %rbx      # Read from the address stored in %rax
```

**Concrete scenario:**
```
If %rax contains 0x2000, and memory at address 0x2000 contains the value 99:
movq (%rax), %rbx    →  %rbx now contains 99
```

This is like dereferencing a pointer in C:  `rbx = *rax`

---

### **3. Base + Displacement:  `Imm(rb)`**

Add a constant offset to a register value.

**Form:** `Imm(rb)`  
**Effective Address:** `Imm + R[rb]`  
**Meaning:** Access memory at address `(value in rb) + Imm`

**Example:**
```assembly
movq 8(%rax), %rbx     # Read from address (%rax + 8)
```

**Concrete scenario:**
```
If %rax contains 0x2000:
- Address = 0x2000 + 8 = 0x2008
- Read 8 bytes starting at 0x2008 into %rbx
```

**Real-world use:** Accessing structure fields. 
```c
struct Point {
    long x;    // offset 0
    long y;    // offset 8
};

struct Point *p;  // stored in %rax
```

```assembly
movq 0(%rax), %rbx     # p->x (offset 0)
movq 8(%rax), %rcx     # p->y (offset 8)
```

---

### **4. Indexed:  `(rb, ri)`**

Add two registers together to form the address.

**Form:** `(rb, ri)`  
**Effective Address:** `R[rb] + R[ri]`  
**Meaning:** Access memory at address `(value in rb) + (value in ri)`

**Example:**
```assembly
movq (%rax, %rcx), %rbx    # Read from address (%rax + %rcx)
```

**Concrete scenario:**
```
If %rax contains 0x3000 and %rcx contains 0x100:
- Address = 0x3000 + 0x100 = 0x3100
- Read from 0x3100 into %rbx
```

---

### **5. Indexed with Displacement: `Imm(rb, ri)`**

Combine a constant offset with two registers.

**Form:** `Imm(rb, ri)`  
**Effective Address:** `Imm + R[rb] + R[ri]`  
**Meaning:** Access memory at address `Imm + (value in rb) + (value in ri)`

**Example:**
```assembly
movq 16(%rax, %rcx), %rbx    # Read from (16 + %rax + %rcx)
```

**Concrete scenario:**
```
If %rax = 0x4000, %rcx = 0x50: 
- Address = 16 + 0x4000 + 0x50 = 0x4060
- Read from 0x4060 into %rbx
```

---

### **6. Scaled Indexed: `(rb, ri, s)`**

Multiply the index register by a scale factor (1, 2, 4, or 8) before adding. 

**Form:** `(rb, ri, s)`  
**Effective Address:** `R[rb] + (R[ri] * s)`  
**Meaning:** Access memory at address `(value in rb) + (value in ri × scale)`

**Example:**
```assembly
movq (%rax, %rcx, 4), %rbx    # Read from (%rax + %rcx * 4)
```

**Concrete scenario:**
```
If %rax = 0x5000, %rcx = 3: 
- Address = 0x5000 + (3 * 4) = 0x5000 + 12 = 0x500C
- Read from 0x500C into %rbx
```

**Real-world use:** Array indexing.
```c
int array[10];    // base address in %rax
int i;           // index in %rcx

array[i] = ... ;
```

```assembly
# Each int is 4 bytes, so array[i] is at (base + i*4)
movl (%rax, %rcx, 4), %ebx    # Load array[i] into %ebx
```

---

### **7. Scaled Indexed with Offset: `Imm(rb, ri, s)` (The Most General Form)**

This combines everything:  a constant, a base register, an index register scaled by a factor. 

**Form:** `Imm(rb, ri, s)`  
**Effective Address:** `Imm + R[rb] + (R[ri] * s)`  
**Meaning:** Access memory at `Imm + (value in rb) + (value in ri × scale)`

**Example:**
```assembly
movq 24(%rax, %rcx, 8), %rbx    # Read from (24 + %rax + %rcx * 8)
```

**Concrete scenario:**
```
If %rax = 0x6000, %rcx = 2:
- Address = 24 + 0x6000 + (2 * 8) = 24 + 0x6000 + 16 = 0x6028
- Read from 0x6028 into %rbx
```

**Real-world use:** Accessing elements in arrays of structures.
```c
struct Data {
    long a;      // offset 0
    long b;      // offset 8
    long c;      // offset 16
};

struct Data array[100];    // base in %rdi
int i;                     // index in %rsi

array[i].c = ...;
```

```assembly
# Each struct is 24 bytes
# array[i].c is at: base + i*24 + 16
movq 16(%rdi, %rsi, 24), %rax
```

Wait - that won't work because 24 isn't a valid scale (only 1, 2, 4, 8 are allowed)!  You'd need to do: 
```assembly
imulq $24, %rsi, %rcx        # %rcx = i * 24
movq 16(%rdi, %rcx, 1), %rax # or just 16(%rdi, %rcx)
```

---

## Why These Modes Exist

All these addressing modes exist for efficiency.  The CPU can calculate complex addresses in a single instruction rather than requiring multiple instructions. For example, accessing `array[i].field` can be done in one memory access instead of several arithmetic operations followed by a memory access.

Every addressing mode is just a special case of the most general form `Imm(rb, ri, s)`:
- Leave out `Imm` → `(rb, ri, s)`
- Leave out `ri` → `Imm(rb)`
- Leave out `s` (or use s=1) → `Imm(rb, ri)`
- Leave out both → `Imm` or `(rb)`

Understanding these addressing modes is crucial for reading assembly code, especially when dealing with arrays and structures, which you'll see extensively when the text covers data structures and procedure implementations. 

