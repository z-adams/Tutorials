# C++ for already-programmers

This guide assumes you are already familiar with basic programming concepts.
The idea of variables, conditions, functions, and logical flow shouldn't be
new to you. This guide will attempt to provide a bare-minimum crash course to
C++ as quickly as possible.

**Note**: C++ is a constantly evolving language. Modern C++ paradigms are highly
abstract and go well beyond the scope of this guide. This guide aims to provide
a basis of knowledge which will help you to reason about the principles of
modern C++ when you learn them on your own. C++11 is the first "modern C++"
iteration of C++; this guide will be based on C++03, the previous version, also
often called "C-with-classes C++" due to its simpler nature.

## File Structure

C++ separates its code into "source" and "header" files. Every program begins
with a main source file:

```cpp
// myprogram.cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n" << std::endl;
    return 0;
}

```

Every program begins with an `int main()` function, which is called
automatically when the program runs. It returns `0` to tell the operating
system that everything ran ok.

Notice the line at the top that says:
```cpp
#include <iostream>
```

We used a function from the standard library, called `std::cout`, which is not
a fundamental language feature. In order to use it, we `#include <iostream>` to
tell the compiler to include some external code that has that function. The
`#include` statement literally causes the file listed to be copied and pasted
into our source file.

Here is the structure of a typical C++ program:

We create a header file which describes the interface for some functions:
```cpp
// simple_math.h
#pragma once

int add(int a, int b);

int subtract(int a, int b);
```

Then, we create an associated source (.cpp) file, which includes the header
and contains the implementations of the functions declared in the header:
```cpp
// simple_math.cpp

#include "simple_math.h"

int add(int a, int b)
{
    return a + b;
}

int subtract(int a, int b)
{
    return a - b;
}
```

Finally, we include the *header* in our main file:
```cpp
// main.cpp
#include "simple_math.h"

int main()
{
    int a = 3;
    int b = 5;

    int c = add(a, b);       // c will be 8
    int d = subtract(a, b);  // d will be -2
}
```

We include the header only, because all the `main()` function needs to know
is what goes into the functions, and what comes out. The implementation details
are irrelevant to the use of the functions.

When the program is compiled, each source (.cpp) file is compiled individually.
When main.cpp is compiled, `#include "simple_math.h"` literally copies and
pastes the code from that header into main.cpp, so that `main()` is aware that
`add()` and `subtract()` are indeed legimitate functions that exist.

In our header file, there's a command that says `#pragma once`. This tells the
compiler to only include the header file if it hasn't been included already;
including it multiple times would cause conflicts.

Once the source files are compiled, the "linker" (another stage of compilation)
looks through all the compiled files and makes sure each declared function
really does have an implementation somewhere.

We can repeat this process with a single file as well:

```cpp
// main.cpp

// Forward declare functions
int add(int a, int b);
int subtract(int a, int b);

int main()
{
    int a = 3;
    int b = 5;

    int c = add(a, b);       // c will be 8
    int d = subtract(a, b);  // d will be -2
}

// Define functions down here

int add(int a, int b)
{
    return a + b;
}

int subtract(int a, int b)
{
    return a - b;
}
```

This behaves equivalently; the top of the file (up until the end of `main()`)
looks identical to the result of `#include "simple_math.h"`. As long as the
functions used in `main()` are declared before they're used, the implementation
(definition) can be placed anywhere as long as the linker can find them later.

With that basic background out of the way, now on to the good stuff.

## Data Types

C++ is a **strongly typed** language, meaning every variable has a data type
which is rigorously enforced by the language's type system:

### Boolean Types

The simplest C++ datatype is `bool`, a boolean value:

```cpp
bool a;
```

The `true` and `false` literal are available to store in this type. Boolean
variables can be operated on logically with `&&` (and), `||` (or), `^` (xor),
and `~`(not).

```cpp
bool a = true;
bool b = false;

bool c = a && b;  // "true" AND "false" is false

bool d = a || b;  // "true" OR "false" is true

bool e = ~a;  // NOT "true" is false
```

### Integer ("integral") Types

The most basic datatype of C++ is `int`, which can be modified by various
keywords to change its behavior:

