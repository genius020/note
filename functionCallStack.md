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
## x86-64 Operand Types:  A Detailed Explanation

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

# Data Movement Instructions
## Data Movement Instructions in x86-64:  A Detailed Explanation

Data movement instructions are among the most frequently used in assembly programming.  They copy data from one location to another.  Let me explain the different types with comprehensive examples.

## The MOV Class:  Basic Data Movement

The MOV class consists of four instructions that differ only in the size of data they move:

- **movb**: Move **byte** (1 byte = 8 bits)
- **movw**: Move **word** (2 bytes = 16 bits)
- **movl**: Move **long/double word** (4 bytes = 32 bits)
- **movq**: Move **quad word** (8 bytes = 64 bits)

These instructions copy data from source to destination without any transformation. 

### Operand Rules

**Source operand** can be:
1.  Immediate (constant value)
2. Register
3. Memory

**Destination operand** can be:
1. Register
2. Memory

**Critical restriction:** Both operands CANNOT be memory at the same time. You cannot do `movq memory1, memory2` directly.  You must use two instructions:
```assembly
movq memory1, %rax      # Load from memory to register
movq %rax, memory2      # Store from register to memory
```

### Register Size Matching

The register size must match the instruction suffix:
- `movb` uses 8-bit registers:  `%al, %bl, %cl, %dl, %sil, %dil, %spl, %bpl, %r8b-%r15b`
- `movw` uses 16-bit registers: `%ax, %bx, %cx, %dx, %si, %di, %sp, %bp, %r8w-%r15w`
- `movl` uses 32-bit registers: `%eax, %ebx, %ecx, %edx, %esi, %edi, %esp, %ebp, %r8d-%r15d`
- `movq` uses 64-bit registers: `%rax, %rbx, %rcx, %rdx, %rsi, %rdi, %rsp, %rbp, %r8-%r15`

---

## How MOV Instructions Affect Destination Registers

This is crucial to understand:

### General Rule: 
MOV instructions update ONLY the bytes specified by the destination operand, **except for one special case.**

### The Exception:
When `movl` (32-bit move) has a **register** as the destination, it sets the upper 4 bytes (high-order 32 bits) to zero.  This is an x86-64 convention:  any instruction generating a 32-bit value in a register automatically zeros the upper 32 bits.

### Detailed Example: 

```assembly
movabsq $0x0011223344556677, %rax    # %rax = 0011223344556677
movb $-1, %al                         # %rax = 00112233445566FF
movw $-1, %ax                         # %rax = 001122334455FFFF
movl $-1, %eax                        # %rax = 00000000FFFFFFFF
movq $-1, %rax                        # %rax = FFFFFFFFFFFFFFFF
```

Let me break this down step by step:

**Line 1:** Initialize `%rax` with the 64-bit pattern `0x0011223344556677`
```
%rax = 00 11 22 33 44 55 66 77
       [byte7.... byte1 byte0]
```

**Line 2:** `movb $-1, %al` 
- `-1` in 8-bit hexadecimal is `0xFF` (all bits set to 1)
- Only modifies the lowest byte (`%al`)
- Other bytes remain unchanged
```
%rax = 00 11 22 33 44 55 66 FF
                          ^^ changed
```

**Line 3:** `movw $-1, %ax`
- `-1` in 16-bit hexadecimal is `0xFFFF`
- Modifies the lowest 2 bytes (`%ax`)
- Other bytes remain unchanged
```
%rax = 00 11 22 33 44 55 FF FF
                       ^^^^^ changed
```

**Line 4:** `movl $-1, %eax`
- `-1` in 32-bit hexadecimal is `0xFFFFFFFF`
- Modifies the lowest 4 bytes (`%eax`)
- **SPECIAL:  Upper 4 bytes are zeroed** (this is the exception!)
```
%rax = 00 00 00 00 FF FF FF FF
       ^^^^^^^^^^^ zeroed! 
       ^^^^^^^^^^^ changed
```

**Line 5:** `movq $-1, %rax`
- `-1` in 64-bit hexadecimal is `0xFFFFFFFFFFFFFFFF`
- Modifies all 8 bytes
```
%rax = FF FF FF FF FF FF FF FF
       ^^^^^^^^^^^^^^^^^^^^^^^^ all changed
```

---

## Five Types of MOV Operations

The text provides five examples showing all possible source-destination combinations:

### 1. Immediate → Register
```assembly
movl $0x4050, %eax
```
- Source: immediate value `0x4050`
- Destination: register `%eax`
- Size: 4 bytes
- Result: `%eax = 0x00004050`, and upper 32 bits of `%rax` are zeroed

