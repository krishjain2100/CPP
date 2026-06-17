A **constexpr function** is a function that is allowed to be called in a constant expression.
Constexpr functions can be evaluated at compile time.

```cpp
constexpr double calcCircumference(double radius) {
    constexpr double pi { 3.14159265359 };
    return 2.0 * pi * radius;
}

int main(){
    constexpr double circumference { calcCircumference(3.0) }; // compiles
    std::cout << "Our circle has circumference " << circumference << "\n";
}
```

When a function call is evaluated at compile-time, the compiler will calculate the return value of the function call at compile-time, and then replace the function call with the return value.

To evaluate at compile-time, two other things must also be true:
- The call to the constexpr function must have arguments that are known at compile time (e.g. are constant expressions).
- All statements and expressions within the constexpr function must be evaluatable at compile-time.
- There are some other lesser encountered criteria as well. These can be found [here](https://en.cppreference.com/w/cpp/language/constexpr)

---
Constexpr functions can also be evaluated at runtime, in which case they will return a non-constexpr result. For example:

```cpp
constexpr int greater(int x, int y) { return (x > y ? x : y);}
int main(){
    int x{ 5 }; // not constexpr
    int y{ 6 }; // not constexpr
    std::cout << greater(x, y) << " is greater!\n"; // evaluated at runtime
}
```

---
Any constexpr function call that is part of a non-required constant expression may be evaluated at either compile-time or runtime.

```cpp
constexpr int getValue(int x) { return x; }

int main() {
    int x { getValue(5) }; // may evaluate at runtime or compile-time
}
```

Because `getValue()` is constexpr, the call `getValue(5)` is a constant expression. However, because variable `x` is not constexpr, it does not require a constant expression initialiser. So the compiler is free to choose whether `getValue(5)` evaluates at runtime or compile-time.

---
Example of a constexpr function that compiles successfully for runtime use, but then fails to compile when evaluated at compile-time.

```cpp
int getValue(int x) { return x; }
constexpr int foo(int x) {
    if (x < 0) return 0; // needed prior to adoption of P2448R1 in C++23 (see note below)
    return getValue(x);  // call to non-constexpr function here
}

int main() {
    int x { foo(5) }; // okay: will evaluate at runtime
    constexpr int y { foo(5) }; // compile error: foo(5) can't evaluate at compile-time
}
```

Prior to C++23, if no argument values exist that would allow a constexpr function to be evaluated at compile-time, the program is ill-formed (no diagnostic required). Without the line `if (x < 0) return 0`, the above example would contain no set of arguments that allow the function to be evaluatable at compile-time, making the program ill-formed. Given that no diagnostic is required, the compiler may not enforce this. This requirement was revoked in C++23 ([P2448R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2448r1.html)).

---

The parameters of a constexpr function are not implicitly constexpr, nor may they be declared as `constexpr`. Because such parameters are not constexpr, they cannot be used in constant expressions within the function.  Even `const` function parameters are treated as runtime constants 

```cpp
// c is not constexpr, and cannot be used in constant expressions
consteval int goo(int c)  { return c; }

 // b is not constexpr, and cannot be used in constant expressions
constexpr int foo(int b)    {
    constexpr int b2 { b }; 
    // compile error: constexpr variable requires constant expr initializer
    return goo(b);  
    // compile error: consteval function call requires constant expr argument
}

int main() {
    constexpr int a { 5 };
    std::cout << foo(a); 
    // okay: constant exp a can be used as argument to constexpr function foo()

}
```


If you need parameters that are constant expressions, see [Non-type template parameters](https://www.learncpp.com/cpp-tutorial/non-type-template-parameters/)

---
When a constexpr function is evaluated at compile-time, the compiler must be able to see the full definition of the constexpr function prior to such function calls (so it can perform the evaluation itself). A forward declaration will not suffice in this case, even if the actual function definition appears later in the same compilation unit.

This means that a constexpr function called in multiple files needs to have its definition included into each translation unit -- which would normally be a violation of the one-definition rule. To avoid such problems, constexpr functions are implicitly inline, which makes them exempt from the one-definition rule.

As a result, constexpr functions are often defined in header files, so they can be #included into any .cpp file that requires the full definition.

Rule

The compiler must be able to see the full definition of a constexpr (or consteval) function, not just a forward declaration.

Best practice

Constexpr/consteval functions used in a single source file (.cpp) should be defined in the source file above where they are used.

Constexpr/consteval functions used in multiple source files should be defined in a header file so they can be included into each source file.

For constexpr function calls that are only evaluated at runtime, a forward declaration is sufficient to satisfy the compiler. This means you can use a forward declaration to call a constexpr function defined in another translation unit, but only if you invoke it in a context that does not require compile-time evaluation.

![Ezoic](https://go.ezodn.com/utilcave_com/ezoic.png "ezoic")

For advanced readers

Per [CWG2166](https://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#2166), the actual requirement for the forward declaration of constexpr functions that are evaluated at compile-time is that “the constexpr function must be defined prior to the outermost evaluation that eventually results in the invocation”. Therefore, this is allowed:

```cpp
#include <iostream>

constexpr int foo(int);

constexpr int goo(int c)
{
	return foo(c);   // note that foo is not defined yet
}

constexpr int foo(int b) // okay because foo is still defined before any calls to goo
{
	return b;
}

int main()
{
	 constexpr int a{ goo(5) }; // this is the outermost invocation

	return 0;
}
```

The intent here is to allow for mutually recursive constexpr functions (where two constexpr functions call each other), which would not be possible otherwise.

Recap

Marking a function as `constexpr` means it can be used in a constant expression. It does not mean “will evaluate at compile-time”.

A constant expression (which may contain constexpr function calls) is only required to evaluate at compile-time in contexts where a constant expression is required.

In contexts that do not require a constant expression, the compiler may choose whether to evaluate a constant expression (which may contain constexpr function calls) at compile-time or at runtime.

A runtime (non-constant) expression (which may contain constexpr function calls or non-constexpr function calls) will evaluate at runtime.

Another example

Let’s do another examine to explore how a constexpr function is required or likely to evaluate further:

![Ezoic](https://go.ezodn.com/utilcave_com/ezoic.png "ezoic")

```cpp
#include <iostream>

constexpr int greater(int x, int y)
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int g { greater(5, 6) };              // case 1: always evaluated at compile-time
    std::cout << g << " is greater!\n";

    std::cout << greater(5, 6) << " is greater!\n"; // case 2: may be evaluated at either runtime or compile-time

    int x{ 5 }; // not constexpr but value is known at compile-time
    std::cout << greater(x, 6) << " is greater!\n"; // case 3: likely evaluated at runtime

    std::cin >> x;
    std::cout << greater(x, 6) << " is greater!\n"; // case 4: always evaluated at runtime

    return 0;
}
```

In case 1, we’re calling `greater()` in a context that requires a constant expression. Thus `greater()` must be evaluated at compile-time.

In case 2, the `greater()` function is being called in a context that does not require a constant expression, as output statements must execute at runtime. However, since the arguments are constant expressions, the function is eligible to be evaluated at compile-time. Thus the compiler is free to choose whether this call to `greater()` will be evaluated at compile-time or runtime.

In case 3, we’re calling `greater()` with one argument that is not a constant expression. So this will typically execute at runtime.

However, this argument has a value that is known at compile-time. Under the as-if rule, the compiler could decide to treat the evaluation of `x` as a constant expression, and evaluate this call to `greater()` at compile-time. But more likely, it will evaluate it at runtime.

Related content

We cover the as-if rule in lesson [5.5 -- Constant expressions](https://www.learncpp.com/cpp-tutorial/constant-expressions/).

Note that even non-constexpr functions could be evaluated at compile-time under the as-if rule!

In case 4, the value of argument `x` can’t be known at compile-time, so this call to `greater()` will always evaluate at runtime.

![Ezoic](https://go.ezodn.com/utilcave_com/ezoic.png "ezoic")

Key insight

Put another way, we can categorize the likelihood that a function will actually be evaluated at compile-time as follows:

Always (required by the standard):

- Constexpr function is called where constant expression is required.
- Constexpr function is called from other function being evaluated at compile-time.

Probably (there’s little reason not to):

- Constexpr function is called where constant expression isn’t required, all arguments are constant expressions.

Possibly (if optimized under the as-if rule):

- Constexpr function is called where constant expression isn’t required, some arguments are not constant expressions but their values are known at compile-time.
- Non-constexpr function capable of being evaluated at compile-time, all arguments are constant expressions.

Never (not possible):

- Constexpr function is called where constant expression isn’t required, some arguments have values that are not known at compile-time.

Note that your compiler’s optimization level setting may have an impact on whether it decides to evaluate a function at compile-time or runtime. This also means that your compiler may make different choices for debug vs. release builds (as debug builds typically have optimizations turned off).

For example, both gcc and Clang will not compile-time evaluate a constexpr function called where a constant expression isn’t required unless the compiler told to optimize the code (e.g. using the `-O2` compiler option).

For advanced readers

The compiler might also choose to inline a function call, or even optimize a function call away entirely. Both of these can affect when (or if) the content of the function call are evaluated.