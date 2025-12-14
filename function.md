# Return Statement
<img width="844" height="801" alt="image" src="https://github.com/user-attachments/assets/2ce211d9-5655-42a6-af10-beb9f5917d5a" />

## How Functions Return

When the program execution reaches the end of a function definition, the function **returns**—meaning control goes back to the place where the function was called, and execution continues from there.

However, you don't always have to wait until the end of the function.  The **return statement** allows you to exit and return from anywhere within the function body. 

## Return Statement Syntax

```c
return expression;
```

The `expression` part is **optional**—you can include it or leave it out depending on your function's purpose.

## Two Types of Functions

### 1. Procedure-Type Functions (No Return Value)

These functions **do not return a value** to the calling program. In most other programming languages, these are called **procedures**. 

**Characteristics:**
- Omit the expression in the return statement
- Should be declared with return type `void`
- Can also return implicitly by reaching the end of the code

**Example:**

```c
void print_message(char *name)
{
    printf("Hello, %s!\n", name);
    // Returns implicitly at the end
}

// Or with explicit return: 
void display_warning(int error_code)
{
    if (error_code == 0)
        return;  // Exit early, no expression needed
    
    printf("Error code: %d\n", error_code);
}
```

### 2. True Functions (Return a Value)

These functions are called from within an expression and **must return a value** that is used in evaluating that expression.

**Characteristics:**
- The return statement **must include** an expression
- Called from within expressions
- The returned value is used in calculations or assignments

**Example:**

```c
int square(int num)
{
    return num * num;  // Returns the calculated value
}

// Using the function:
int result = square(5);  // result gets value 25
int total = square(3) + square(4);  // total gets 9 + 16 = 25
```

## Return Type Matching

**Usually**, the expression you return should match the type that the function was declared to return. 

However, you can return a **different type** if the compiler can convert it to the proper type through automatic (arithmetic) conversions.

**Example:**

```c
double get_average(int a, int b)
{
    return (a + b) / 2.0;  // Returns double as declared
}

int get_sum(double x, double y)
{
    return x + y;  // Returns double, but compiler converts to int
    // Warning: precision may be lost! 
}
```

## Parentheses in Return Statements

Some programmers write return statements like this:

```c
return(x);  // With parentheses
```

**Important note:** The syntax does **not require** parentheses around the expression. You can use them if you prefer, because parentheses are always legal around an expression.  Both styles are correct: 

```c
return x;     // Without parentheses - preferred by many
return (x);   // With parentheses - also valid
```

## Common Pitfalls and Important Warnings

### Both are Called "Functions" in C

In C, both procedure-type subprograms (void functions) and true functions (value-returning functions) are called **functions**. This is different from some other languages that distinguish between "functions" and "procedures."

### Pitfall #1: Discarding Return Values (Usually OK)

It's **possible** to call a true function (one that returns a value) without using the value in any expression. The returned value is simply discarded.

**Example:**

```c
int calculate_total(int a, int b)
{
    return a + b;
}

// Called but value ignored:
calculate_total(5, 10);  // Legal but wasteful - return value discarded

// Proper use:
int sum = calculate_total(5, 10);  // Value is used
```

While this is legal, it's generally pointless unless the function has side effects (like printing or modifying global variables).

### Pitfall #2: Using Void Function in Expression (SERIOUS ERROR!)

Calling a **procedure-type function** (void function) from within an expression is a **serious error**. Since the function doesn't return a value, an unpredictable **garbage value** will be used in evaluating the expression. 

**Example of the ERROR:**

```c
void print_number(int num)
{
    printf("%d\n", num);
    // Returns nothing (void)
}

// SERIOUS ERROR: 
int result = print_number(5) + 10;  // ERROR!  Garbage value used!
int x = print_number(3);             // ERROR! x gets garbage value! 
```

**Good news:** Modern compilers usually **catch this error** because they enforce stricter rules about function return types than older compilers did.  You'll typically get a compile-time error or warning. 
6. You can safely **discard** return values from true functions
7. **Never** use void functions in expressions - this causes unpredictable garbage values
8. Modern compilers help catch return-type errors

# Function Declaration

<img width="874" height="303" alt="image" src="https://github.com/user-attachments/assets/1fd59d0e-0a7d-4e2b-b4ef-0091b3e7976a" />

## The Compiler's Challenge

When the compiler encounters a call to a function, it needs to generate code to: 
1. **Pass the arguments** to the function
2. **Call the function** itself
3. **Receive the value** (if any) that the function sends back

But here's the critical question: **How does the compiler know what kinds of arguments (and how many) the function expects, and what kind of value (if any) the function returns?**

## What Happens Without Specific Information

If there **isn't any specific information** given about the function, the compiler makes **dangerous assumptions**:

### Assumption #1: Arguments Are Correct
The compiler **assumes** that the call has the correct number and types of arguments. It trusts you completely without checking! 

### Assumption #2: Returns an Integer
The compiler **assumes** that the function will return an **integer**, which usually leads to **errors** for functions that return non-integral types (like `float`, `double`, `char*`, etc.).

## Example: The Problem Without Function Information

```c
#include <stdio.h>

int main()
{
    // Compiler has NO information about square() function yet
    int result = square(5);  
    // Compiler assumes:  square returns int, takes whatever arguments given
    
    printf("Result: %d\n", result);
    return 0;
}

// Actual function defined later
double square(double num)  // Actually returns double!
{
    return num * num;
}
```

**What goes wrong:**
- The compiler assumed `square()` returns `int`
- But it actually returns `double`
- This mismatch causes **incorrect results** or **garbage values**