```cpp
int a;  // signed 32-bit integer in range [-2,147,483,648, 2,147,483,647]

unsigned int b;  // unsigned 32-bit integer in range [0, 4,294,967,295]

short int c;  // signed 16-bit integer in range [-32,768 to 32,767]
```

**Caveat:** the standard only enforces the minimum size of these types;
if a compiler wanted, it could make them all 64-bit. If we want integers that
are exactly a certain size, we can use the `<cstdint>` standard library header:

```cpp
#include <cstdint>

uint8_t a;  // unsigned 8-bit integer

int8_t b;   // signed 8-bit integer

uint16_t c;  // unsigned 16-bit integer

// ...

uint64_t h; // unsigned 64-bit integer
```

Integers are simple numbers stored in some number of binary bits. The exact
range of values that can be stored by *n* bits can be found anywhere on the
internet. Generally, 32-bit integers are used for everything unless a different
size is needed for a special reason.

#### Integer Literals

Variables can be assigned with literal types that are understood natively by
the language.

Normal integers can be assigned as you would expect:
```cpp
int a = 3;

int b = -5;
```

There are special characters that can indicate a certain literal type:
```cpp
uint32_t a = 1u;  // 'u' indicates that the literal value '1' is unsigned
```
These are not typically important though as the compiler will automatically
convert a value into the correct type if the types are similar.

There are also integer literals in other base systems; hexadecimal, octal,
and binary. These can be useful if the data being stored is easier to
interpret in these bases.
```cpp
uint32_t a = 0xFFFFFFFF; // 0x or 0X prefixes a hexadecimal literal
uint64_t b = 0xbb9f1f9ae57a80c0; // lower case hex letters are ok too

uint16_t c = 052; // a leading 0 indicates an octal literal
// in practice octal is rarely used

uint8_t d = 0b01000101;  // 0b prefixes a binary literal
```

#### Character Types

A subset of integer types is the `char` type, which represents an ASCII
character. The ASCII character set assigns a number between 0 and 127 to all
the basic english keyboard characters, allowing them to be stored in a
one-byte integer type called `char`. This type can be used just like a normal
8-bit integer, but it also can be assigned to with **character literals**:

```cpp
char myChar = 97;
// single quotes around a single character forms an ASCII character literal
char myChar2 = 'a';  // these two lines are exactly equivalent
```
This will be useful later for handling text.

#### Integer Arithmetic

C++ has built-in operators that can perform integral arithmetic:

```cpp
int a = 3;
int b = 5;

int c = a + b;  // 8
int d = a - b;  // -2

int e = a * b;  // 15
int e = a / b;  // 0
int f = a % b;  // 3
```

Addition, subtraction, and multiplication work as you'd expect. Note that the
`/` operator between integers performs **integer division**, so the result of
`3/5` is not 0.6 (integers have no conception of decimals). `5` goes into `3`
zero times, with `3` left over, so `3/5` is zero. The **modulo** operator (`%`)
performs the same division operation, but returns the remainder instead of the
quotient.

#### Integer Comparison

C++ has logical comparison operators thta behave as they do in any other
language. Their results can be stored in `bool` variables:

```cpp
int a = 3;
int b = 5;


bool d = (3 < 5);  // (3 < 5) is true

bool e = (1 == 2);  // (1 == 2) (equals) is false
```

### Floating-point Types

Integers are very handy and simple datatypes, but they're not very useful for
representing real numbers (decimals).

```cpp
float a;  // 32-bit floating-point number

double b; // 64-bit floating-point number
```

#### Floating-point Literals

Any decimal number is interpreted as a floating point literal. The exact type
can be specified as follows:

```cpp
float a = 3.4f;  // 'f' signifies a "float" type

double b = 4.2;  // without an 'f', the literal is assumed to be a "double"
```

They can also be specified in scientific notation:
```cpp
float a = 3e8f;  // 3 * 10^8

double b = 2.341e-5;  // 0.00002341
```

Floats behave as expected with the normal arithmetic (`+`, `-`, `/`, `*`) and
comparison (`<`, `>`) operators.


#### Floating-point Caveat

The inner workings of floating-point types is magic and beyond the scope of this
crash course, but there are two important caveats to their behavior that you may
not have thought about before:

**Exact value representation**: Floating point values are still ultimately stored
in base-2 (binary). This fact means that any number that's not a multiple of a
power of two cannot be represented exactly. For instance, the float literal
`0.3` is actually stored as the number `0.30000000000000004`, which is the
closest value to `0.3` that floating point values can store. For this reason
it is important to **never compare floats for equality**
(e.g. `floatA == floatB`), as the behavior of this will be unpredictable. Use
greater than/less than and ranges instead.

**Limited precision**: Although floating-point numbers can store a huge range
of values (for 32-bit `float`, about 10^(+/- 38); for 32-bit `double`, about
10^(+/- 308)), they can only store a limited amount of significant figures.
A 32-bit `float` has about 7 sig figs, which can lead to trouble. If you store
the number `3e8f` (300,000,000) in a float, only the first 7 ([300,000,0]00)
digits are represented explicitly. If you try to add `1.0` to this value,
nothing will change at all as `1.0` is out of the range of the significant
figures. The smallest value you can add to the number and see results is
`100.0`; anything smaller will be ignored.

### The End

That's all there is to C++ fundamental types. Every program imaginable is built
out of these fundamental building blocks. Crazy, right?

## Logical Flow

C++ has basic logical flow primitives like most other languages:

```cpp
if (condition)
{
    // do something
}

if (condition)
{
    // do something
}
else
{
    // fallback
}

if (condition)
{
    // do something
}
else if (another condition)
{
    // do something else
}
else
{
    // fallback
}
```

And loops:
```cpp
for (int i = 0; i < max; i++)
{
    // do something "max" times
}

while (condition)
{
    // do something as long as "condition" is true
}

do
{
    // do something and check condition after each time
}
while (condition);
```

## Arrays

C and C++ have a primitive concept of an array. Arrays are simply a big chunk of
values all packed together in memory:

```cpp
int myArray[100];  // 100 ints

float anotherArray[3];  // 3 floats
```

Arrays can be indexed (indices start at zero):
```cpp
int a = myArray[0]; // first element of the array

int b = myArray[5]; // 6th element of the array
```

The size of the array must be specified directly, and be resolvable at
compile time. If the size were unknown, the computer wouldn't know how much
space the array needs in memory.


## Functions

A C++ function is defined by its name, and the datatypes of its inputs and
outputs. This is called the function's **signature**. The function
must be declared before it's used, and then can be defined anywhere.

```cpp
// declaring a function that takes 3 ints and returns another int
int my_function(int a, int b, int c);

// declaring a function that takes zero inputs and returns a float
float generate_val();

// declaring a function that takes two floats and returns nothing
void do_magic(float a, float b);
```

A function is defined elsewhere using the same signature:
```cpp
int my_function(int a, int b, int c)
{
    return a*b + c;
}
```

## Scope

Everything in C++ lives within some **scope**, which defines the lifetime and
visibility of that thing.

In the most simple example, variables can be declared at **global scope**:
```cpp
// Global variable
int myVar = 0;

// Declare function
void set_var_to_3();

int main()
{
    set_var_to_3();

    return 0;
}


// Define function
void set_var_to_3()
{
    myVar = 3;
}
```

The varibale `myVar` is visible everywhere in this file, allowing us to do
things like set its value with a function that doesn't need to accept any
inputs, or return any outputs.

When writing functions with local variables, the variables are scoped to the
function:

```cpp
int foo()
{
    int total = 0;

    // sum up all numbers from 0 to 99
    for (int i = 0; i < 100; i++)
    {
        total = total + i;
    }

    return total;
}
```

The variable `total` is created when the function is called, and is destroyed
when the function returns, so it can't be accessed from anywhere outside the
function.

Variables declared inside conditional blocks are also scoped to that block:
```cpp
int foo()
{
    bool condition = //...

    if (condition)
    {
        int total = 0;
        // ...
    }

    // total is not accessible from here
}
```

`total` in this case is confined to within the `{``}` (curly braces) of the `if`
statement, and are not visible outside. You can even define an arbitrary scope
to keep things clean:

```cpp
int foo()
{
    // do some stuff

    { // begin arbitrary scope
        
        int myVar = 0;

    } // end arbitrary scope

    // myVar is not accessible outside arbitrary scope
}
```


