There is no way to tell the compiler that a constexpr function should prefer to evaluate at compile-time whenever it can. However, we can force a constexpr function that is eligible to be evaluated at compile-time to actually evaluate at compile-time by ensuring the return value is used where a constant expression is required. This needs to be done on a per-call basis.

The most common way to do this is to use the return value to initialize a constexpr variable. Unfortunately, this requires introducing a new variable into our program just to ensure compile-time evaluation, which is ugly and reduces code readability.

C++20 introduced `consteval` functions, which is used to indicate that a function _must_ evaluate at compile-time, otherwise a compile error will result. 

```cpp
consteval int greater(int x, int y) {
    return (x > y ? x : y);
}
int main() {
    std::cout << greater(5, 6) << " is greater!\n"; 
    // ok: will evaluate at compile-time
    int x{ 5 }; // not constexpr
    std::cout << greater(x, 6) << " is greater!\n"; 
    // error: consteval functions must evaluate at compile-time
}
```

Perhaps surprisingly, the parameters of a consteval function are not constexpr (even though consteval functions can only be evaluated at compile-time). This decision was made for the sake of consistency.


---
### `std::is_constant_evaluated` and `if consteval`
Neither of these, tell you whether a function call is evaluating at compile-time or runtime

`std::is_constant_evaluated()`, defined in the`<type_traits>` header,  returns a `bool` indicating whether the current function is executing in a constant-evaluated context. A **constant-evaluated context** (also called a **constant context**) is defined as one in which a constant expression is required (such as the initialization of a constexpr variable). So in cases where the compiler is required to evaluate a constant expression at compile-time `std::is_constant_evaluated()` will `true` as expected.

This is intended to allow you to do something like this:

```cpp
#include <type_traits> 
constexpr int someFunction() {
    if (std::is_constant_evaluated()) // if evaluating in constant context
        doSomething();
    else
        doSomethingElse();
}
```

However, the compiler may also choose to evaluate a constexpr function at compile-time in a context that does not require a constant expression. In such cases, it will return `false` even though the function did evaluate at compile-time. So it really means “the compiler is being forced to evaluate this at compile-time”, not “this is evaluating at compile-time”.

There are several reasons for such a behaviour:
1. The standard doesn’t actually make a distinction between compile time and runtime. Defining behaviour involving that distinction would have been a larger change.
2. Optimizations should not change the observable behavior of a program (unless explicitly allowed by the standard). If `std::is_constant_evaluated()` were to return `true` when the function was evaluated at compile-time for any reason, then the optimizer deciding to evaluate a function at compile-time instead of runtime could potentially change the observable behavior of the function. As a result, your program might behave very differently depending on what optimization level it was compiled with.


Introduced in C++23, `if consteval` is a replacement for the following. It provides a nicer syntax and fixes some other issues. However, it evaluates the same way.

```cpp
if (std::is_constant_evaluated())
```

---
### Forcing compile time evaluation

It would still be useful to have a convenient way to force constexpr functions to evaluate at compile-time, so that we can explicitly force compile-time evaluation when possible, and runtime evaluation when we can’t. Here’s an example that shows how this is possible:


```cpp
#define CONSTEVAL(...) [] consteval { return __VA_ARGS__; }()               // C++20 version per Jan Scultke (https://stackoverflow.com/a/77107431/460250)
#define CONSTEVAL11(...) [] { constexpr auto _ = __VA_ARGS__; return _; }() // C++11 version per Justin (https://stackoverflow.com/a/63637573/460250)

constexpr int compare(int x, int y)  {
    if (std::is_constant_evaluated()) return (x > y ? x : y);
    else return (x < y ? x : y);
}

int main() {
    int x { 5 };
	cout << compare(x, 6) << '\n';  
    // will execute at runtime and return 5
	cout << compare(5, 6) << '\n';  
    // may or may not execute at compile-time, but will always return 5
	cout << CONSTEVAL(compare(5, 6)) << '\n'; 
    // will always execute at compile-time and return 6
}
```

This uses a variadic preprocessor macro (the `#define`, `...`, and `__VA_ARGS__`) to define an consteval lambda that is immediately invoked (by the trailing parentheses).  
You can find information on variadic macros at [https://en.cppreference.com/w/cpp/preprocessor/replace](https://en.cppreference.com/w/cpp/preprocessor/replace).  
We cover lambdas in lesson [20.6 -- Introduction to lambdas (anonymous functions)](https://www.learncpp.com/cpp-tutorial/introduction-to-lambdas-anonymous-functions/).

The following should also work (and is a bit cleaner since it doesn’t use preprocessor macros):
There is a bug in GCC 14 onward that causes the following example to produce the wrong answer when any level of optimization is enabled.

```cpp
consteval auto CONSTEVAL(auto value) { return value; }

constexpr int compare(int x, int y)  {
    if (std::is_constant_evaluated()) return (x > y ? x : y);
    else return (x < y ? x : y);
}

int main() {
    std::cout << CONSTEVAL(compare(5, 6)) << '\n';      
     // will execute at compile-time
}
```

Because the arguments of consteval functions are always manifestly constant evaluated, if we call a constexpr function as an argument to a consteval function, that constexpr function must be evaluated at compile-time! The consteval function then returns the result of the constexpr function as its own return value, so the caller can use it.

Note that the consteval function returns by value. While this might be inefficient to do at runtime (if the value was some type that is expensive to copy, e.g. `std::string`), in a compile-time context, it doesn’t matter because the entire call to the consteval function will simply be replaced with the calculated return value.

---