## Example: Wrong Number of Arguments

```c
#include <stdio.h>

int main()
{
    // Compiler has no info about add() function
    int sum = add(5);  // Called with 1 argument
    // Compiler assumes this is correct! 
    
    printf("Sum:  %d\n", sum);
    return 0;
}

// Actual function expects 2 arguments! 
int add(int a, int b)
{
    return a + b;  // What value is b?  Garbage!
}
```

**What goes wrong:**
- The function expects **2 arguments** (`a` and `b`)
- But we called it with only **1 argument**
- The second parameter `b` gets a **garbage value** from memory
- The compiler didn't warn us because it had no information! 

## Example: Wrong Return Type

```c
#include <stdio.h>

int main()
{
    // Compiler assumes get_pi() returns int
    int value = get_pi();
    printf("Pi as int: %d\n", value);  // Wrong! 
    
    // Trying to use it as double
    double pi = get_pi();
    printf("Pi: %f\n", pi);  // Still wrong!  Compiler already assumed int
    
    return 0;
}

// Actually returns double
double get_pi()
{
    return 3.14159;
}
```

**What goes wrong:**
- Compiler assumed `get_pi()` returns `int` 
- Actually returns `double`
- The value gets **incorrectly interpreted** as an integer
- Results in **completely wrong values** being used

## Visual Example: The Communication Problem

Think of it like ordering at a restaurant without a menu:

**Without function information (no menu):**
```
You:  "I'll have the special!"
Waiter: "Okay..." (assumes you want meal #1, assumes it costs $10)
Kitchen: Actually sends meal #3, costs $25
Result: Wrong meal, wrong bill, confusion! 
```

**With function information (with menu):**
```
You: "I'll have the special!"
Waiter: (checks menu) "Which special? We have 3. They're $15, $20, and $25."
You: "The $20 one, please."
Kitchen: Sends correct meal
Result: Everyone knows what to expect!
```

## Real-World Consequences

### Problem 1: Incorrect Integer Assumption

```c
#include <stdio.h>

int main()
{
    // Compiler assumes returns int
    char *message = get_message();  
    printf("%s\n", message);  // Crash or garbage!
    
    return 0;
}

char *get_message()  // Returns pointer, NOT int! 
{
    return "Hello, World!";
}
```

**Result:** The pointer value gets corrupted because the compiler treated it as an integer.  This often causes **crashes** or **security vulnerabilities**.

### Problem 2: Missing Argument Goes Undetected

```c
#include <stdio.h>

int main()
{
    // Should pass 3 arguments, only passing 2
    int result = calculate(10, 20);  // Compiler says:  "Looks fine to me!"
    printf("Result:  %d\n", result);
    
    return 0;
}

int calculate(int a, int b, int c)  // Expects 3 arguments!
{
    return a + b + c;  // c has garbage value!
}
```

**Result:** The third parameter contains **unpredictable garbage**, leading to **wrong calculations** and **hard-to-find bugs**.

### Problem 3: Type Mismatch

```c
#include <stdio.h>

int main()
{
    // Passing int, but function expects double
    int result = compute(5);  // Compiler assumes this is OK
    printf("Result: %d\n", result);
    
    return 0;
}

double compute(double x)  // Expects double
{
    return x * 2.5;
}
```

**Result:** Argument type mismatch may cause **incorrect conversions** and **wrong results**.

## Why This Matters

These assumptions by the compiler lead to: 

1. **Runtime errors** - Programs crash unexpectedly
2. **Wrong results** - Calculations produce incorrect values
3. **Security vulnerabilities** - Memory corruption and exploits
4. **Hard-to-debug problems** - The error happens far from its cause
5. **Unpredictable behavior** - Works on one system, fails on another

# Prototypes

## Giving the Compiler Information About Functions

## Two Ways to Inform the Compiler

It's safer to give the compiler **specific information** about functions.  There are **two different ways** to do this:

### Method 1: Define the Function Earlier in the Same File

