# C++ in Two Pages

Here is an example C++ program:

```cpp
#include <iostream>
#include "simple_math.h"

Vector3 add(Vector3 a, Vector3 b);

int main()
{
    Vector3 v1;
    v1.x = 3.0f;
    v1.y = 1.0f;
    v1.z = 5.0f;

    // Aggregate initialization
    Vector3 v2 = {-2.0f, 0.0f, 1.0f};

    // Global function call
    Vector3 v3 = add(v1, v2);

    // member function call
    float dotProduct = v1.dot(v2);

    // Print result to console
    std::cout << "Dot product: " << dotProduct << std::endl;

    return 0;
}

Vector3 add(Vector3 a, Vector3 b)
{
    Vector3 result;
    result.x = a.x + b.x;
    result.y = a.y + b.y;
    result.z = a.z + b.z;

    return result;
}
```

`simple_math.h`:
```cpp
#pragma once

struct Vector3
{
    float x;
    float y;
    float z;

    float dot(Vector3 other);
};

```

`simple_math.cpp`:
```cpp

float Vector3::dot(Vector3 other)
{
    return x * other.x + y * other.y + z * other.z;
}
```

The entry point for every C++ program is the `main()` function.
Functions in C++ must be **forward-declared** before use, as seen with the 
`add()` function. The implementation can be anywhere as long as the declaration
precedes the function's use.

C++ declarations are often collected into **header files** (.h). Putting the
declarations together in a file provides a convenient and readable interface
without clutter from implementation details. The functions are actually
implemented in **source files** (.cpp).

Source files can `#include` header files, in order to use the functions listed
in the header. This is identical to forward declaration, as `#include "file.h"`
simply copies and pastes the contents of "file.h" at the location of the
`#include` statement.

Header files begin with a `#pragma once` statement, which prevents the header
file from being accidentally included multiple times and causing conflicts.

Source files are compiled individually. During compilation, the compiler
validates the usage of any function that has been declared, and assumes the
implementation will be found later. After compilation, the **linker** searches
all the compiled sources and resolves these missing links.

In C++, local variables are created instantly. If a function creates a local
variable:
```cpp
int main()
{
    int a;
}
```
memory is immediately allocated to store `a`, and this memory lives directly in
the function's working memory.

Datatypes can be grouped into custom datatypes called structs:
```cpp
struct Point
{
    float x;
    float y;
};
```

All variables belong to some scope. Global variables are scoped to the file,
and other variables are scoped by curly braces (`{`/`}`). For instance, a local
function variable is scoped to the function:
```cpp
void foo()
{
    int bar = 0;
 
    //...
}
```
When the function exits, `bar` is deleted immediately from memory.

Functions accept all arguments by value:
```cpp
struct GiantStruct { /* ... */ };

int main()
{
    GiantStruct s;
    bar(s);  // s is copied by value
}
```
We will see how to avoid this later.

### Memory management, OOP

Here is another example program (now that you know how files work, we can
stuff everything into one file for convenience):

```cpp
class DynamicArray
{
public:
    DynamicArray(int size);
    DynamicArray();
    ~DynamicArray();

    float get(int index);
    void set(int index, float value);
private:
    float* arrayPtr;
    int arraySize;
};

int main()
{
    float staticArray[100];  // statically allocated array

    int dynamicArraySize = 125;
    DynamicArray dynamicArray(dynamicArraySize); // custom dynamic array wrapper

    float firstValue = dynamicArray.get(0);  // get first array element
    dynamicArray.set(1, 1.0f);  // set 2nd array element

    return 0;
}

// Constructor implementation
DynamicArray::DynamicArray(int size)
{
    // Allocate a dynamic array with "size" elements
    internalArray = new float[size];

    // Store size for later
    arraySize = size;
    
    // Initialize all array elements to zero
    for (int i = 0; i < size; i++)
    {
        internalArray[i] = 0.0f;
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

// Getter
float DynamicArray::get(int index)
{
    if (internalArray != nullptr)
    {
        return internalArray[index];
    }
}

// Setter
void DynamicArray::set(int index, float value)
{
    if (internalArray != nullptr)
    {
        internalArray[index] = value;
    }
}
```

C++ classes and structs are identical, except that classes have private
members by default. By convention, structs (which are left over from C) are used
for POD ("plain old data") types, which don't have complicated logic built-in.

Classes and structs are forward-declared like functions, and their internal
functions are implemented elsewhere by prefixing their function name with the
class name (e.g. `void DynamicArray::set()`).

C++ operates and passes by value. It allows for dealing with data by reference
indirectly with **pointers**:
```cpp
int a = 5;  // an int

int* p = &a;  // a pointer (int*) which stores the address of a (&a)

int b = *p;  // dereferencing p to access the contents of a, store it in b
```

If the program requires data whose size is unknown at runtime, or needs to live
longer than the scope in which it's declared, it must be allocated
**dynamically**:
```cpp
int* p = new int;  // dynamically allocate an int

delete p;  // don't forget to delete your dynamic memory

// For arrays:
int* pArray = new int[100];

delete[] pArray;
```

Local arrays are allowed too:
```cpp
int myArray[100];  // local array of 100 ints

myArray[3]; // array access
```
These arrays can only be created if the size is known at compile time. Under
the hood the array is a contiguous region of memory and the datatype of
`myArray` is actually a pointer. The square-bracket array access is shorthand
for `*(myArray + 3)`, and can be used with dynamic pointers as well.

C++ also has references, which are the same as pointers but convert implicitly
and can't be null:

```cpp
int a = 5;
int& r = a;  // reference to a created
int b = r;  // a's value fetched automatically

// Contrast with pointers:
int* p = &a;  // fill pointer with address of a, explicitly
int c = *p;  // explicitly dereference p to get value of a
```

You can see in the 2nd example program how these concepts come together to form
an encapsulated `DynamicArray` type which stores a private pointer to a
dynamically allocated array of floats. The class's **constructor** (a
return-less function with the same name as the class) is called when the object
is created, and the **destructor** (a return-less function with the same name as
the class, prefixed by an `~`) is called automatically when the object goes out
of scope or is deleted with `delete`. This class demonstrates **RAII** (Resource
Acquisition is Initialization), a core principle of C++ which uses constructors
and destructors to automatically allocate and free memory to avoid memory leaks.
