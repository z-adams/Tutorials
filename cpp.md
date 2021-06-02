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

## File Structure, Language Overview

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

### Object Lifetime

Unlike memory-managed languages like Java, when a local variable is created, it
exists immediately. This is in contrast to Java, where `new Object()` is
required to actually create the object in memory.

When an object is created, it is created somewhere in the computer's physical
memory. This memory region may be full of random **"garbage"** values from some
previous usage, so every object *must be manually initialized*, or it may
contain a garbage value. We will discuss how to do this later.

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

**Caveat:** the standard only enforces the minimum size of these types.
This means that you can never be sure exactly how large each datatype is.
When you ask people what size an `int` is, they usually say something like "it
depends on the platform." Although almost all platforms use 4-byte (32-bit)
ints, it's true that you never know exactly how big integer type is.
If we want integers that are exactly a certain size, we can use the `<cstdint>`
standard library header:

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

## Structures

Structures, or **structs** are simple aggregations of variables. If we were
stuck with only making variables of fundamental types, it would get pretty
arduous. Luckily, structs allow us to group values together into a single unit.
For example:
```cpp
struct Point
{
    float x;
    float y;
};
```

Rather than storing each coordinate individually, we can define a struct that
stores them in a convenient package. `x` and `y` here are called
**member fields** or **member variables** of the struct. Now, we can use the
struct like a normal datatype:

```cpp
Point myPoint;

myPoint.x = 1.0f;
myPoint.y = 3.0f;
```

Structures can be "aggregate initialized," which accepts a list of elements in
curly braces that will be mapped onto the member of the struct, in order
(in this case `5.0f` will go into `x`, `6.0f` will go into `y`)
```cpp
Point otherPoint = {5.0f, 6.0f};
```

We can use structs to package fundamental types into conceptually powerful
data:
```cpp
struct Vector3
{
    float x;
    float y;
    float z;
};

struct PhysicalObject
{
    Vector3 position;
    Vector3 velocity;
    float mass;
};
```

And all of a sudden, we can start to see how fundamental types are built up into
real programs.

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

myArray[10] = 3; // setting the value of an array element
```

Arrays can also be "aggregate initialized":
```cpp
int someNumbers[5] = {3, 12, -5, 99, 42};
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

## Pass by Value

Everything in C++ is pass-by-value. When you assign `int a = b;`, you are
copying the value of `b` into `a`. There is no soft link between the two like
there is in Java or Python. The same goes for function parameters:
```cpp
// A structure containing an array of 1000 64-bit ints, for a total size of 8kB
struct HugeData
{
    uint64_t data[1000];
};

HugeData myData;  // 8kB variable

// The "data" parameter will be copied, all 8kB of it, into the function. This
// is not very efficient...
void process_data(HugeData data);
```

We will see how to resolve this issue when we discuss pointers.

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

## Pointers

Now, onto the good stuff. Sometimes, we want values to transcend the structure
of our program imposed by things like variable scoping. It can also be useful to
refer to variables indirectly. Enter the **pointer**.

The pointer is literally just a memory location. Variables in the program all
live somewhere in memory. If we want to get a particular piece of data (not the
value of the data, but the data itself), we can do that with a pointer:

```cpp
int* p;  // a variable called p with type "int*" (pointer to an int)
```

We can assign the pointer to the **address** of an existing variable:
```cpp
int a = 100;

int* p = &a;  // &a means "address of a"
```

Now `p` stores `a`'s address, which is just some big ugly number that describes
`a`'s location in memory (it usually looks something like `0x31f30af1ba31a8c0`).
The value inside `p` itself isn't important though; what is important is that we
can use `p` to find the value of `a` again:

```cpp
int a = 100;

a = 200;

int* p = &a;

int b = *p;  // b now stores 200
```

We **dereference** `p` with `*p`, which just means "`p` holds a memory address;
take that memory address and actually go there, and find whatever data is stored
there." In this case, that address is the location of `a`, so the value stored
there is `100`. Dereferencing the pointer always gets us the most recent value
stored in `a`, even if it's changed.

#### Concrete Example

Say we have an array of integers, full of random 32-bit unsigned integers:
```cpp
uint32_t myIntegers[100] = {3, 13, 107, //...
```

We want to find the largest element of this array. Not the largest *value*, the
largest *element itself* (maybe we want to modify this element once it's found,
perhaps as part of a sorting algorithm). We can accomplish this easily with
pointers.

Our function signature looks like this:
```cpp
uint32_t* find_largest(uint32_t* array, int arraySize);
```
We want our array to process the array, but we don't want to copy it all into
the function (remember, all functions pass by value). Instead, we are going to
pass a pointer to the beginning of the array, along with the number of elements
in the array. With these pieces of data, we can access the array from inside the
function without needing to pass a large amount of data.

