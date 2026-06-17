**Why it exists:**
- Replace template meta-programming hacks
- Improve readability and error messages
- Enable zero-overhead abstractions

This allows the compiler to optimise code by calculating values before the program even runs.

- If you feed them **Constants** (like `5` or `const int`), they run at **Compile Time**.
- If you feed them **Variables** (like `x`), they run at **Runtime** just like a normal function.

```cpp
// 1. Compile-time function
constexpr int square(int x) {
    return x * x;
}

int main() {
    // Calculated at compile time. 
    // The compiled binary effectively just says "arr[25]".
    int arr[square(5)]; 
    // C-Style Arrays are strict:
    // The size of a raw array (`int arr[SIZE]`) must be known before the program ever runs.
}
```

#### Creating Types with `constexpr`

Writing code that writes code. Specifically, using numbers to change **Types**.

**1. Compile-Time Dimensions:** 

```cpp
using Matrix = std::array<int, square(N)>;
```

If `N=4`, `square(N)` is 16. The compiler creates a `std::array<int, 16>`. This is much faster than `std::vector` because it lives on the Stack, not the Heap.

**2. Branching Types (`std::conditional`):** This is a very cool optimisation technique. You can choose different variable types based on a number size.
- **Scenario:** You have a number `N`.
- If `N < 8`, it fits in a small byte (`uint8_t`).
- If `N >= 8`, you need a big integer (`uint64_t`).

```cpp
template <size_t N>
struct Storage {
    // std::conditional is a compile-time "if statement" for TYPES.
    // Format: conditional< Boolean, TypeIfTrue, TypeIfFalse >
    using type = std::conditional_t<
        (N < 8), // Condition
        uint8_t, // Result if True
        uint64_t, // Result if False
    >;
};
```

- **Why use this?** If you are writing a high-frequency trading platform or a game engine, you don't want to waste 64 bits of memory to store the number "5". This automates the memory optimisation for you.


The **As-If Rule** is the legal foundation for every single optimisation the C++ compiler performs.
The rule states: **The compiler is allowed to completely rewrite, reorder, or delete your code, as long as the observable behaviour of the final program is exactly as-if the original code was executed.**

```cpp
const double x { 1.2 };
const double y { 3.4 };
const double z { x + y }; // x + y may evaluate at runtime or compile-time
```

The expression `x + y` would normally evaluate at runtime, but since the value of `x` and `y` are known at compile-time, the compiler may opt to perform compile-time evaluation instead and initialize `z` with the compile-time calculated value `4.6`.

A few common cases where a compile-time evaluatable expression is required:
- The initialiser of a constexpr variable
- A non-type template argument 
- The defined length of a `std::array`  or a C-style array.


Expression: a non-empty sequence of literals, variables, operators, and function calls
Constant Expression: Each part of the expression must be evaluatable at compile-time.

Examples of constant expressions: 
- Literals (e.g. ‘5’)
- Operators with constant expression operands (e.g. `3 + 4`).
- Const **integral** variables with a constant expression initialiser (e.g. `const int x { 5 };`).
- Constexpr variables.
- Constexpr function calls with constant expression arguments

Notably, the following cannot be used in a constant expression:
- Non-const variables.
- Const non-integral variables (e.g. `const double d { 1.2 };`). To use such variables in a constant expression, define them as constexpr variables instead.
- The return values of non-constexpr functions
- Function parameters
- Operators `new`, `delete`, `throw`, `typeid`, and `operator,` (comma).


**Note:** When constant expressions were defined, `const` integral types were grandfathered in because they were already being treated as constant expressions within the language (to be able to use const variables to initialise C-style arrays). The committee discussed whether `const` non-integral types should also be treated as constant expressions. Ultimately, they decided not to, in order to promote more consistent usage of `constexpr`.

Since constant expressions are always capable of being evaluated at compile-time, you may have assumed that constant expressions will always be evaluated at compile-time. Counterintuitively, this is not the case.

**The compiler is only _required_ to evaluate constant expressions at compile-time in contexts that _require_ a constant expression.** But modern compilers will _usually_ evaluate a constant expression at compile-time when optimizations are enabled.

```cpp
const int x { 3 + 4 }; // constant expression 3 + 4 must be evaluated at compile-time
int y { 3 + 4 };       // constant expression 3 + 4 may be evaluated at compile-time or runtime
```

The likelihood that an expression is fully evaluated at compile-time can be categorized as follows:

- Never: A non-constant expression where the compiler is not able to determine all values at compile-time.
- Possibly: A non-constant expression where the compiler is able to determine all values at compile-time (optimized under the as-if rule).
- Likely: A constant expression used in a context that does not require a constant expression.
- Always: A constant expression used in a context that requires a constant expression.


So why doesn’t C++ require all constant expressions to be evaluated at compile-time? There are at least two good reasons:

1. Compile-time evaluation makes debugging harder. 
2. To provide the compiler with the flexibility to optimize as it sees fit (or as influenced by compiler options). For example, a compiler might want to offer an option that defers all non-required constant expression evaluation to runtime, in order to improve compile times for developers.

---
### How to create compile time constants

#### Issues with const

First, use of `const` does not make it immediately clear whether the variable is usable in a constant expression or not. In some cases, we can figure it out fairly easily:

```cpp
int a { 5 };       // not const at all
const int b { a }; // not a constant expression (since initializer is non-const)
const int c { 5 }; // a constant expression (since initializer is a constant expression)
const int d { someVar }; // not obvious whether d is usable in a constant expression or not
const int e { getValue() }; // not obvious whether e is usable in a constant expression or not
```

We have to go inspect the definitions of those initializers and infer what case we’re in. And that may not even be sufficient, if `someVar` is const and initialized with a variable or a function call, we’ll have to go inspect the definition of its initializer too.

Second, use of `const` does not provide a way to inform the compiler that we require a variable that is usable in a constant expression (and that it should halt compilation if it isn’t). Instead, it will just silently create a variable that can only be used in runtime expressions.

Third, the use of `const` to create compile-time constant variables does not extend to non-integral variables. And there are many cases where we would like non-integral variables to be compile-time constants too.

#### Constexpr

A **constexpr** variable is always a compile-time constant. As a result, a constexpr variable must be initialized with a constant expression, otherwise a compilation error will result. Example

```cpp
// The return value of a non-constexpr function is not constexpr
int five() { return 5; }

int main() {
    constexpr double gravity { 9.8 }; // ok: 9.8 is a constant expression
    constexpr int sum { 4 + 5 };      // ok: 4 + 5 is a constant expression
    constexpr int something { sum };  // ok: sum is a constant expression
    std::cout << "Enter your age: ";
    int age{};
    std::cin >> age;
    constexpr int myAge { age }; // compile error: age is not a constant expression
    constexpr int f { five() };  // compile error: return value of five() is not constexpr

    return 0;
}
```

Because functions normally execute at runtime (except [[Constexpr Functions]]), the return value of a function is not constexpr (even when the return expression is a constant expression). This is why `five()` is not a legal initialisation value for `constexpr int f`.

Unlike `const`, `constexpr` is not part of an object’s type. Therefore a variable defined as `constexpr int` actually has type `const int` (due to the implicit `const` that `constexpr` provides for objects).

**Note:** Some types that are not fully compatible with `constexpr` (including `std::string`, `std::vector`, and other types that use dynamic memory allocation). For constant objects of these types, either use `const` instead of `constexpr`, or pick a different type that is constexpr compatible (e.g. `std::string_view` or `std::array`).

---
