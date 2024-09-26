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
