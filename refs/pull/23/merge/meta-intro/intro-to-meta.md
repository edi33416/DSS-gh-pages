---
nav_order: 1
---
# Introduction to Meta-Programming

During this lab you will be introduced to the basic concepts of meta-programming: compile time execution and templates.

## Manifest Constants

Manifest constants are variables evaluated at compile time.
They cannot change their value.
A manifest constant is declared by using the keywork **enum**:

```d
enum i = 4;      // i is 4 of type int
enum long l = 3; // l is 3 of type long
```

Manifest constants are not lvalues, meaning their address cannot be taken. They exist only in the memory of the compiler.
Declaring a manifest constant that cannot be evaluated at compile time is an error:

```d
void main()
{
    int a = 5;
    enum b = 5 + 2;   // ok, '5' and '2' are integer literals, known at compile time
    enum b = a + 2;   // error: 'a' cannot be read at compile time  
}
```

To make the above code work, **a** should be declared as an **enum**.

Manifest constants can be seen as compile time variable declarations.

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

## Templates

Templates are the feature that allows describing the code as a pattern, for the compiler to generate program code automatically.
Parts of the source code may be left to the compiler to be filled in until that part is actually used in the program.
Templates are very useful especially in libraries because they enable writing generic algorithms and data structures, instead of tying them to specific types.

To see the benefits of templates let's start with a function that prints values in parentheses:

```d
void printInParens(int value)
{
    writefln("(%s)", value);
}
```

Because the parameter is specified as `int`, that function can only be used with values of type `int`, or values that can automatically be converted to `int`.
For example, the compiler would not allow calling it with a `floating point` type.
Let's assume that the requirements of a program changes and that other types need to be printed in parentheses as well.
One of the solutions for this would be to take advantage of function overloading and provide overloads of the function for all of the types that the function is used with:

```d
// The function that already exists
void printInParens(int value)
{
    writefln("(%s)", value);
}

// Overloading the function for 'double'
void printInParens(double value)
{
    writefln("(%s)", value);
}
```

This solution does not scale well because this time the function cannot be used with e.g. real or any user-defined type.
Although it is possible to overload the function for other types, the cost of doing this may be prohibitive.
An important observation here is that regardless of the type of the parameter, the contents of the overloads would all be generically the same: a single **writefln()** expression.

Such genericity is common in algorithms and data structures.
For example, the binary search algorithm is independent of the type of the elements: it is about the specific steps and operations of the search.
Similarly, the linked list data structure is independent of the type of the elements: linked list is merely about how the elements are stored in the container, regardless of their type.
Templates are useful in such situations: once a piece of code is described as a template, the compiler generates overloads of the same code automatically according to the actual uses of that code in the program.

### Function templates

Defining a function as a template is leaving one or more of the types that it uses as unspecified, to be deduced later by the compiler.
The types that are being left unspecified are defined within the template parameter list, which comes between the name of the function and the function parameter list.
For that reason, function templates have two parameter lists: 1) the template parameter list and 2) the function parameter list.

```d
void printInParens(T)(T value)
{
    writefln("(%s)", value);
}
```

The `T` within the template parameter list above means that `T` can be any type.
Although `T` is an arbitrary name, it is an acronym for "type" and is very common in templates.
Since `T` represents any type, the templated definition of `printInParens()` above is sufficient to use it with almost every type, including the user-defined ones:

```d
import std.stdio : writefln;
void printInParens(T)(T value)
{
    writefln("(%s)", value);
}
void main()
{
    printInParens(42);    // with int
    printInParens(1.2);   // with double

    auto myValue = MyStruct();
    printInParens(myValue);   // with MyStruct
}

struct MyStruct
{
    string toString() const
    {
        return "hello";
    }
}
```

The compiler considers all of the uses of `printInParens()` in the program and generates code to support all those uses.
The program is then compiled as if the function has been overloaded explicitly for `int`, `double`, and `MyStruct`:

```d
/* Note: These functions are not part of the source
 *  code. They are the equivalents of the functions that
 * the compiler would automatically generate.
 */
void printInParens(int value)
{
    writefln("(%s)", value);
}

void printInParens(double value)
{
    writefln("(%s)", value);
}

void printInParens(MyStruct value)
{
    writefln("(%s)", value);
}
```

The output of the program is produced by those different instantiations of the function template:

```d
(42)
(1.2)
(hello)
```

#### More than one template parameter

Let's change the function template to take the parentheses characters as well:

```d
void printInParens(T)(T value, char opening, char closing)
{
    writeln(opening, value, closing);
}
```

Now we can call the same function with different sets of parentheses:

```d
printInParens(42, '<', '>');
```

Although being able to specify the parentheses makes the function more usable, specifying the type of the parentheses as char makes it less flexible because it is not possible to call the function with characters of type `wchar` or `dchar`:
```d
printInParens(42, '→', '←');    // compilation error
```

One solution would be to specify the type of the parentheses as `dchar` but this would still be insufficient as this time the function could not be called e.g. with `string` or user-defined types.
Another solution is to leave the type of the parentheses to the compiler as well.
Defining an additional template parameter instead of the specific `char` is sufficient:

```d
void printInParens(T, ParensType)(T value, ParensType opening, ParensType closing)
{
    writeln(opening, value, closing);
}
```

The meaning of the new template parameter is similar to `T`'s: `ParensType` can be any type.
It is now possible to use many different types of parentheses.
The following are with `wchar` and `string`:

```d
printInParens(42, '→', '←');
printInParens(1.2, "-=", "=-");
```

Output:
```d
→42←
-=1.2=-
```

The flexibility of `printInParens()` has been increased, as it now works correctly for any combination of `T` and `ParensType` as long as those types are printable with `writeln()`.

#### Type deduction

The compiler's deciding on what type to use for a template parameter is called type deduction.

Continuing from the last example above, the compiler decides on the following types according to the two uses of the function template:
  * `int` and `wchar` when 42 is printed
  * `double` and `string` when 1.2 is printed

The compiler can deduce types only from the types of the parameter values that are passed to function templates.
Although the compiler can usually deduce the types without any ambiguity, there are times when the types must be specified explicitly by the programmer.

#### Explicit Type Specification

Sometimes it is not possible for the compiler to deduce the template parameters.
A situation that this can happen is when the types do not appear in the function parameter list.
When template parameters are not related to function parameters, the compiler cannot deduce the template parameter types.
To see an example of this, let's design a function that asks a question to the user, reads a value as a response, and returns that value.
Additionally, let's make this a function template so that it can be used to read any type of response:

```d
T getResponse(T)(string question)
{
    writef("%s (%s): ", question, T.stringof);
    T response;
    readf(" %s", &response);
    
    return response;
}
```

That function template would be very useful in programs to read different types of values from the input.
For example, to read some user information, we can imagine calling it as in the following line:
```d
getResponse("What is your age?");
```

Unfortunately, that call does not give the compiler any clue as to what the template parameter `T` should be.
What is known is that the question is passed to the function as a string, but the type of the return value cannot be deduced:
```d
Error: template deneme.getResponse(T) cannot deduce template function from argument types !()(string)
```

In such cases, the template parameters must be specified explicitly by the programmer.
Template parameters are specified in parentheses after an exclamation mark:
```d
getResponse!(int)("What is your age?");
```

#### Template Instantiation

Automatic generation of code for a specific set of template parameter values is called an instantiation of that template for that specific set of parameter values.
For example, `to!string` and `to!int` are two different instantiations of the `to` function template.

### Struct and Class templates

Structs and classes can be defined as templates as well, by specifying a template parameter list after their names:

```d
class Stack(T)
{
private:
    T[] elements;
public:
    void push(T element)
    {
        elements ~= element;
    }
    void pop()
    {
        --elements.length;
    }
    T top()
    {
        return elements[$ - 1];
    }
    size_t length()
    {
        return elements.length;
    }
}

void main()
{
    auto stack = new Stack!double();
}
```

Since structs and classes are not functions, they cannot be called with parameters.
This makes it impossible for the compiler to deduce their template parameters.
The template parameter list must always be specified for struct and class templates.

### Default Template Parameters

Sometimes it is cumbersome to provide template parameter types every time a template is used, especially when that type is almost always a particular type.
For example, `getResponse()` may almost always be called for the int type in the program, and only in a few places for the double type.
It is possible to specify default types for template parameters, which are assumed when the types are not explicitly provided.
Default parameter types are specified after the `=` character:

```d
T getResponse(T = int)(string question)
{
// ...
}
// ...
    auto age = getResponse("What is your age?");
```

As no type has been specified when calling `getResponse()` above, `T` becomes the default type `int` and the call ends up being the equivalent of `getResponse!int()`.

Default template parameters can be specified for struct and class templates as well, but in their case the template parameter list must always be written even when empty:

```d
struct Stack(T = int)
{
// ...
}
// ...
    Stack!() stack;
```

> :warning: Every instantiation of a template for a given set of types is considered to be a distinct type.
> For example, `Stack!int` and `Stack!double` are separate types:
```d
Stack!int stack = Stack!double(0.25, 0.75); // ← compilation ERROR
```

TODO: Note-tip style
Templates are entirely a compile-time feature.
The instances of templates are generated by the compiler at compile time.

## Static If

`static if` is the compile time equivalent of the `if` statement.
Just like the `if` statement, `static if` takes a logical expression and evaluates it.
Unlike the `if` statement, `static if` is not about execution flow; rather, it determines whether a piece of code should be included in the program or not.
The logical expression must be evaluable at compile time.
If the logical expression evaluates to true, the code inside the static if gets compiled.
If the condition is false, the code is not included in the program as if it has never been written.
`static if` can appear at module scope or inside definitions of `struct`, `class`, `template`, etc.
Optionally, there may be else clauses as well.
Let's use `static if` with a simple template:

```d
enum Square;
enum Rectangle;

struct Parallelogram(T)
{
    int length;

    static if(is(T == Rectangle))
    {
        int width;
        this(int length, int width)
        {
            this.length = length;
            this.width = width;
        }
    }
    else
    {
        this(int length)
        {
            this.length = length;
        }
    }
}

void main()
{
    auto x = Parallelogram!(Rectangle)(2, 5);
    auto z = Parallelogram!(Square)(2);
}
```

