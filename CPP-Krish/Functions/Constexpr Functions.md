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
When a constexpr function is evaluated at compile-time, the compiler must be able to see the full definition of the constexpr function prior to such function calls (so it can perform the evaluation itself). A forward declaration will not suffice in this case, even if the actual function definition appears later in the same compilation unit. So constexpr functions are by defualt inline.
As a result, constexpr functions are often defined in header files, so they can be `#included` into any .cpp file that requires the full definition.

For constexpr function calls that are only evaluated at runtime, a forward declaration is sufficient to satisfy the compiler. This means you can use a forward declaration to call a constexpr function defined in another translation unit, but only if you invoke it in a context that does not require compile-time evaluation.

Per [CWG2166](https://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#2166), the actual requirement for the forward declaration of constexpr functions that are evaluated at compile-time is that “the constexpr function must be defined prior to the outermost evaluation that eventually results in the invocation”. Therefore, this is allowed:

```cpp
#include <iostream>

constexpr int foo(int);
constexpr int goo(int c) { return foo(c); } // note that foo is not defined yet
constexpr int foo(int b)  { return b; }
// okay because foo is still defined before any calls to goo

int main() {
	constexpr int a{ goo(5) }; // this is the outermost invocation
}
```

The intent here is to allow for mutually recursive constexpr functions (where two constexpr functions call each other), which would not be possible otherwise.

---