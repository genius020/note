<img width="844" height="801" alt="image" src="https://github.com/user-attachments/assets/2ce211d9-5655-42a6-af10-beb9f5917d5a" />

# Understanding Function Returns in C

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