If the **definition** for the function appears **earlier** in the same source file (before it's called), the compiler will remember:
- The **number** of arguments
- The **types** of arguments  
- The **type** of the return value

The compiler can then **check all subsequent calls** to the function (in that source file) to make sure they are correct.

**Example:**

```c
#include <stdio.h>

// Function DEFINED first
double get_pi()
{
    return 3.14159;
}

int main()
{
    // Now compiler knows get_pi() returns double
    double pi = get_pi();  // ✓ Correct! 
    printf("Pi: %f\n", pi);
    
    // If we make a mistake:
    // int x = get_pi(5);  // ✗ Compiler catches error:  wrong number of arguments! 
    
    return 0;
}
```

#### Important Warning About Old-Style Syntax

If a function is defined using the **old K&R syntax** (with a separate list for argument types), then the compiler remembers **only the return value type**. **No information is saved** on the number or types of the arguments.

**Old style (dangerous):**
```c
int *find_int(key, array, len)  // Old K&R style
int key;
int array[];
int len;
{
    // function body
}
// Compiler only remembers return type (int*), NOT argument info!
```

**Because of this limitation, it is important to use the new function declaration style whenever possible.**

---

### Method 2: Use a Function Prototype (Better!)

A **function prototype** (introduced in Chapter 1) summarizes the function declaration, giving the compiler **complete information** on how the function should be called.

## What is a Function Prototype?

Here's a prototype for the `find_int` function:

```c
int *find_int(int key, int array[], int len);
```

**Note the semicolon** at the end—it **distinguishes** a prototype from the beginning of a function definition.

**The prototype tells the compiler:**
1. The **number** of arguments
2. The **type** of each argument  
3. The **type** of the returned value

After the prototype has been seen, the compiler will:
- **Check calls** to the function to ensure arguments are correct
- **Verify** that the returned value is used properly
- **Convert values** to the correct type where possible (if there are mismatches)

**Example:**

```c
#include <stdio.h>

// Function prototype
double get_pi(void);
int add(int a, int b);

int main()
{
    double pi = get_pi();  // ✓ Compiler checks this is correct
    printf("Pi: %f\n", pi);
    
    int sum = add(5, 10);  // ✓ Correct call
    printf("Sum: %d\n", sum);
    
    // int wrong = add(5);  // ✗ Error: too few arguments
    // int bad = add(5, 10, 15);  // ✗ Error: too many arguments
    
    return 0;
}

// Function definitions
double get_pi(void)
{
    return 3.14159;
}

int add(int a, int b)
{
    return a + b;
}
```

## Include Parameter Names in Prototypes

While **not required**, it is wise to **include descriptive parameter names** in function prototypes because they give useful information to clients calling the function.

**Which prototype is more useful?**

```c
// Version 1: No parameter names (confusing)
char *strcpy(char *, char *);

// Version 2: With parameter names (clear!)
char *strcpy(char *destination, char *source);
```

**Answer:** Version 2!  Now you know which argument is the destination and which is the source. 

---

## Dangerous Way to Use Prototypes ⚠️

The following code illustrates a **dangerous way** to use function prototypes:

```c
void a()
{
    int *func(int *value, int len);  // Prototype inside function a
    // ...  use func
}

void b()
{
    int func(int len, int *value);  // DIFFERENT prototype inside function b! 
    // ... use func
}
```

**Look closely:** The prototypes are **different**! 
- Arguments are **reversed**
- Return values are **different types** (`int *` vs `int`)

**Why doesn't the compiler catch this error?**

Each prototype is written **inside the body of a function**. They have **block scope**, so: 
1. The compiler throws out what it learned from the first prototype at the end of function `a()`
2. When it sees the second prototype in function `b()`, it has no memory of the first one
3. The scopes **don't overlap**, so the compiler **never detects the mismatch**

One or both prototypes are **wrong**, but the compiler never sees the contradiction—**no error messages** are produced!  This leads to **serious runtime bugs**. 

---

## The Preferred Way:  Use Header Files ✓

```c
// File: func.h
#ifndef FUNC_H
#define FUNC_H

int *func(int *value, int len);  // Prototype in header file

#endif
```

```c
// File: main.c
#include "func.h"  // Include the prototype

void a()
{
    int array[] = {1, 2, 3};
    int *result = func(array, 3);  // Compiler checks this call
    // ... 
}

void b()
{
    int data[] = {4, 5, 6};
    int *ptr = func(data, 3);  // Compiler checks this call too
    // ...
}
```

```c
// File: func.c
#include "func.h"  // Include prototype here too! 

int *func(int *value, int len)  // Definition matches prototype
{
    // function implementation
    return value;
}
```

## Why This Technique is Better

### 1. File Scope
The prototype now has **file scope**, so **one copy** applies to the entire source file.  This is easier than writing a separate copy everywhere the function is called.

#### Explaining File Scope with Example

When you put a prototype **inside a function**, it only works within that function (block scope). When you put it **at the top of the file** or in a header file, it works for the **entire file** (file scope).

## Without File Scope (Bad - Writing Multiple Copies)

```c
void a()
{
    int *func(int *value, int len);  // Prototype written here
    int array[] = {1, 2, 3};
    int *result = func(array, 3);
}

void b()
{
    int *func(int *value, int len);  // Same prototype written AGAIN here
    int data[] = {4, 5, 6};
    int *ptr = func(data, 3);
}

void c()
{
    int *func(int *value, int len);  // And AGAIN here! 
    int nums[] = {7, 8, 9};
    int *p = func(nums, 3);
}
```

Notice the prototype `int *func(int *value, int len);` is **written 3 times**—once inside each function. This is tedious and error-prone (you might type it wrong!).

## With File Scope (Good - Write Once, Use Everywhere)

```c
#include "func.h"  // Contains:  int *func(int *value, int len);

// Now ALL functions in this file can use func()

void a()
{
    // No prototype needed here - already available! 
    int array[] = {1, 2, 3};
    int *result = func(array, 3);
}

void b()
{
    // No prototype needed here either!
    int data[] = {4, 5, 6};
    int *ptr = func(data, 3);
}

void c()
{
    // Still no prototype needed! 
    int nums[] = {7, 8, 9};
    int *p = func(nums, 3);
}
```

By including the header file once at the top, the prototype has **file scope**—it applies to the **entire source file**. All functions (`a`, `b`, and `c`) can call `func()` without needing to write the prototype again.  You write it **once**, and it works **everywhere** in that file. 

### 2. No Duplicate Typing
The prototype is only written **once**, so there's **no chance for disagreements** between multiple copies. 

### 3. Easy Maintenance
If the function definition changes: 
- Modify the prototype **once** in the header file
- Recompile each source file that includes it
- All uses are automatically updated! 

### 4. Compiler Verification
If the prototype is **also #included** in the file where the function is defined, the compiler can **verify that the prototype matches the definition**.

**Example showing the benefit:**

```c
// File: math_ops.h
double calculate(double x, double y);

// File: math_ops.c
#include "math_ops.h"

// If we accidentally write this: 
int calculate(int x, int y)  // ✗ Compiler error:  conflicts with prototype!
{
    return x + y;
}
```

The compiler catches the mismatch immediately!

---

## Special Case: Functions Without Arguments

Consider this declaration—it looks **ambiguous**:

```c
int *func();
```

**Question:** Is this: 
- An **old-style declaration** (giving only the return type)?
- A **new-style prototype** for a function with no arguments? 

**Answer:** To maintain compatibility with pre-ANSI programs, this declaration **must be interpreted as an old-style declaration**. 

### Correct Way to Declare a Function With No Arguments

A prototype for a function **without arguments** is written like this: 

```c
int *func(void);
```

The keyword `void` indicates that there **aren't any arguments** (not that there is one argument of type `void`).

**Example:**

```c
#include <stdio.h>

// Correct prototypes: 
int get_random(void);     // No arguments
void print_header(void);  // No arguments, no return value

int main()
{
    print_header();
    int num = get_random();
    printf("Random:  %d\n", num);
    
    // int wrong = get_random(5);  // ✗ Error:  too many arguments!
    
    return 0;
}

void print_header(void)
{
    printf("=== My Program ===\n");
}

int get_random(void)
{
    return 42;  // Very random!  : )
}
```
# Default Function Assumptions
## Why Prototyping Non-Integer Functions is Critical

While it's recommended that **all functions be prototyped**, it is **especially important** to prototype functions that return **non-integral values** (like `float`, `double`, pointers, etc.).

## Understanding the Problem:  Values Don't Know Their Own Type

Remember that **the type of a value is not inherent in the value itself**, but rather **in the way that it is used**. 

Think of it like this:  The binary bits `10110011` could represent:
- The integer `179`
- Part of a floating-point number
- A character
- Part of a pointer address

The **same bits** mean different things depending on how you interpret them!

## What Goes Wrong Without a Prototype

If the compiler **assumes** that a function returns an integral value, it will generate **integer instructions** to manipulate the value. If the value is actually a **non-integral type** (like floating-point), the result will usually be **incorrect**.

## Real Example: The Floating-Point Disaster

Let's look at a concrete example of this error. 

### The Function

Imagine a function `xyz()` that returns the `float` value `3.14`:

```c
// Function definition (somewhere in the code)
float xyz()
{
    return 3.14;
}
```

On a Sun Sparc workstation, the bits used to represent this floating-point number `3.14` are:

```
01000000010010001111010111000011
```

These 32 bits represent `3.14` in **IEEE 754 floating-point format**.

### The Problem:  Calling Without a Prototype

Now assume the function is called like this, **without a prototype**:

```c
float f;
...
f = xyz();  // NO PROTOTYPE for xyz()!
```

### What the Compiler Does (Step-by-Step)

**Step 1:** The compiler sees the call to `xyz()` but has **no prototype** telling it what type `xyz()` returns.

**Step 2:** The compiler **assumes** `xyz()` returns an **integer** (the default assumption).

**Step 3:** Since `f` is declared as `float`, the compiler thinks:  "I need to convert an integer to float." So it generates **integer-to-float conversion instructions**.

**Step 4:** The function `xyz()` executes and returns these bits: 
```
01000000010010001111010111000011
```

These bits represent the float `3.14`.

**Step 5:** The conversion instructions receive these bits but **interpret them as an integer**. When you interpret these bits as a 32-bit integer, you get: 

```
1,078,523,331
```

**Step 6:** The compiler's conversion instructions then convert the integer `1,078,523,331` to floating-point format, which gives approximately: 

```
1078523331.0
```

**Step 7:** This completely wrong value (`1078523331.0` instead of `3.14`) is stored in `f`.

## Visual Breakdown

```
Function xyz() returns:
    Bits: 01000000010010001111010111000011
    Meaning: 3.14 (as float)
             ↓
Without prototype, compiler thinks these are INTEGER bits
    Bits: 01000000010010001111010111000011  
    Meaning: 1,078,523,331 (as integer)
             ↓
Compiler converts "integer" 1,078,523,331 to float
    Result: 1078523331.0
             ↓
Stored in f:  1078523331.0  ← COMPLETELY WRONG! 

Expected: 3.14
Got: 1078523331.0
```

## Why Did This Happen?

**Question:** Why was this conversion done when the value returned was **already in floating-point format**? 

**Answer:** The compiler has **no way of knowing** it was already floating-point, because there was **no prototype or declaration to tell it so**.

The compiler saw: 
- "I'm getting some bits from `xyz()`"
- "I don't have a prototype, so I'll assume it's an integer"
- "But `f` is a float, so I need to convert integer → float"
- "Let me interpret these bits as an integer and convert them"

And that's where everything went wrong!

## The Correct Way:  With a Prototype

```c
#include <stdio.h>

// PROTOTYPE tells compiler: xyz() returns float
float xyz(void);

int main()
{
    float f;
    f = xyz();  // Compiler knows xyz() returns float - no conversion! 
    
    printf("Value: %f\n", f);  // Correctly prints:  3.14
    
    return 0;
}

float xyz(void)
{
    return 3.14;
}
```

With the prototype `float xyz(void);`, the compiler knows:
1. `xyz()` returns a **float**
2. The bits coming back are **already in float format**
3. **No conversion needed** - just store them in `f` as-is
4. Result: `f` correctly contains `3.14`

## Another Example: Pointer Functions

The problem is even worse with pointer functions: 

```c
#include <stdio.h>
#include <string.h>

// NO PROTOTYPE - DANGER! 

int main()
{
    char *message;
    message = get_greeting();  // Compiler assumes returns int! 
    
    printf("%s\n", message);  // CRASH or garbage! 
    
    return 0;
}

char *get_greeting()  // Actually returns pointer
{
    return "Hello, World!";
}
```

**What happens:**
1. `get_greeting()` returns a **pointer** (memory address)
2. Without a prototype, compiler thinks it returns an **int**
3. The pointer bits get **misinterpreted as an integer**
4. On 64-bit systems, pointers are 64 bits but compiler expects 32-bit int
5. **Half the pointer bits are lost! **
6. When you try to use `message`, you're using a **corrupted pointer**
7. Result: **Crash**, **garbage output**, or **security vulnerability**

### With Prototype (Correct):

```c
#include <stdio.h>

char *get_greeting(void);  // PROTOTYPE - tells compiler it returns char*

int main()
{
    char *message;
    message = get_greeting();  // Compiler knows it's a pointer!
    
    printf("%s\n", message);  // Works correctly:  "Hello, World!"
    
    return 0;
}

char *get_greeting(void)
{
    return "Hello, World!";
}
```

## Real-World Demonstration

Here's a complete program showing the problem:

```c
#include <stdio.h>

// Uncomment the prototype to fix the problem: 
// float calculate_pi(void);

int main()
{
    float pi;
    pi = calculate_pi();  // Without prototype:  disaster!
    
    printf("Pi value: %f\n", pi);
    printf("Expected: 3.141593\n");
    printf("Difference: %f\n", pi - 3.141593);
    
    return 0;
}

float calculate_pi(void)
{
    return 3.141593;
}
```

**Without prototype:**
```
Pi value: 1074340347.000000
Expected: 3.141593
Difference: 1074340343.858407
```

**With prototype uncommented:**
```
Pi value: 3.141593
Expected: 3.141593
Difference: 0.000000
```

## Why This Matters

This example illustrates why it is **vital** for functions that return values other than integers to have prototypes.  Without them: 

- **Float/double functions** return garbage values (off by billions!)
- **Pointer functions** return corrupted addresses (crashes and security holes)
- **Long/short functions** may get truncated or sign-extended incorrectly
- The compiler has **no way to generate the correct instructions**

The bits are correct when they leave the function, but they get **completely mangled** by the incorrect conversion instructions the compiler generates when it doesn't know the real return type.

Always prototype your functions, especially if they return anything other than `int`! 

# Function Arguments

## Understanding Function Arguments: Call by Value and Call by Reference

## The Basic Rule:  Call by Value

All arguments to C functions are passed with a technique known as **call by value**, which means that the function gets a **copy** of the argument value.  Thus the function may modify its parameters **without fear of affecting** the values of the arguments passed from the calling program.  This behavior is the same as value (not var) parameters in Modula and Pascal.

**The rule in C is simple:  all arguments are passed by value.**

## Example: Function Can Safely Destroy Its Parameters

Let's look at Program 7.2, which checks whether a number has **even parity** (an even number of 1-bits):

```c
/*
** Check the value for even parity.
*/
int even_parity(int value, int n_bits)
{
    int parity = 0;
    
    /*
    ** Count the number of 1-bits in the value. 
    */
    while(n_bits > 0) {
        parity += value & 1;  // Check rightmost bit
        value >>= 1;          // Shift value right
        n_bits -= 1;          // Decrease counter
    }
    
    /*
    ** Return TRUE if the low order bit of the count is zero
    ** (which means that there were an even number of 1's).
    */
    return (parity % 2) == 0;
}
```

**How it works:**
- The function shifts `value` right bit by bit, checking each bit
- It adds each rightmost bit to `parity` to count the 1-bits
- After the loop, it checks if the count is even

**The interesting feature:** This function **destroys both of its arguments** as the work progresses. The original `value` is shifted to nothing, and `n_bits` is decremented to zero. 

**Why this works fine:** With call-by-value, the arguments are **copies** of the caller's values. Destroying the copies does **not affect** the original values.

**Example usage:**

```c
int main()
{
    int number = 15;      // Binary: 1111 (four 1-bits)
    int bits = 4;
    
    int result = even_parity(number, bits);
    
    // After the function call: 
    printf("Number: %d\n", number);  // Still 15 - unchanged! 
    printf("Bits: %d\n", bits);      // Still 4 - unchanged! 
    printf("Even parity: %d\n", result);  // 1 (true - four 1's is even)
    
    return 0;
}
```

Even though the function modified `value` and `n_bits` internally, the original variables `number` and `bits` in `main()` remain **unchanged** because the function only received **copies**. 

## When Call by Value Doesn't Work:  The Swap Problem

Program 7.3a shows a function that **tries** to exchange two values but **doesn't work**:

```c
/*
** Exchange two integers in the calling program (doesn't work!)
*/
void swap(int x, int y)
{
    int temp;
    
    temp = x;
    x = y;
    y = temp;
}
```

**Trying to use it:**

```c
int main()
{
    int a = 10;
    int b = 20;
    
    printf("Before:  a=%d, b=%d\n", a, b);
    
    swap(a, b);  // Try to swap
    
    printf("After: a=%d, b=%d\n", a, b);  // Still a=10, b=20! 
    
    return 0;
}
```

**Output:**
```
Before: a=10, b=20
After:  a=10, b=20
```

**Why it doesn't work:** The function wants to modify the caller's arguments, but it **can't** because all it exchanges are the **copies** of the values that were sent to the function.  The original values are **untouched**.

**What actually happened inside the function:**
1.  Copies are made:  `x = 10` (copy of `a`), `y = 20` (copy of `b`)
2. The copies are swapped: `x = 20`, `y = 10`
3. The function ends, the copies are discarded
4. Original `a` and `b` remain unchanged! 

## The Solution: Pass Pointers (Simulating Call by Reference)

To access the caller's values, you must pass **pointers** to the locations you wish to modify. The function must then use **indirection** to follow the pointers and modify the desired locations.

Program 7.3b uses this technique:

```c
/*
** Exchange two integers in the calling program. 
*/
void swap(int *x, int *y)  // Parameters are POINTERS
{
    int temp;
    
    temp = *x;   // Get value at address x
    *x = *y;     // Put value from y into location x
    *y = temp;   // Put saved value into location y
}
```

Because the function expects **pointers** as arguments, we call it like this:

```c
int main()
{
    int a = 10;
    int b = 20;
    
    printf("Before: a=%d, b=%d\n", a, b);
    
    swap(&a, &b);  // Pass ADDRESSES of a and b
    
    printf("After: a=%d, b=%d\n", a, b);  // Now a=20, b=10! 
    
    return 0;
}
```

**Output:**
```
Before: a=10, b=20
After:  a=20, b=10
```

**Why it works now:**
1. We pass `&a` and `&b` (the **addresses** of `a` and `b`)
2. The function receives **copies of the addresses** (pointers)
3. Using `*x` and `*y`, the function follows the pointers to access the **original** variables
4. The function modifies the **original** `a` and `b` through the pointers
5. Changes are visible in the calling program! 

## Arrays:  The Special Case

However, if an **array name** is passed as an argument and a subscript is used on the argument in the function, then modifying array elements in the function **actually changes** the elements of the array in the calling program.  The function accesses the **very same array** that exists in the calling program; the array is **not copied**. This behavior is termed **call by reference** and is how var parameters are implemented in many other languages. 

Program 7.4 demonstrates this:

```c
/*
** Set all of the elements of an array to zero.
*/
void clear_array(int array[], int n_elements)
{
    /*
    ** Clear the elements of the array starting with the last
    ** and working towards the first. Note the predecrement
    ** avoids going off the end of the array. 
    */
    while(n_elements > 0)
        array[--n_elements] = 0;
}
```

**Using the function:**

```c
int main()
{
    int numbers[] = {5, 10, 15, 20, 25};
    int size = 5;
    
    printf("Before:\n");
    for(int i = 0; i < size; i++)
        printf("%d ", numbers[i]);
    printf("\n");
    
    clear_array(numbers, size);  // Pass array
    
    printf("After:\n");
    for(int i = 0; i < size; i++)
        printf("%d ", numbers[i]);
    printf("\n");
    
    printf("Size variable: %d\n", size);  // Still 5! 
    
    return 0;
}
```

**Output:**
```
Before:
5 10 15 20 25
After:
0 0 0 0 0
Size variable: 5
```

**What happened:**
- `n_elements` is a **scalar**, so it's passed by value—modifying it in the function does **not** affect the `size` variable in `main()`
- The array `numbers` is modified—the function **does indeed** set the elements of the calling program's array to zero

**Why arrays behave differently:** The value of the array name is really a **pointer**, and a copy of the pointer is passed to the function. A subscript is really a form of **indirection**, and applying indirection to the pointer accesses the locations that it points to.  The argument (the pointer) is indeed a **copy**, but the indirection uses the copy to access the **original array values**.

## Visual Explanation:  Scalars vs. Arrays

**Scalars (call by value):**
```
Calling program:        Function:
   a = 10    ------>    x = 10 (copy)
                        x = 20 (modify copy)
   a = 10    <------    (original unchanged)
```

**Arrays (behave like call by reference):**
```
Calling program:              Function:
   array = [5,10,15]  ------>  array (pointer to same memory)
   Memory:  [5,10,15]           array[0] = 0
   Memory: [0,10,15]  <------  (modifies original memory!)
   Memory: [0, 0, 0]           array[1] = 0, array[2] = 0
```

## Array Parameters Without Size

This example also illustrates another feature. It is **legal to declare array parameters without specifying a size** because memory is **not allocated** for the array elements in the function; the indirection causes the array elements in the calling program to be accessed instead. 

```c
void process_array(int array[])  // No size specified! 
{
    // Can work with array of any size
}
```

Thus, a **single function can access arrays of any size**, which should excite Pascal programmers! 

**However**, there isn't any way for the function to figure out the **actual size** of an array argument, so this information must also be passed explicitly if it is needed.

**Example:**

```c
void print_array(int arr[], int size)  // Size passed separately
{
    for(int i = 0; i < size; i++)
        printf("%d ", arr[i]);
    printf("\n");
}

int main()
{
    int small[] = {1, 2, 3};
    int large[] = {10, 20, 30, 40, 50, 60};
    
    print_array(small, 3);  // Same function, different sizes! 
    print_array(large, 6);
    
    return 0;
}
```

## Two Simple Rules to Remember

1. **Scalar arguments** to a function are passed **by value** (function gets a copy).
2. **Array arguments** to a function behave as though they are passed **by reference** (function accesses the original).

## Old K&R Style Warning

Recall that in K&R C, function parameters were declared like this:

```c
int func(a, b, c)
int a;
char b;
float c;
{
    // ...
}
```

Another reason to **avoid this style** is that K&R compilers handled arguments differently: `char` and `short` arguments were promoted to `int` before being passed, and `float` arguments were promoted to `double`. These conversions are called the **default argument promotions**, and because of them you will frequently see function parameters in pre-ANSI programs declared as `int` when in fact `char` values are passed.

**CAUTION!** To maintain compatibility, ANSI compilers also perform these conversions for functions declared in the old style. They are **not done** on functions that have been prototyped, though, so **mixing the two styles can lead to errors**.

Short explanation:
- Default argument promotions: when a function is called without a prototype, float arguments are promoted to double, and char/short are promoted to int before being passed.
- That can break things when the called function actually expects a float (or char) because the caller and callee disagree about what was passed.

Concrete examples

**1) Buggy (no prototype — promotions happen)**
```c
#include <stdio.h>

int main(void) {
    print_float(3.14f); // no prototype in scope -> 3.14f is promoted to double
    return 0;
}

void print_float(float x) {        // callee expects a float
    printf("x = %f\n", x);        // may print garbage on some systems
}
```
What goes wrong: the caller passes a double (because of promotion) but the function expects a float — the bit-level calling convention can differ, so the value received inside print_float can be wrong.

