---
title: OmniOpenCon Workshop
parent: OmniOpenCon Workshop
nav_order: 1
---

<details markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# OmniOpenCon Workshop

## Write fast. Read fast. Run fast.

> "D is a multi-paradigm system programming language that combines a wide range of powerful programming concepts from the lowest to the highest levels.
> It emphasizes memory safety, program correctness, and pragmatism." - Ali Çehreli, Programming in D, 2018

## Intro to D

### Syntax

The D programming language uses a C-style syntax that ensures a smooth transition for programmers coming from a C\C++ background.
With no further ado, let us take a random C program and see what it takes to compile it with the D compiler:

```c
#include <stdio.h>

int main()
{
    int position = 7, c, n = 10;
    int array[n] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    for (c = position - 1; c < n - 1; c++)
        array[c] = array[c+1];

    printf("Resultant array:\n");
    for (c = 0; c < n - 1; c++)
        printf("%d\n", array[c]);

    return 0;
}
```

The code above simply deletes an element in an array.
Now let's take a look on what the minimum modifications are to make the code compile and run with D:

```d
import std.stdio;

int main()
{
    int position = 7, c, n = 10;
    int[10] array = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    for (c = position - 1; c < n - 1; c++)
        array[c] = array[c+1];

    printf("Resultant array:\n");
    for (c = 0; c < n - 1; c++)
        printf("%d\n", array[c]);

    return 0;
}
```

As you can see, the only differences are:

- the `#include` directive was replaced by an `import` statement;
- the array definition and initialization are slightly modified;

> Most C programs require minimal changes in order to be compiled with the D compiler.
> So do not worry, even if you don't have any previous experience in D, you will be able to understand most of the programs written in it because the syntax is extremely similar to the C one.

### Imports

In D, `imports` represent the counterpart of the C `include` directive.
However there are 2 fundamental differences:

- Imports may selectively specify which symbols are to be imported. For example, in the above code snippet, the full standard IO module of the standard library is imported, even though only the `printf` function is used.
This results in a degradation in compile time since there is a larger symbol pool that needs to be examined when trying to resolve a symbol.
In order to fix this, we can replace the blunt import with:
```d
import std.stdio : printf;
```

- Imports may be used at any scope level.
```d
int main()
{
    int position = 7, c, n = 10;
    int[10] array = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    for (c = position - 1; c < n - 1; c++)
        array[c] = array[c+1];

    // If the lines calling printf are deleted,
    // it is easier to spot the now useless import
    import std.stdio : printf;
    printf("Resultant array:\n");
    for (c = 0; c < n - 1; c++)
        printf("%d\n", array[c]);

    return 0;
}
```

### Data Types

The D programming language defines 3 classes of data types:

