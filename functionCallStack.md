# Accessing Information
<img width="723" height="824" alt="image" src="https://github.com/user-attachments/assets/8c050164-0447-4c0b-9219-30ea45cd6a2f" />

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