**2) Correct (with prototype)**
```c
#include <stdio.h>

void print_float(float x);        // prototype — prevents promotion

int main(void) {
    print_float(3.14f);           // 3.14f stays a float when passed
    return 0;
}

void print_float(float x) {
    printf("x = %f\n", x);        // prints 3.140000 as expected
}
```

Short note about char/short:
- If you call a function without a prototype and pass a char, it will be promoted to int. Usually this is harmless, but mixing styles (prototype vs old-style/no-prototype) can still cause confusion or bugs.

Takeaway: always use prototypes (or declare the function before calling) so the compiler knows the exact parameter types and no unexpected promotions occur.

# Basics of Functions
## Building a Pattern-Matching Program: A Practical Example

## The Problem

Let's design and write a program to print each line of its input that contains a particular **"pattern"** or string of characters. This is a special case of the UNIX program called `grep`.

**Example:** Searching for the pattern **"ould"** in these lines of poetry:

```
Ah Love! could you and I with Fate conspire
To grasp this sorry Scheme of Things entire,
Would not we shatter it to bits -- and then
Re-mould it nearer to the Heart's Desire! 
```

Should produce this output (only lines containing "ould"):

```
Ah Love! could you and I with Fate conspire
Would not we shatter it to bits -- and then
Re-mould it nearer to the Heart's Desire!
```