1. [Basic data types](https://dlang.org/spec/type.html#basic-data-types): such as `int`, `float`, `long` etc. that are similar to the ones provided by C;
1. [Derived data types](https://dlang.org/spec/type.html#derived-data-types): pointer, array, associative array, delegate, function;
1. [User defined types](https://dlang.org/spec/type.html#user-defined-types): class, struct, union, enum;

We will not insist on basic data types and pointers, as those are the same (or slightly modified versions) as the ones in C\C++.
We will focus on arrays, associative arrays, classes, structs and unions.
Delegates, functions and enums will be treated in a future lab.

Note that in D all types have a default value.
This means that there are no uninitialized variables.

```d
    int a; // equivalent to int a = 0;
    int *p; // equivalent to int *p = null;
    // The same goes for structs and classes: their fields are recursively initialised.
```

### Arrays

The fundamental difference between a C array and a D array is that the former is represented by a simple pointer, while the latter is represented by a pointer and a size.
This design decision was taken because of the numerous cases of C buffer overflow attacks that can be simply mitigated by automatically checking the array bounds.
Let's take a simple example:

```c
#include <stdio.h>

void main()
{
    int a = 5;
    int b[10];
    b[11] = 9;
    printf("%d\n", a);
}
```

Compiling this code (without the stack protector activated) will lead to a buffer overflow in which the variable `a` will be modified and the print will output `9`.
Compiling the same code with D (simply replace the include with an `import std.stdio : printf`) will issue a compile time error that states that 11 is beyond the size of the array.
Aggregating the pointer of the array with its size facilitates a safer implementation of arrays.

D implements 2 types of arrays:
1. [Static Arrays](https://dlang.org/spec/arrays.html#static-arrays): their length must be known at compile time and therefore cannot be modified; all examples up until this point have been using static arrays.
1. [Dynamic Arrays](https://dlang.org/spec/arrays.html#dynamic-arrays): their length may change dynamically at run time.

#### Slicing

Slicing an array means to specify a subarray of it.
An array slice does not copy the data, it is only another reference to it:

```d
void main()
{
    int[10] a;       // declare array of 10 ints
    int[] b;

    b = a[1..3];     // a[1..3] is a 2 element array consisting of a[1] and a[2]
    int x = b[1];    // equivalent to `int x = 0;`
    a[2] = 3;
    int y = b[1];    // equivalent to `int y = 3;`
}
```

#### Array setting

```d
void main()
{
    int[3] a = [1, 2, 3];
    int[3] c = [ 1, 2, 3, 4];  // error: mismatched sizes
    int[] b = [1, 2, 3, 4, 5];
    a = 3;                     // set all elements of a to 3
    a[] = 2;                   // same as `a = 3`; using an empty slice is the same as slicing the full array
    b = a[0 .. $];             // $ evaluates to the length of the array (in this case 10)
    b = a[];                   // semantically equivalent to the one above
    b = a[0 .. a.length];      // semantically equivalent to the one above
    b[] = a[];                 // semantically equivalent to the one above
    b[2 .. 4] = 4;             // same as b[2] = 4, b[3] = 4
    b[0 .. 4] = a[0 .. 4];     // error, a does not have 4 elements
    a[0 .. 3] = b;             // error, operands have different length
}
```

**.length** is a builtin array property.
For an extensive list of array properties click [here](https://dlang.org/spec/arrays.html#array-properties).

#### Array Concatenation

```d
void main()
{
    int[] a;
    int[] b = [1, 2, 3];
    int[] c = [4, 5];

    a = b ~ c;       // a will be [1, 2, 3, 4, 5]
    a = b;           // a refers to b
    a = b ~ c[0..0]; // a refers to a copy of b
    a ~= c;          // equivalent to a = a ~ c;
}
```

Concatenation always creates a copy of its operands, even if one of the operands is a 0-length array.
The operator `~=` does not always create a copy.

When adjusting the length of a dynamic array there are 2 possibilities:
1. The resized array would overwrite data, so in this case a copy of the array is created.
1. The resized array does not interfere with other data, so it is resized in place.
For more information click [here](https://dlang.org/spec/arrays.html#resize).

#### Vector operations

```d
void main()
{
    int[] a = [1, 2, 3];
    int[] b;
    int[] c;

    b[] = a[] + 4;         // b = [5, 6, 7]
    c[] = (a[] + 1) * b[];  // c = [10, 18, 28]
}
```

Many array operations, also known as vector operations, can be expressed at a high level rather than as a loop.
A vector operation is indicated by the slice operator appearing as the left-hand side of an assignment or an op-assignment expression.
The right-hand side can be an expression consisting either of an array slice of the same length and type as the left-hand side or a scalar expression of the element type of the left-hand side, in any combination.

Using the concepts of slicing and concatenation, we can modify the original example (that does the removal of an element from the array) so that the `for` loop is no longer necessary:

```d
int main()
{
    int position = 7, c, n = 10;
    int[] array = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    array = array[0 .. position] ~ array[position + 1 .. $];

    import std.stdio;
    writeln("Resultant array:");
    writeln(array);

    return 0;
}
```

[`writeln`](https://dlang.org/library/std/stdio/writeln.html) is a function from the standard D library that does not require a format and is easily usable with a plethora of types.

As you can see, the resulting code is much more expressive and fewer lines of code were utilized.

### Associative Arrays (AA)

Associative arrays represent the D language hashtable implementation and have the following syntax:

```d
void main()
{
    // Associative array of ints that are indexed by string keys.
    // The KeyType is string.
    int[string] aa;

    // set value associated with key "hello" to 3
    aa["hello"] = 3;
    int value = aa["hello"];

    // remove the pair ("hello", 3)
    aa.remove("hello")
}
```

`remove(key)` does nothing if the given key does not exist and returns `false`.
If the given key does exist, it removes it from the AA and returns `true`.
All keys can be removed by using the method `clear`.

For more advanced operations on AAs check this [link](https://dlang.org/spec/hash-map.html#construction_assignment_entries).
For an exhaustive list of the AA properties check this [link](https://dlang.org/spec/hash-map.html#properties).

### Structs

In D, structs are simple aggregations of data and their associated operations on that data:

```d
struct Rectangle
{
    size_t length, width;
    int id;

    size_t area() { return length * width; }
    size_t perimeter() { return 2 * (length + width); }
    size_t isSquare() { return length == width; }
    void setId(int id) { this.id = id; }
}
```

### Classes

In D, classes are very similar with Java classes:

1. Classes can implement any number of interfaces
1. Classes can inherit a single class
1. Class objects are instantiated by reference only
1. super has the same meaning as in Java
1. overriding rules are very similar

The fundamental difference between **structs** and **classes** is that the former are **value** types, while the latter are **reference** types.
This means that whenever a struct is passed as an argument to an lvalue function parameter, the function will operate on a copy of the object.
When a class is passed as an argument to an lvalue function parameter, the function will receive a reference to the object.

Both **structs** and **classes** are covered more in depth in [Advanced Structs and Classes](../structs-classes/asc.md).

### Functions

Functions are declared the same as in C. In addition, D offers some convenience features like:

#### Uniform function call syntax (UFCS)

UFCS allows any call to a free function `fun(a)` to be written as a member function call: `a.fun()`

```d
import std.algorithm : group;
import std.range : chain, dropOne, front, retro;

//retro(chain([1, 2], [3, 4]));
//front(dropOne(group([1, 1, 2, 2, 2])));

[1, 2].chain([3, 4]).retro; // 4, 3, 2, 1
[1, 1, 2, 2, 2].group.dropOne.front; // (2, 3)

```

#### Overloading:

```d
import std.stdio : writefln;

void add(int a, int b)
{
    writefln("sum = %d", a + b);
}

void add(double a, double b)
{
    writefln("sum = %f", a + b);
}

void main()
{
    add(10, 2);
    add(5.3, 6.2);
}
```

Function overloading is a feature of object-oriented programming where two or more functions can have the same name but different parameters.
Overloading is done based on the **type of parameters**, not on the **return type**.

#### Default parameters:

```d
void fun(int a, int b=8) {}

void main()
{
    fun(7);    // calls fun(7, 8)
    fun(2, 3); // calls fun(2, 3)
}
```

A function may have any number of default parameters. If a default parameter is given, all following parameters must also have default values.

#### Auto return type:

```d
auto fun()
{
    return 7;
}

void main()
{
    int b = fun();
    auto c = fun();   // works for variables also
}
```

Auto functions have their return type inferred based on the type of the return statements.
Auto can be also used to infer the type of a variable declaration.