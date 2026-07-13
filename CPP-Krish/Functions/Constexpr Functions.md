
One challenge with constant expressions is that function call to a normal function are not allowed in constant expressions. This means we cannot use function calls anywhere a constant expression is required.

A **constexpr function** is a function that is allowed to be called in a constant expression.
Constexpr functions can be evaluated at compile time. 
If a required constant expression contains a constexpr function call, that constexpr function call must evaluate at compile-time.
Also if a constexpr function is called from other function being evaluated at compile-time then the constexpr function must evaluate at compile-time.

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

Constexpr functions can also be evaluated at runtime, in which case they will return a non-constexpr result. For example:

```cpp
constexpr int greater(int x, int y) { return (x > y ? x : y);}
int main(){
    int x{ 5 }; // not constexpr
    int y{ 6 }; // not constexpr
    std::cout << greater(x, y) << " is greater!\n"; // evaluated at runtime
}
```

Allowing functions with a constexpr return type to be evaluated at either compile-time or runtime was allowed so that a single function can serve both cases. Otherwise, you’d need to have separate functions (a function with a constexpr return type, and a function with a non-constexpr return type). This would not only require duplicate code, the two functions would also need to have different names.

Any constexpr function call that is part of a non-required constant expression may be evaluated at either compile-time or runtime. Therefore, compile time evaluation of constexpr functions that can be resolved at compile time is not mandated and depends on the level of optimization used. This is because the compiler is _not_ required to determine whether a constexpr function is evaluatable at compile-time until it is actually evaluated at compile-time.

Example of a constexpr function that compiles successfully for runtime use, but then fails to compile when evaluated at compile-time.

```cpp
int getValue(int x) { return x; }
constexpr int foo(int x) {
    if (x < 0) return 0; // needed prior to adoption of P2448R1 in C++23 (see note below)
    return getValue(x);  // call to non-constexpr function here
}

int main() {
    int x { foo(5) }; // okay: will evaluate at runtime
    constexpr int y { foo(5) }; 
    // compile error: foo(5) can't evaluate at compile-time
}
```

As an aside: Prior to C++23, the C++ standard says that a constexpr function must return a constexpr value for at least one set of arguments, otherwise it is technically ill-formed. Calling a non-constexpr function unconditionally in a constexpr function makes the constexpr function ill-formed. However, compilers are not required to generate errors or warnings for such cases, therefore, the compiler probably won’t complain unless you try to call such a constexpr function in a constant context. In C++23, this requirement was rescinded.

So all constexpr functions should be evaluatable at compile-time, as they will be required to do so in contexts that require a constant expression.

C++ does not currently provide any reliable mechanisms to determine if a constexpr function call is evaluating at compile-time or runtime.

---
### Parameters of Constexpr/Consteval Functions

The parameters of a constexpr function are not implicitly constexpr, nor may they be declared as `constexpr`. Because such parameters are not constexpr, they cannot be used in constant expressions within the function.  Even `const` function parameters are treated as runtime constants. If you need parameters that are constant expressions, use Non-type template parameters.

---
### Constexpr Functions are `inline` by default

When a constexpr function is evaluated at compile-time, the compiler must be able to see the full definition of the constexpr function prior to such function calls (so it can perform the evaluation itself). A forward declaration will not suffice in this case, even if the actual function definition appears later in the same compilation unit. So constexpr functions are by default inline.
As a result, constexpr functions are often defined in header files, so they can be `#included` into any .cpp file that requires the full definition.