Our function returns a pointer, which we want to point to the largest element of
the array. Here's how the function definition looks:

```cpp
uint32_t* find_largest(uint32_t* array, int arraySize)
{
    uint32_t largestValueSoFar = 0;  // initialize to smallest possible value

    uint32_t* largestElement = nullptr; // nullptr is the "nothing" pointer value

    for (int i = 0; i < arraySize; i++)
    {
        if (array[i] > largestValueSofar)
        {
            // Found a value bigger than anything so far

            // New biggest value
            largestValueSoFar = array[i];

            // "Bookmark" this element as the largest by storing its address in
            // our "largest element" pointer
            largestElement = &array[i];
        }
    }

    // At the end, the "largestElement" pointer stores the biggest array element
    // we found, so return it
    return largestElement;
}
```

Now we can call our function:
```cpp
uint32_t myIntegers[100] = {3, 13, 107, //...

uint32_t* find_largest(uint32_t* array, int arraySize);

uint32_t* largestElement = find_largest(myIntegers, 100);
```

There are a few details here I glossed over, such as the fact that the array
behaves just like a pointer when passing it into the function, and the function
indexes the array even though it's a pointer. That's because arrays **decay** to
pointers, meaning that when a named array (such as `myIntegers`) is used,
its name actually refers to a pointer to the beginning of the array, not the
chunk of data itself.

## Dynamic Memory

Sometimes, though, we need more. Named variables are limiting, because they
require you to list them before they are used. What if we don't know what kind
of data our program needs? What if we don't know how much? Dynamic memory allows
us to circumvent these limitations at runtime.

Say we are making a program that keeps track of player scores. Each player has
a score represented by an `int`. We want our program to begin with something
like:
```
> How many players?
> Enter number of players:
```
and have the user input a number. In order to support this behavior, we need to
be able to allocate data at runtime.

#### Throwback; Memory Leaks

In C, memory was allocated with `malloc()`. `malloc()` accepts a memory size, in
bytes, and returns a pointer to the memory it creates. This would be used like:

```cpp
struct MyStruct
{
    int a;
    float b;
};

MyStruct* myStruct = malloc(sizeof(MyStruct));
```

`sizeof()` is a build-in C/C++ function which returns the side of a datatype. In
this case, `MyStruct` stores a 4-byte `int` and a 4-byte `float`, so its total
size is 8 bytes. `malloc()` will then go interact with the Operating System's
kernel, and request access to more memory. It will then return a pointer to the
new chunk of memory, which you use to store your struct.

This dynamically allocated memory is called "heap memory," because it's
allocated in an unorganized way compared to the local memory of functions, which
is allocated on the "stack" and is automatically cleaned up because the compiler
knows exactly how much memory each function needs by the size of its local
variables.

Heap memory is not tracked automatically, and must be manually freed. In C, this
was done with the `free()` function:
```cpp
free(myStruct);
```

**If you don't free your dynamic memory, and you lose track of it (by losing
the pointer variable or pointing the pointer to something else), it will be
"leaked" and become inaccessible to your program. Try to avoid doing this.**

#### `new`/`delete`

In C++, we have a more convenient tool for creating and freeing dynamic/heap
memory, called `new` and `delete`. A new object is allocated like so:
```cpp
struct MyStruct
{
    int a;
    float b;
};

// Allocates memory for a MyStruct, returns pointer to it to store in myStruct
MyStruct* myStruct = new MyStruct;

// Deletes the dynamic memory pointed to by myStruct
delete myStruct;
```

The words are different, and there are some subtle technical differences, but
the behavior is the same.

#### Back to our program:

Now we're equipped to deal with our original problem:
```cpp
int main()
{
    int numPlayers = // get from user

    // Dynamically allocate score array
    int* playerScoreArray = new int[numPlayers];

    playerScoreArray[0] = // ... do stuff with scores

    // Before exit, free score array
    delete[] playerScoreArray;
    
    return 0;
}
```
`new T[]` and `delete[]` are special versions of `new`/`delete` used for dealing
with arrays, but their behavior is identical.

There are other scenarios that are impossible without pointers. For instance,
chaining objects together.

Let's say we want to build a chain of nodes, where each node lets us access the
next node:
```cpp
struct Node
{
    int value;

    Node nextNode;
};
```
This is impossible. We can inspect the size of one node. Each node stores an
`int` (4 bytes), and another Node. Except the size of the other node is also
(4 + size of child node). This continues recursively, implying that each node
requires an infinite amount of memory. Pointers break this nesting behavior:

```cpp
struct Node
{
    int value;
    Node* nextNode;
};

Node a;
Node b;

a.nextNode = &b;
```
Now the chain is broken; `sizeof(Node)` is now just 12: `sizeof(int)` (4, almost
always) + `sizeof(Node*)` (8; all pointers are 64-bits on a 64-bit machine),
and our program is now well defined.

## Object-Oriented Programming

OOP (Object-Oriented Programming) is built into C++, in the form of
**member functions**. A struct in C++ can have a function as a member:
```cpp
struct Vector3
{
    float x;
    float y;
    float z;

    float vector_length();
}
```

This function is declared within the struct, and defined externally:
```cpp
float Vector3::vector_length()
{
    return sqrt(x*x + y*y + z*z);
}
```
Note the `Vector3::` which indicates that the function belongs to that struct.
The workings of a member function are as with any other OOP language; the
function is automatically given access to the member fields of the object
whose function was called. As with many other languages, member functions can
refer to its parent object via the `this` pointer:

```cpp
float Vector3::vector_length()
{
    float val = this->x;
}
```

Normally this is not required though; the `this` is implied and only is
necessary if another variable name conflicts with a member variable name.

### Classes

In addition to structs, C++ also has **classes**. In C++, structs and classes
are practically identical. The only difference between the two is that, by
default, the contents of a class are **private**, meaning they cannot be
accessed from the outside:

```cpp
class EmployeeDatabase
{
    float salaries[100];
};

int main()
{
    EmployeeDatabase db;

    float stolenData = db.salaries[0];  // ERROR: can't access private member

    return 0;
}
```

If a class needs to communicate with the outside, it needs to explicitly declare
members as public:
```cpp
struct Date { /* ... */ };

class EmployeeDatabase
{
public:
    void set_salary(int index, float salary);

    Date birthdays[100];

private:
    float salaries[100];
};

int main()
{
    EmployeeDatabase db;

    float stolenData = db.salaries[0];  // ERROR: can't access private member
    
    db.set_salary(0, -1000.0f);  // Legal, member function is public

    Date birthday = db.birthdays(4);  // Legal, member variable is public

    return 0;
}
```

Note that once you've started a `public:` section, you need to explicitly start
a `private:` section to make private members. By convention, it's good to *put
the public members first* as someone using your class will be primarily
interested in the interface it provides, not its inner workings.

All of this can be done exactly the same in structs (as they are identical to
classes, apart from the default-public member visibility). However, by
convention, structs are typically used for **POD** ("Plain Old Data") types,
which store only data, and have no member functions. Use classes for any object
that has nontrivial logic embedded in it.

### Constructors and Destructors

In C++, structs and classes can have constructors and destructors:

```cpp
class DynamicArray
{
public:
    DynamicArray(int size);  // Constructor

    DynamicArray();  // Default constructor

    ~DynamicArray();  // Destructor

private:
    int arraySize;
    int* internalArray;
};
```

Constructors are functions with no return type, and with the same name as
the struct. Destructors are the same, except their name leads with a `~`.

Constructors are likely familiar from other OOP languages. *Destructors* are
probably not. In C++, constructors and destructors are particularly useful
because they allow for dynamic memory to be handled in a programmatic way. Let's
look at the implementations of this class:

```cpp
// Constructor implementation
DynamicArray::DynamicArray(int size)
{
    // Allocate a dynamic array with "size" elements
    internalArray = new int[size];

    // Store size for later
    arraySize = size;
    
    // Initialize all array elements to zero
    for (int i = 0; i < size; i++)
    {
        internalArray[i] = 0;
    }
}

// Default constructor implementation
DynamicArray::DynamicArray()
{
    internalArray = nullptr;
    arraySize = 0;
}

// Destructor implementation
DynamicArray::~DynamicArray()
{
    // Check to see if array exists
    if (internalArray != nullptr)
    {
        // Free dynamic memory
        delete[] internalArray;
    }
}
```

In the constructor, we allocate the dynamic memory needed by our class. In our
destructor, we free the dynamic memory needed by our class. Since the
constructor and destructor are guaranteed to be called:

```cpp
int main()
{
    DynamicArray myArray(50);  // Passes 50 to DynamicArray's constructor
    // Memory is allocated in constructor

    // ... do stuff

    return 0;

    /* When main() ends, all its local variables (including myArray) go out of
       scope. When a variable goes out of scope, its destructor is automatically
       called, so myArray's dynamic memory is freed automatically, and is not
       leaked.
    */
}
```

This principle is called **RAII** (**R**esource **A**cquisition **i**s
**i**nitialization) and is a core guideline for writing C++ code. If you follow
RAII carefully, it becomes much harder to accidentally lose track of dynamic
memory.