Notice that "could", "Would", and "Re-mould" all contain the pattern "ould". 

## Breaking Down the Problem

The job falls neatly into **three pieces**:

```
while (there's another line)
    if (the line contains the pattern)
        print it
```

Although it's certainly possible to put the code for all of this in `main`, a **better way** is to use the structure to advantage by making each part a **separate function**. Three small pieces are better to deal with than one big one, because: 

- **Irrelevant details** can be buried in the functions
- The chance of **unwanted interactions** is minimized  
- The pieces may even be **useful in other programs**

## What We Need

Let's look at what functions we need for each piece:

1. **"While there's another line"** → We need `getline`, a function to read one line of input
2. **"Print it"** → We already have `printf`, which someone has provided for us
3. **"The line contains the pattern"** → We need to write this! 

This means we need only write a routine to decide whether the line contains an occurrence of the pattern. 

## Solving the Pattern Search Problem

We can solve that problem by writing a function `strindex(s, t)` that returns the **position or index** in the string `s` where the string `t` begins, or **-1** if `s` does not contain `t`.

**Why -1 for failure?** Because C arrays begin at position zero, indexes will be zero or positive, and so a negative value like -1 is convenient for signaling failure. 

**Example of how `strindex` works:**

```c
strindex("could you", "ould")  → returns 1 (found at position 1)
strindex("hello world", "wor")  → returns 6 (found at position 6)
strindex("hello world", "xyz")  → returns -1 (not found)
```