Per [CWG2166](https://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#2166), the actual requirement for the forward declaration of constexpr functions that are evaluated at compile-time is that “the constexpr function must be defined prior to the outermost evaluation that eventually results in the invocation”. Therefore, this is allowed:

```cpp
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
### Constexpr/consteval functions can use non-const local variables

Within a constexpr or consteval function, we can use local variables that are not constexpr, and the value of these variables can be changed.

```cpp
consteval int doSomething(int x, int y)  {
    x = x + 2;       
    int z { x + y };
    if (x > y) z = z - 1;  
    return z;
}

int main() {
    constexpr int g { doSomething(5, 6) };
    std::cout << g << '\n';
}
```

When such functions are evaluated at compile-time, the compiler will essentially “execute” the function and return the calculated value.

---
### Constexpr/Consteval functions can use function parameters and local variables as arguments in constexpr function calls

Above, we noted, “When a constexpr (or consteval) function is being evaluated at compile-time, any other functions it calls are required to be evaluated at compile-time.”

A constexpr or consteval function can use its function parameters (which aren’t constexpr) or even local variables (which may not be const at all) as arguments in a constexpr function call. When a constexpr or consteval function is being evaluated at compile-time, the value of all function parameters and local variables must be known to the compiler (otherwise it couldn’t evaluate them at compile-time). Therefore, in this specific context, C++ allows these values to be used as arguments in a call to a constexpr function, and that constexpr function call can still be evaluated at compile-time.

```cpp
constexpr int goo(int c) { return c; }
constexpr int foo(int b) { return goo(b); }
 // if foo() is resolved at compile-time, 
 // then `goo(b)` can also be resolved at compile-time

std::cout << foo(5);
```

In the above example, `foo(5)` may or may not be evaluated at compile time. If it is, then the compiler knows that `b` is `5`. And even though `b` is not constexpr, the compiler can treat the call to `goo(b)` as if it were `goo(5)` and evaluate that function call at compile-time. 

---
### Can a constexpr function call a non-constexpr function?

Yes, when the constexpr function is being evaluated in a non-constant context. 
No, otherwise, and doing so will produce a compilation error.


```cpp
#include <type_traits> // for std::is_constant_evaluated
constexpr int someFn() {
    if (std::is_constant_evaluated()) return someConstexprFcn();
    else return someNonConstexprFcn();
}
```

Now consider this variant:

```cpp
constexpr int someFn(bool b) {
    if (b) return someConstexprFcn();
    else return someNonConstexprFcn();
}
// This is legal as long as `someFn(false)` is never called in a constant expr.

```

---

A **pure function** is a function that:
- Always returns the same return result when given the same arguments
- The function has no side effects (e.g. it doesn’t change the value of static local or global variables, doesn’t do input or output, etc…).
Pure functions should generally be made constexpr.

Prior to C++23, writing a non-const `static` local variable inside a `constexpr` function resulted in an immediate compiler error. 

C++23 relaxed this restriction. You are now allowed to declare non-literal and `static` variables inside a `constexpr` function, but their behavior depends entirely on the execution context.
- **At Runtime:** If the `constexpr` function is called during standard runtime execution, it can use and modify the `static` local variable. 
- **At Compile-Time:** If you attempt to evaluate the function in a constant expression context, and the control flow reaches the non-const `static` local variable, it will give compile error (as now do we continue with modified value in to runtime or reset it again ? ).

Also, inside constexpr function:
- non-const globals cannot be read or modified (or else order of compile time execution would start to matter).
- const globals with constexpr initializers or constexpr variables can be read, but cannot be modified (const can never be modified).
- const globals with non-constexpr cannot be read at compile time obviously.

---
### Why not constexpr every function?

There are a few reasons you may not want to `constexpr` a function:
1. `constexpr` signals that a function can be used in a constant expression. If your function cannot be evaluated as part of a constant expression, it should not be marked as `constexpr`.
2. `constexpr` is part of the interface of a function. Once a function is made constexpr, it can be called by other constexpr functions or used in contexts that require constant expressions. Removing the `constexpr` later will break such code.
3. `constexpr` functions can be harder to debug since you can’t breakpoint or step through them in a debugger.

---
### Why constexpr a function when it is not actually evaluated at compile-time?

Unless you have a specific reason not to, a function that can be evaluated as part of a constant expression should be made `constexpr` (even if it isn’t currently used that way).

There are a few reasons:
1. There’s little downside to using constexpr, and it may help the compiler optimize your program to be smaller and faster.
2. You may use it in constant context somewhere later, and may miss out on performance benefits, as you may or may not remember to make it constexpr.

---