### 2. Register → Register
```assembly
movw %bp, %sp
```
- Source: register `%bp` (16-bit base pointer)
- Destination: register `%sp` (16-bit stack pointer)
- Size: 2 bytes
- Result: Copy the 16-bit value from `%bp` to `%sp`

### 3. Memory → Register
```assembly
movb (%rdi,%rcx), %al
```
- Source: memory at address `R[%rdi] + R[%rcx]` (indexed addressing)
- Destination: register `%al` (low byte of `%rax`)
- Size: 1 byte
- Result:  Read 1 byte from computed memory address into `%al`

**Example:** If `%rdi = 0x1000` and `%rcx = 0x5`, read from address `0x1005`

### 4. Immediate → Memory
```assembly
movb $-17, (%esp)
```
- Source: immediate value `-17` (which is `0xEF` in 8-bit two's complement)
- Destination: memory at address `R[%esp]`
- Size: 1 byte
- Result: Write the byte `0xEF` to the memory location pointed to by `%esp`

### 5. Register → Memory
```assembly
movq %rax, -12(%rbp)
```
- Source: register `%rax`
- Destination: memory at address `R[%rbp] - 12` (base + displacement)
- Size: 8 bytes
- Result: Write 8 bytes from `%rax` to memory at address `(%rbp - 12)`

**Example:** If `%rbp = 0x2000`, write to address `0x2000 - 12 = 0x1FF4`

---

## The movabsq Instruction:  Moving Large 64-bit Immediates

Regular `movq` has a limitation: immediate values must fit in 32 bits (as signed integers). The value is then sign-extended to 64 bits.

**Example of limitation:**
```assembly
movq $0x123456789ABCDEF0, %rax    # ERROR!  Too large for regular movq
```

## Understanding movq Immediate Value Limitations

Let me explain this limitation clearly with examples.

## The Basic Problem

The regular `movq` instruction has a restriction when using immediate (constant) values:  **The immediate value must fit within 32 bits as a signed number.**

## What Does "32-bit Two's-Complement" Mean?

In 32-bit two's-complement representation, you can represent numbers in this range:
- **Minimum:** -2,147,483,648 (which is -2³¹)
- **Maximum:** +2,147,483,647 (which is 2³¹ - 1)

In hexadecimal:
- **Range:** `0x80000000` to `0x7FFFFFFF` (when interpreted as signed)

## How Sign Extension Works

When you use `movq` with an immediate value that fits in 32 bits, the CPU:
1. Takes your 32-bit value
2. Looks at the most significant bit (MSB) - the sign bit
3. Extends it to 64 bits by copying the sign bit to all upper 32 bits

**If MSB = 0 (positive):** Fill upper 32 bits with zeros  
**If MSB = 1 (negative):** Fill upper 32 bits with ones

---

## Examples That Work with Regular movq

### Example 1: Small Positive Number
```assembly
movq $100, %rax
```

**Process:**
- 100 in 32-bit:  `0x00000064`
- MSB = 0 (positive)
- Sign extend to 64 bits: `0x0000000000000064`

**Result:** `%rax = 0x0000000000000064` ✅

---

### Example 2: Larger Positive Number
```assembly
movq $0x7FFFFFFF, %rax    # 2,147,483,647 (max positive 32-bit)
```

**Process:**
- 32-bit value: `0x7FFFFFFF`
- MSB = 0 (bit 31 is 0, so positive)
- Sign extend: `0x000000007FFFFFFF`

**Result:** `%rax = 0x000000007FFFFFFF` ✅

---

### Example 3: Negative Number
```assembly
movq $-1, %rax
```

**Process:**
- -1 in 32-bit two's-complement: `0xFFFFFFFF`
- MSB = 1 (negative)
- Sign extend by copying the 1 bit:  `0xFFFFFFFFFFFFFFFF`

**Result:** `%rax = 0xFFFFFFFFFFFFFFFF` ✅

This correctly represents -1 in 64-bit! 

---

### Example 4: Another Negative Number
```assembly
movq $-100, %rax
```

**Process:**
- -100 in 32-bit two's-complement: `0xFFFFFF9C`
- MSB = 1 (negative)
- Sign extend: `0xFFFFFFFFFFFFFF9C`

**Result:** `%rax = 0xFFFFFFFFFFFFFF9C` ✅

This correctly represents -100 in 64-bit!

---

### Example 5: Small Negative Number
```assembly
movq $-2147483648, %rax    # Minimum 32-bit signed value
```

**Process:**
- -2,147,483,648 in 32-bit: `0x80000000`
- MSB = 1 (negative)
- Sign extend: `0xFFFFFFFF80000000`

**Result:** `%rax = 0xFFFFFFFF80000000` ✅

---

## Examples That DON'T Work with Regular movq

### Example 6: Large Positive Number (Too Big!)
```assembly
movq $0x123456789ABCDEF0, %rax    # ❌ ERROR!
```

**Problem:**
- This value is `0x123456789ABCDEF0`
- This requires more than 32 bits to represent
- The upper part `0x12345678` doesn't fit in 32 bits
- **This will cause an assembler error! **

---

### Example 7: Value Just Beyond 32-bit Range
```assembly
movq $0x80000000, %rax    # ❌ Doesn't work as you might expect! 
```

**Problem:**
- You might want:  `%rax = 0x0000000080000000` (2,147,483,648 unsigned)
- But `0x80000000` in 32-bit two's-complement is **-2,147,483,648** (MSB=1)
- Sign extension gives:  `%rax = 0xFFFFFFFF80000000` (negative number!)

**This is legal, but might not do what you want if you intended an unsigned value.**

---

### Example 8: Medium-Large Positive Number
```assembly
movq $0x100000000, %rax    # ❌ ERROR!
```

**Problem:**
- This is 2³² = 4,294,967,296
- Needs 33 bits to represent as a positive number
- Cannot fit in 32-bit signed range
- **Assembler error!**

---

## Visual Comparison

| Value You Want | Can Use movq?  | Why?  |
|----------------|---------------|------|
| `0x00000000FFFFFFFF` | ❌ No | Would need to represent 4,294,967,295, which is > 2³¹-1 |
| `0x000000007FFFFFFF` | ✅ Yes | 2,147,483,647 fits in 32-bit signed |
| `0x0000000080000000` | ❌ No | Can't represent as positive; movq would interpret as negative and extend to `0xFFFFFFFF80000000` |
| `0xFFFFFFFFFFFFFFFF` | ✅ Yes | Use movq $-1, which sign-extends correctly |
| `0x123456789ABCDEF0` | ❌ No | Far too large for 32 bits |

---

## The Solution: movabsq

When you need a full 64-bit immediate value, use `movabsq`:

```assembly
movabsq $0x123456789ABCDEF0, %rax    # ✅ Works!
```

**Result:** `%rax = 0x123456789ABCDEF0`

No sign extension, no limitations - the full 64-bit value is loaded directly. 

---

## Practical Examples

### Scenario 1: Setting up a pointer to high memory
```assembly
# Won't work: 
movq $0x7FFFFFFFFFFF, %rax    # ❌ Too large! 

# Must use:
movabsq $0x7FFFFFFFFFFF, %rax    # ✅ Correct
```

---

### Scenario 2: Loading a bit pattern
```assembly
# Want: %rax = 0x00000000FFFFFFFF (all lower 32 bits set)

# Won't work as intended:
movq $0xFFFFFFFF, %rax    # ❌ Gives 0xFFFFFFFFFFFFFFFF (sign extended!)

# Must use:
movabsq $0x00000000FFFFFFFF, %rax    # ✅ Correct

# OR use this trick:
movl $0xFFFFFFFF, %eax    # ✅ Also works!  (zeros upper 32 bits)
```

---

### Scenario 3: Small values always work
```assembly
movq $0, %rax           # ✅ %rax = 0x0000000000000000
movq $100, %rax         # ✅ %rax = 0x0000000000000064
movq $-50, %rax         # ✅ %rax = 0xFFFFFFFFFFFFFFCE
movq $0x7FFFFFFF, %rax  # ✅ %rax = 0x000000007FFFFFFF
```

---

The `movabsq` instruction solves this: 
- Can have any arbitrary 64-bit immediate value
- Destination must be a register (cannot be memory)

**Example:**
```assembly
movabsq $0x123456789ABCDEF0, %rax    # OK!  %rax = 123456789ABCDEF0
```

The "abs" stands for "absolute" - meaning the full absolute 64-bit value. 

---

## MOVZ Class: Move with Zero Extension

These instructions copy a smaller source value to a larger destination and fill the remaining high-order bytes with zeros.

The instruction name format is `movz[source_size][dest_size]`:
- First letter after `movz`: source size (`b`=byte, `w`=word)
- Second letter:  destination size (`w`=word, `l`=long, `q`=quad)

### Available Instructions: 

1. **movzbw**:  Zero-extend byte → word (1 byte → 2 bytes)
2. **movzbl**: Zero-extend byte → long (1 byte → 4 bytes)
3. **movzwl**: Zero-extend word → long (2 bytes → 4 bytes)
4. **movzbq**: Zero-extend byte → quad (1 byte → 8 bytes)
5. **movzwq**:  Zero-extend word → quad (2 bytes → 8 bytes)

**Important:** Source can be register or memory, but destination must be a register.

### Example:
```assembly
movzbq %dl, %rax
```

**Scenario:** `%dl = 0xA5` (binary: 10100101)

**Result:** `%rax = 0x00000000000000A5`
```
Source:   1010 0101
Result:  0000... 0000 1010 0101
         [56 zeros]  [8 bits]
```

The high-order 56 bits are filled with zeros. 

---

## MOVS Class: Move with Sign Extension

These instructions copy a smaller source value to a larger destination and fill the remaining high-order bytes by **sign extension** - replicating the most significant bit (sign bit) of the source.

The instruction name format is `movs[source_size][dest_size]`:

### Available Instructions:

1. **movsbw**: Sign-extend byte → word (1 byte → 2 bytes)
2. **movsbl**: Sign-extend byte → long (1 byte → 4 bytes)
3. **movswl**: Sign-extend word → long (2 bytes → 4 bytes)
4. **movsbq**: Sign-extend byte → quad (1 byte → 8 bytes)
5. **movswq**: Sign-extend word → quad (2 bytes → 8 bytes)
6. **movslq**: Sign-extend long → quad (4 bytes → 8 bytes)

**Important:** Destination must be a register.

### Example:
```assembly
movsbq %dl, %rax
```

**Scenario 1:** `%dl = 0x05` (binary: 00000101, positive number, MSB = 0)
**Result:** `%rax = 0x0000000000000005`
```
Sign bit = 0, so fill with zeros
```

**Scenario 2:** `%dl = 0xAA` (binary: 10101010, negative number, MSB = 1)
**Result:** `%rax = 0xFFFFFFFFFFFFFFAA`
```
Sign bit = 1, so fill with ones (FF repeated)
```

This preserves the sign of signed integers in two's complement representation.

---

## Missing Instruction: movzlq (Why it doesn't exist)

You might expect a `movzlq` instruction to zero-extend a 4-byte (long) value to 8 bytes (quad). **This instruction doesn't exist** because it's not needed! 

Remember the special rule: `movl` with a register destination automatically zeros the upper 4 bytes.  So `movl` already performs zero extension from 32 to 64 bits.

**Instead of:**
```assembly
movzlq %eax, %rax    # This instruction doesn't exist
```

**Just use:**
```assembly
movl %eax, %rax      # This zeros the upper 32 bits automatically
```

**Example:**
```assembly
movl $0x12345678, %eax    # %rax = 0x0000000012345678
```

The upper 32 bits are automatically zeroed, achieving the same effect as zero extension.

---

## The cltq Instruction: Compact Sign Extension

`cltq` is a special instruction for sign-extending `%eax` to `%rax`:
- **No operands needed** (implicit)
- Source: always `%eax`
- Destination: always `%rax`
- More compact encoding than `movslq %eax, %rax`

**Example:**
```assembly
cltq    # Sign-extend %eax to %rax
```

If `%eax = 0x80000000` (negative in 32-bit two's complement):
```
Before: %rax = ? ?????? ???  80000000
After:  %rax = FFFFFFFF 80000000
                ^^^^^^^^ sign extended
```

If `%eax = 0x12345678` (positive):
```
Before: %rax = ? ????????? 12345678
After:  %rax = 00000000 12345678
                ^^^^^^^^ sign extended
```

---

## Comparing Byte Movement Instructions:  A Detailed Example

Let's compare `movb`, `movsbq`, and `movzbq`:

```assembly
movabsq $0x0011223344556677, %rax    # %rax = 0011223344556677
movb $0xAA, %dl                       # %dl = AA
movb %dl, %al                         # %rax = 00112233445566AA
movsbq %dl, %rax                      # %rax = FFFFFFFFFFFFFFAA
movzbq %dl, %rax                      # %rax = 00000000000000AA
```

### Step-by-step breakdown:

**Line 1:** Initialize
```
%rax = 0x0011223344556677
```

**Line 2:** Load `0xAA` into `%dl`
```
%dl = 0xAA (binary: 10101010)
```

**Line 3:** `movb %dl, %al` - Simple byte move
- Copies only the low byte
- Other bytes of `%rax` unchanged
```
%rax = 0x00112233445566AA
                        ^^ only this changed
```

**Line 4:** `movsbq %dl, %rax` - Sign-extended move
- `%dl = 0xAA` has MSB = 1 (binary: 1010 1010)
- Sign bit is 1, so fill all upper bits with 1
```
%rax = 0xFFFFFFFFFFFFFFAA
       ^^^^^^^^^^^^^^ all F's (sign extension)
                    ^^ original AA
```

In hexadecimal, `0xAA` is a negative number in 8-bit two's complement (-86 in decimal). Sign extension preserves this as a negative 64-bit number.

**Line 5:** `movzbq %dl, %rax` - Zero-extended move
- Fill all upper bits with 0
```
%rax = 0x00000000000000AA
       ^^^^^^^^^^^^^^ all zeros
                    ^^ original AA
```

Zero extension treats the source as an unsigned number. 

---

## Practical Use Cases

### When to use movz (zero extension):
- Working with unsigned integers
- Array indexing (indices are unsigned)
- Bit manipulation

**Example:**
```c
unsigned char c = 200;
unsigned long x = c;    // Need zero extension
```
```assembly
movzbq %cl, %rax    # x = 200 (0x00000000000000C8)
```

### When to use movs (sign extension):
- Working with signed integers
- Preserving negative values

**Example:**
```c
signed char c = -56;    // 0xC8 in 8-bit two's complement
signed long x = c;      // Need sign extension
```
```assembly
movsbq %cl, %rax    # x = -56 (0xFFFFFFFFFFFFFFC8)
```

---

This comprehensive explanation covers all the data movement instructions, their behaviors, special cases, and the subtle but critical differences between them. Understanding these instructions is fundamental to reading and writing x86-64 assembly code, especially when dealing with data of different sizes and types. 

# Procedures
## Understanding Procedures at the Machine Level

Procedures (also called functions, methods, or subroutines) are a fundamental abstraction in programming. They package code with a defined interface:  they accept arguments and optionally return a value. This allows you to hide implementation details while providing a clear, reusable interface. 

---

## The Three Key Mechanisms

When procedure P calls procedure Q, and Q executes and returns to P, three mechanisms must work: 

---

### 1. Passing Control

The CPU must transfer execution from P to Q, then back to P after Q finishes.

**What happens:**
- The **program counter** must jump to Q's first instruction (entry)
- When Q finishes, the program counter must return to the instruction in P right after the call (return)

**The challenge:** The CPU must remember **where to return to**—it needs to save the return address. 

**Example:**
```c
void caller() {
    // ...  some code
    callee();        // Jump to callee, save return address
    // ... continue here after callee returns
}
```

In assembly:
```assembly
caller:
    # ...  some instructions
    call callee      # Saves return address and jumps to callee
    # ... execution resumes here after callee returns
```

The `call` instruction:
1. Pushes the return address (address of next instruction) onto the stack
2. Sets program counter to the first instruction of `callee`

The `ret` instruction in `callee`:
1. Pops the return address from the stack
2. Sets program counter to that address (returns to `caller`)

---

### 2. Passing Data

P must send arguments to Q, and Q must return a value to P.

**Arguments (P → Q):** Values passed from caller to callee  
**Return value (Q → P):** Value passed back from callee to caller

**How x86-64 does it:**
- First 6 integer/pointer arguments go in registers:  `%rdi, %rsi, %rdx, %rcx, %r8, %r9`
- Return value goes in register `%rax`
- Additional arguments (beyond 6) go on the stack

**Example:**
```c
long add(long a, long b) {
    return a + b;
}

long caller() {
    return add(5, 10);
}
```

Assembly:
```assembly
add:
    # a is in %rdi, b is in %rsi (passed by caller)
    addq %rsi, %rdi
    movq %rdi, %rax      # Put result in %rax (return value)
    ret

caller:
    movq $5, %rdi        # First argument: 5
    movq $10, %rsi       # Second argument: 10
    call add
    # After return, %rax contains 15
    ret
```

---

### 3. Allocating and Deallocating Memory

Q may need space for local variables when it starts, and must free that space before returning.

**Two approaches:**

**Option 1: Use registers** (fastest, no memory allocation needed)
```c
long add(long a, long b) {
    long sum = a + b;    // 'sum' can live in a register
    return sum;
}
```

**Option 2: Use the stack** (when you run out of registers or need to save values)
```c
long complex_function(long a, long b) {
    long var1 = a * 2;
    long var2 = b * 3;
    long var3 = var1 + var2;
    long var4 = var3 * 4;
    // ... many more variables
    return var4;
}
```

For many local variables, the stack is used: 
```assembly
complex_function:
    pushq %rbp           # Save old base pointer
    movq %rsp, %rbp      # Set up new stack frame
    subq $32, %rsp       # Allocate 32 bytes for local variables
    
    # ... function body uses stack space
    
    movq %rbp, %rsp      # Deallocate local variables
    popq %rbp            # Restore old base pointer
    ret
```

---

## The Minimalist Strategy

x86-64 follows a **"do only what's necessary"** approach for efficiency:

- Procedure calls happen millions/billions of times in a program
- Overhead must be minimized
- Only implement mechanisms that are actually needed for each specific procedure

**Examples:**

**Simple procedure (minimal overhead):**
```c
long add(long a, long b) {
    return a + b;
}
```
- Arguments in registers ✓
- Return value in register ✓
- No local variables → **no stack allocation needed**
- Just `call` and `ret` instructions

**Complex procedure (uses more mechanisms):**
```c
long complex(long a, long b, long c) {
    long temp1 = a * b;
    long temp2 = b * c;
    long temp3 = a * c;
    helper(temp1);       // Calls another function
    return temp1 + temp2 + temp3;
}
```
- Arguments in registers ✓
- Multiple local variables → **allocate stack space**
- Calls another function → **save return address**
- May need to save/restore registers

---

## Complete Example with All Three Mechanisms

```c
long add_and_multiply(long a, long b) {
    long sum = a + b;
    long result = sum * 2;
    return result;
}

long caller() {
    long x = 5;
    long y = 10;
    long answer = add_and_multiply(x, y);
    return answer;
}
```

**Assembly:**
```assembly
add_and_multiply: 
    # MECHANISM 1: Control - entered via 'call', will exit via 'ret'
    
    # MECHANISM 2: Data passing - arguments in %rdi (a=5), %rsi (b=10)
    addq %rsi, %rdi       # sum = a + b  (%rdi = 15)
    
    # MECHANISM 3: Memory - using registers, no stack allocation needed
    leaq (%rdi,%rdi), %rax  # result = sum * 2  (%rax = 30)
    
    # MECHANISM 2: Return value in %rax (30)
    ret                     # MECHANISM 1: Return control to caller

caller:
    # MECHANISM 2: Set up arguments
    movq $5, %rdi           # First argument
    movq $10, %rsi          # Second argument
    
    # MECHANISM 1: Transfer control
    call add_and_multiply   # Pushes return address, jumps to function
    
    # Execution resumes here after return
    # MECHANISM 2: Return value is in %rax (30)
    
    ret
```

**Tracing execution:**

1. **Before call:** `%rdi = 5`, `%rsi = 10`
2. **call instruction:** Pushes return address onto stack, jumps to `add_and_multiply`
3. **Inside add_and_multiply:** 
   - Reads arguments from `%rdi` and `%rsi`
   - Computes result using registers (no stack needed)
   - Puts result (30) in `%rax`
4. **ret instruction:** Pops return address, jumps back to `caller`
5. **After return:** `%rax = 30`, execution continues in `caller`

---

# The Run-Time Stack
## Stack Frames and Procedure Calls

The procedure-calling mechanism in C and most other languages takes advantage of a fundamental property of the stack data structure: **last-in, first-out (LIFO) memory management**. This property perfectly matches the way procedures are called and return in a program.

---

## Why Procedures Match Stack Behavior

Consider the scenario where procedure P calls procedure Q. While Q is executing, procedure P is temporarily suspended—it's waiting for Q to finish. In fact, P doesn't just sit idle; it's actually paused in the middle of its own execution.  If you trace back from Q, there's an entire chain of procedure calls:  maybe some procedure R called P, and some procedure S called R, and so on, all the way back to the program's starting point (typically `main`). All of these procedures in the call chain are suspended, waiting for the procedures they called to return.

The crucial observation is this: while Q is running, **only Q** needs the ability to allocate new storage for its local variables or to set up calls to other procedures. All the other procedures in the call chain (P, R, S, etc.) don't need to do anything—they're frozen in time, waiting.  When Q eventually returns, any local storage it allocated can be completely freed because Q no longer exists as an active procedure.  Then P resumes where it left off, and **P** becomes the only procedure that needs to actively manage storage.

This pattern of "most recent caller is the only one doing work" is exactly what a stack provides. As P calls Q, all the information needed for that call—how to pass control to Q, what data Q needs, where Q should store its local variables—gets pushed onto the end of the stack. When Q returns to P, all of Q's information gets popped off and deallocated. The stack naturally manages this because it follows LIFO discipline.

---

## Stack Frame Structure

In x86-64, the stack grows toward lower addresses (remember this counterintuitive fact), and the stack pointer `%rsp` always points to the top element of the stack. You can push and pop data with `pushq` and `popq`. You can also allocate uninitialized space by simply decrementing `%rsp` (moving it to a lower address), and you can deallocate that space by incrementing `%rsp` back. 

When a procedure needs more storage than what fits in registers, it allocates a region on the stack called its **stack frame**. Figure 3.25 shows the general structure of the run-time stack with multiple stack frames. 

The key principle is:  **the frame for the currently executing procedure is always at the top of the stack**.   This makes sense because the most recently called procedure is the one that's actively running and needs access to its storage.

---

## What Happens During a Procedure Call

Let's trace what happens when procedure P calls procedure Q:

1. **Before the call**, P is executing and P's stack frame is at the top of the stack. 

2. **P prepares to call Q** by setting up arguments.  The first six integer/pointer arguments go in registers (`%rdi, %rsi, %rdx, %rcx, %r8, %r9`). If Q needs more than six arguments, P stores the extra arguments in its own stack frame, in a region that will be accessible to Q.

3. **P executes `call Q`**. The `call` instruction pushes the return address onto the stack.  The return address is the memory address of the instruction in P that should execute after Q returns.  This return address is considered **part of P's stack frame** because it's state information relevant to P—it tells where P should resume. 

4. **Q begins executing**. Now Q needs its own stack frame. Q's code allocates space for this frame by extending the stack boundary, which means decrementing `%rsp` to move it to a lower address.  The amount Q subtracts from `%rsp` depends on how much storage Q needs. 

5. **Within Q's stack frame**, Q can do several things:
   - Save the values of registers that Q needs to use but that should be preserved for P
   - Allocate space for Q's local variables
   - Set up arguments for procedures that Q might call

6. **When Q finishes**, it deallocates its stack frame (typically by restoring `%rsp` to its value at entry), executes `ret` which pops the return address and jumps to it, and control returns to P right where P left off.

---

## Stack Frame Layout

Looking at Figure 3.25, we can see the general structure from bottom to top (remember, higher addresses are shown at the top of diagrams):

**Earlier frames** (procedures that called P): These are higher in memory (higher addresses). They're suspended, waiting for their called procedures to return.

**P's frame** (the calling procedure):
- **Argument 7, Argument 8, ...  Argument n**:  If P is calling Q with more than 6 arguments, the extra arguments go here, pushed before the call
- **(Between frames)** More space if needed
- **Return address**:  Pushed by the `call` instruction, this is where P should resume after Q returns

**Q's frame** (the currently executing procedure, at stack top):
- **Saved registers**: If Q needs to use certain registers but must preserve their values for P (callee-saved registers), Q saves them here
- **Local variables**:  Space for Q's local variables that don't fit in registers
- **Argument build area**: If Q calls another procedure and needs more than 6 arguments, Q stores the extra arguments here

The **stack pointer %rsp** points to the very top of Q's frame (lowest address in use).

---

## Concrete Example

Let's trace a concrete example with actual code:

```c
long add(long a, long b) {
    return a + b;
}

long compute(long x, long y, long z) {
    long temp1 = x * 2;
    long temp2 = y * 3;
    long result = add(temp1, temp2);
    return result + z;
}

long main() {
    return compute(5, 10, 15);
}
```

**Initial state:** `main` is executing, and its stack frame is at the top. 

**main calls compute(5, 10, 15):**

```assembly
main:
    # Set up arguments (all fit in registers)
    movq $5, %rdi      # First argument:  x = 5
    movq $10, %rsi     # Second argument: y = 10
    movq $15, %rdx     # Third argument: z = 15
    call compute       # Push return address, jump to compute
```

After `call compute`:
- Stack has return address (where main should resume)
- %rsp points to this return address
- compute begins executing

**compute needs a stack frame:**

```assembly
compute:
    pushq %rbp              # Save old frame pointer
    movq %rsp, %rbp         # Set up new frame pointer
    subq $16, %rsp          # Allocate 16 bytes for local variables
```

Stack now looks like (addresses are examples):
```
Address    Content
0x118      [earlier frames - main's variables if any]
0x110      Return address (where to go back in main)  ← P's frame
0x108      Saved %rbp                                 ← Q's frame starts
0x100      Space for local variables                  ← %rsp points here
```

**compute executes its body:**

```assembly
    # long temp1 = x * 2;
    movq %rdi, %rax
    addq %rax, %rax        # temp1 = x * 2 = 10
    movq %rax, -8(%rbp)    # Store temp1 on stack
    
    # long temp2 = y * 3;
    movq %rsi, %rcx
    imulq $3, %rcx         # temp2 = y * 3 = 30
    movq %rcx, -16(%rbp)   # Store temp2 on stack
    
    # Set up call to add
    movq -8(%rbp), %rdi    # First argument: temp1 = 10
    movq -16(%rbp), %rsi   # Second argument: temp2 = 30
    call add               # Push return address, jump to add
```

Now the stack has three frames:
```
Address    Content
0x118      [main's frame]
0x110      Return address to main
0x108      Saved %rbp from compute
0x100      temp1 and temp2
0x0F8      Return address to compute  ← add's frame
           (more frames if add needed them)
```

**add is very simple:**

```assembly
add:
    # No stack frame needed!  All work done in registers
    # a in %rdi (10), b in %rsi (30)
    movq %rdi, %rax
    addq %rsi, %rax        # %rax = 40
    ret                    # Pop return address, jump back to compute
```

add doesn't allocate a stack frame because: 
- It has only 2 arguments (fit in registers)
- It has no local variables that need the stack
- It doesn't call any other functions
- This is called a **leaf procedure**

**add returns:** The `ret` pops the return address and jumps back to compute.

**compute continues:**

```assembly
    # result is in %rax (40)
    addq %rdx, %rax        # result + z = 40 + 15 = 55
    
    # Clean up and return
    movq %rbp, %rsp        # Deallocate local variables
    popq %rbp              # Restore old frame pointer
    ret                    # Return to main
```

**compute returns:** Stack frame deallocated, return to main.

**main continues:** Gets result in %rax (55) and can use it.

---

## Fixed vs.  Variable Size Frames

Most procedures have **fixed-size stack frames**—the amount of space needed is determined at compile time and allocated at the beginning of the procedure. This is efficient because the compiler knows exactly how much space to allocate with a single `subq` instruction.

However, some procedures need **variable-size frames**.  This happens when:
- The procedure uses variable-length arrays (VLAs) whose size isn't known until runtime
- The procedure uses `alloca()` to allocate stack space dynamically

For example: 
```c
void process(int n) {
    int array[n];   // Variable-length array - size depends on n
    // ...  use array
}
```

The amount of stack space needed for `array` isn't known until runtime when the value of `n` is known. Such procedures require special handling, which will be discussed later in Section 3.10.5.

---

## The Minimalist Principle:  Omitting Unnecessary Parts

The text emphasizes that x86-64 procedures are designed for space and time efficiency. They **only allocate the portions of stack frames they actually require**. Many parts shown in Figure 3.25 are optional and can be omitted. 

**Example 1: Six or fewer arguments**

If a procedure has six or fewer integer/pointer arguments, they all fit in registers. The "Argument 7, 8, ...  n" portion of the caller's stack frame can be completely omitted.

```c
long func(long a, long b, long c) {  // Only 3 arguments
    return a + b + c;
}
```

The caller doesn't need to put anything on the stack for arguments—everything goes in registers.

**Example 2: No local variables**

If all local variables fit in registers, the "Local variables" portion can be omitted.

```c
long simple(long x) {
    long y = x * 2;    // y can live in a register
    return y;
}
```

No stack space needed for `y`.

**Example 3: No saved registers**

If the procedure doesn't need to preserve any register values, the "Saved registers" portion can be omitted.

**Example 4: Leaf procedures**

A **leaf procedure** is one that doesn't call any other procedures. It's called a "leaf" in reference to the tree structure of procedure calls—it's at the end of a branch with no further branches. 

Leaf procedures often need **no stack frame at all** if: 
- All arguments fit in registers
- All local variables fit in registers
- No registers need to be saved

Looking back at our examples, the `add` function is a perfect leaf procedure: 

```c
long add(long a, long b) {
    return a + b;
}
```

Assembly: 
```assembly
add:
    movq %rdi, %rax
    addq %rsi, %rax
    ret
```

No stack frame!  Just three instructions. This is extremely efficient.

---