**Benefit:** When we later need more sophisticated pattern matching, we only have to **replace strindex**; the rest of the code can remain the same.

Note: The standard library provides a function `strstr` that is similar to `strindex`, except that it returns a **pointer** instead of an **index**.

## The Complete Program

Given this much design, filling in the details of the program is straightforward. Here is the whole thing, so you can see how the pieces fit together: 

```c
#include <stdio.h>

#define MAXLINE 1000  /* maximum input line length */

int getline(char line[], int max);
int strindex(char source[], char searchfor[]);

char pattern[] = "ould";  /* pattern to search for */

/* find all lines matching pattern */
main()
{
    char line[MAXLINE];
    int found = 0;
    
    while (getline(line, MAXLINE) > 0)
        if (strindex(line, pattern) >= 0) {
            printf("%s", line);
            found++;
        }
    return found;
}

/* getline: get line into s, return length */
int getline(char s[], int lim)
{
    int c, i;
    
    i = 0;
    while (--lim > 0 && (c=getchar()) != EOF && c != '\n')
        s[i++] = c;
    if (c == '\n')
        s[i++] = c;
    s[i] = '\0';
    return i;
}

/* strindex: return index of t in s, -1 if none */
int strindex(char s[], char t[])
{
    int i, j, k;
    
    for (i = 0; s[i] != '\0'; i++) {
        for (j=i, k=0; t[k]!='\0' && s[j]==t[k]; j++, k++)
            ;
        if (k > 0 && t[k] == '\0')
            return i;
    }
    return -1;
}
```