For the two instantiations, the compiler will generate the following `struct` declarations:

```d
struct Parallelogram!Rectangle
{
    int length;
    int width;
    this(int length, int width)
    {
        this.length = length;
        this.width = width;
    }
}

struct Parallelogram!Square
{
    int length;
    this(int length)
    {
        this.length = length;
    }
}
```

## is Expression

```d
is (/* ... */) // is expression
```

The is expression is evaluated at compile time.
It produces an `int` value, either `0` or `1` depending on the expression specified in parentheses.
Although the expression that it takes is not a logical expression, the is expression itself is used as a compile time logical expression.
It is especially useful in static if conditionals and template constraints.
The condition that it takes is always about types, which must be written in one of several syntaxes.

### `1. is(T)`

The expression `is(T)` Determines whether `T` is a valid type.

```d
static if (is (int))     // passes
{
    writeln("valid");
} 

static if(is (asd))      // if asd is not a user defined type, the check will fail
{
    writeln("stop using such crippled names");
}
```

### `2. is(T Alias)`

The `is(T Alias)` expression works in the same way as the previous syntax: it checks if `T` is a valid type.
Additionally, it defines `Alias` as an alias of `T`:

```d
static if (is (int NewAlias))
{
    writeln("valid");
    NewAlias var = 42; // int and NewAlias are the same
} 
else
{
    writeln("invalid");
}
```

### `3. is(T : OtherT)`

The `is(T : OtherT)` expression determines whether `T` can automatically be converted to `OtherT`.
It is used to detect automatic type conversions:

```d
import std.stdio;
interface Clock
{
    void tellTime();
}
class AlarmClock : Clock
{
    override void tellTime()
    {
        writeln("10:00");
    }
}

void myFunction(T)(T parameter)
{
    static if (is (T : Clock))
    {
        // If we are here then T can be used as a Clock
        writeln("This is a Clock; we can tell the time");
        parameter.tellTime();
    }
    else
    {
        writeln("This is not a Clock");
    }
}
void main()
{
    auto var = new AlarmClock;
    myFunction(var);
    myFunction(42);
}
```

### `4. is(T Alias : OtherT)`

The `is(T Alias : OtherT)` expression works in the same way as the previous syntax.
It checks is `T` can be implicitly converted to `OtherT` and it defines `Alias` as an alias of `T`.

### `5. is(T == Specifier)`

The `is(T == Specifier)` expression determines whether `T` is the same type as `Specifier` or whether `T` matches that specifier.

When we change the previous example to use `==` instead of `:`, the condition would not be satisfied for `AlarmClock`:

```d
static if (is (T == Clock))
{
    writeln("This is a Clock; we can tell the time");
    parameter.tellTime();
}
else
{
    writeln("This is not a Clock");
}
```

Although `AlarmClock` is a `Clock` , it is not exactly the same type as `Clock`.
For that reason, now the condition is invalid for both `AlarmClock` and `int`.

These are the simplest uses of the is expression.
For an exhaustive guide to all the variations of is expression check this [link](https://dlang.org/spec/expression.html#is_expression).

## Typeof

`typeof` is a way to specify a type based on the type of an expression.
For example:

```d
void func(int i)
{
    typeof(i) j;       // j is of type int
    typeof(3 + 6.0) x; // x is of type double
    typeof(1)* p;      // p is of type pointer to int
    int[typeof(p)] a;  // a is of type int[int*]

    writefln("%d", typeof('c').sizeof); // prints 1
    double c = cast(typeof(1.0))j; // cast j to double
}
```

The expression is not evaluated, just the type of it is generated:

```d
void func()
{
    int i = 1;
    typeof(++i) j; // j is declared to be an int, i is not incremented
    writefln("%d", i);  // prints 1
}
```

## Template Constraints

The fact that templates can be instantiated with any argument yet not every argument is compatible with every template brings an inconvenience.
If a template argument is not compatible with a particular template, the incompatibility is necessarily detected during the compilation of the template code for that argument.
As a result, the compilation error points at a line inside the template implementation:

```d
struct A
{
    int doSomething() { return 42; }
}

void fun(T)(T a)
{
    a.doSomething();    // <--- error here
}

void main()
{
    A a;
    int b;
    fun(a);
    fun(b);
}
```

Compiling this code will issue an error due to the fact that `int` does not have a method `doSomething`.
The error will be issued after the code for `fun!int` was generated, during the semantical analysis phase, at the call to `doSomething`.
Imagine that `fun` was defined in a library and the user does not have access to the source code; in this situation it is hard to spot the exact bug, however using template constraints eases this process:

```d
struct A
{
    int doSomething() { return 42; }
}

void fun(T)(T a)
if(is(typeof(a.doSomething)))    // <--- error here
{
    a.doSomething();
}

void main()
{
    A a;
    int b;
    fun(a);
}
```

Template constraints have the same syntax as an if statement (without else).




