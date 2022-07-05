---
title: Compile time function execution (CTFE)
parent: Introduction to Meta-Programming
nav_order: 2
---

## Compile time function execution (CTFE)

CTFE is a mechanism which allows the compiler to execute functions at compile time.
There is no special set of the D language necessary to use this feature - whenever a function just depends on compile time known values the D compiler might decide to interpret it during compilation.

Keywords like [static](https://dlang.org/spec/attribute.html#static), [ immutable](https://dlang.org/spec/attribute.html#immutable) or [enum](https://dlang.org/spec/enum.html#manifest_constants) instruct the compiler to use CTFE.

```d
int sum(int a, int b)
{
    return a + b;
}

void main()
{
    enum a = 5;
    enum b = 7;
    enum c = sum(a, b);
}
```

In the above example, **sum(a, b)** is evaluated at compile time.
In the object file, no call to **sum** is issued.

TODO: quiz
> :warning: If the type of **a** or **b** is changed to **int**, the compiler will issue an error. Why?