## Understanding Each Function

### The `main` Function

```c
main()
{
    char line[MAXLINE];
    int found = 0;
    
    while (getline(line, MAXLINE) > 0)
        if (strindex(line, pattern) >= 0) {
            printf("%s", line);
            found++;
        }
    return found;
}
```

**What it does:**
1. Declares a character array `line` to hold each input line
2. Keeps a counter `found` to track how many matching lines we find
3. **Loop:** Reads lines one at a time using `getline`
4. **Check:** Uses `strindex` to see if the pattern exists in the line (returns >= 0 if found)
5. **Print:** If pattern found, prints the line and increments counter
6. **Returns:** The total number of matches found (useful for the calling environment)

### The `getline` Function

```c
int getline(char s[], int lim)
{
    int c, i;
    
    i = 0;
    while (--lim > 0 && (c=getchar()) != EOF && c != '\n')
        s[i++] = c;
    if (c == '\n')
        s[i++] = c;
    s[i] = '\0';
    return i;
}
```

**What it does:**
1. Reads characters one at a time using `getchar()`
2. Stops when it encounters: 
   - End of file (`EOF`)
   - A newline character (`'\n'`)
   - The limit is reached (`lim`)
3. If it stopped at a newline, includes the newline in the string
4. Adds the null terminator `'\0'` at the end
5. Returns the length of the line read

