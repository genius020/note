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