**Example:**
```
Input:  "hello\n"
Result: s contains "hello\n\0"
Returns: 6
```

### The `strindex` Function

```c
int strindex(char s[], char t[])
{
    int i, j, k;
    
    for (i = 0; s[i] != '\0'; i++) {
        for (j=i, k=0; t[k]!='\0' && s[j]==t[k]; j++, k++)
            ;
        if (k > 0 && t[k] == '\0')
            return i;
    }
    return -1;
}
```

**What it does:** Searches for string `t` inside string `s`.

**How it works step-by-step:**

Let's trace through with `s = "could"` and `t = "ould"`:

```
i=0: Check position 0
     s[0]='c', t[0]='o' → Don't match, continue

i=1: Check position 1
     s[1]='o', t[0]='o' → Match! Continue inner loop
     s[2]='u', t[1]='u' → Match! 
     s[3]='l', t[2]='l' → Match!
     s[4]='d', t[3]='d' → Match!
     t[4]='\0' → Reached end of t, found complete match! 
     Return i=1
```

**The inner loop logic:**
- `j` tracks position in `s` (starts at current `i`)
- `k` tracks position in `t` (starts at 0)
- Continue while characters match and haven't reached end of `t`
- After loop: if `k > 0` (matched at least one char) AND `t[k] == '\0'` (reached end of `t`), we found a complete match! 

## Understanding Function Structure

Each function definition has this form: 

```c
return-type function-name(argument declarations)
{
    declarations and statements
}
```

Various parts may be absent.  A minimal function is:

```c
dummy() {}
```

This does nothing and returns nothing. A do-nothing function like this is sometimes useful as a **place holder** during program development. 

**Note:** If the return type is omitted, `int` is assumed.

## How the Program Works as a Whole

A program is just a set of definitions of variables and functions. Communication between the functions is by: 
- **Arguments** passed to functions
- **Values returned** by the functions  
- **External variables** (like `pattern` in our example)

The functions can occur in **any order** in the source file, and the source program can be split into multiple files, so long as no function is split.

## The Return Statement

The `return` statement is the mechanism for returning a value from the called function to its caller. Any expression can follow return:

```c
return expression;
```

The expression will be converted to the return type of the function if necessary. Parentheses are often used around the expression, but they are optional: 

```c
return 5;           // Returns 5
return (x + y);     // Returns sum (parentheses optional)
return x > y ?  x : y;  // Returns larger value
```

**Important points about return:**

1. The calling function is **free to ignore** the returned value
2. There need not be an expression after `return`:
   ```c
   return;  // Returns no value
   ```
3. Control also returns to the caller with no value when execution **"falls off the end"** of the function by reaching the closing right brace
4. It is not illegal, but probably a sign of trouble, if a function returns a value from one place and no value from another
5. If a function fails to return a value, its "value" is certain to be **garbage**

**In our program:** The pattern-searching program returns a status from `main`: the number of matches found. This value is available for use by the environment that called the program.

## Running the Program

**Example input** (saved in file `poem.txt`):
```
Ah Love! could you and I with Fate conspire
To grasp this sorry Scheme of Things entire,
Would not we shatter it to bits -- and then
Re-mould it nearer to the Heart's Desire!
```

**Running the program:**
```bash
./a.out < poem.txt
```

**Output:**
```
Ah Love! could you and I with Fate conspire
Would not we shatter it to bits -- and then
Re-mould it nearer to the Heart's Desire! 
```

**Return value:** The program returns 3 (three matches found), which the shell can access using `echo $?` on UNIX. 

## Compiling Multi-File Programs

The mechanics of how to compile and load a C program that resides on multiple source files vary from one system to the next. 

**On the UNIX system**, suppose the three functions are stored in three files: 
- `main.c` (contains main function)
- `getline.c` (contains getline function)
- `strindex.c` (contains strindex function)

**Compile all three files:**
```bash
cc main.c getline.c strindex.c
```

This compiles the three files, placing the resulting object code in files `main.o`, `getline.o`, and `strindex.o`, then loads them all into an executable file called `a.out`.

**If there is an error** (say in `main.c`), the file can be recompiled by itself and the result loaded with the previous object files: 

```bash
cc main.c getline.o strindex.o
```

**How it works:** The `cc` command uses the `.c` versus `.o` naming convention to distinguish source files from object files.  Files ending in `.c` get compiled; files ending in `.o` are already compiled and just get linked together.
