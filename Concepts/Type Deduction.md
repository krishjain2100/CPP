
### Type deduction for initialized variables

**Type deduction** (also sometimes called **type inference**) is a feature that allows the compiler to deduce the type of an object from the object’s initializer. 
```cpp
auto d { 5.0 }; // 5.0 is a double literal, so d will be deduced as a double
auto i { 1+2 }; // 1+2 evaluates to an int, so i will be deduced as an int
auto x { i }; // i is an int, so x will be deduced as an int
```

Because function calls are valid expressions, we can even use type deduction when our initializer is a non-void function call:

```cpp
int add(int x, int y) { return x + y; }
int main() {
    auto sum { add(5, 6) }; 
    // add() returns an int, so sum's type will be deduced as an int
}
```

Literal suffixes can be used in combination with type deduction to specify a particular type:

```cpp
auto a { 1.23f }; // f suffix causes a to be deduced to float
auto b { 5u };    // u suffix causes b to be deduced to unsigned int

```

Variables using type deduction may also use other specifiers/qualifiers, such as `const` or `constexpr`:

```cpp
int a { 5 };            // a is an int
const auto b { 5 };     // b is a const int
constexpr auto c { 5 }; // c is a constexpr int
```

Type deduction must have something to deduce from:  It will not work for objects that either do not have initializers or have empty initializers. It also will not work when the initializer has type `void` (or any other incomplete type). Thus, the following is not valid:

```cpp
void foo() { }
int main() {
    auto a;           // The compiler is unable to deduce the type of a
    auto b { };       // The compiler is unable to deduce the type of b
    auto c { foo() }; // Invalid: c can't have type incomplete type void
}
```

In most cases, type deduction will drop the `const` from deduced types. For example:

```cpp
const int a { 5 }; // a has type const int
auto b { a };      // b has type int (const dropped)
```

If you want a deduced type to be const, you must supply the `const` yourself as above.

For historical reasons, string literals in C++ have a strange type. Therefore, the following probably won’t work as expected:

```cpp
auto s { "Hello, world" }; // s will be type const char*, not std::string
```

If you want the type deduced from a string literal to be `std::string` or `std::string_view`, you’ll need to use the `s` or `sv` literal suffixes

---
### Type deduction for functions

Since the compiler already has to deduce the return type from the return statement (to ensure that the value can be converted to the function’s declared return type), in C++14, the `auto` keyword was extended to do function return type deduction. 

```cpp
auto add(int x, int y) {
    return x + y;
} // int + int will give int
```

When using an `auto` return type, all return statements within the function must return values of the same type, otherwise an error will result. 

```cpp
auto someFcn(bool b) {
    if (b) return 5; // return type int
    else return 6.7; // return type double
}
```

If such a case is desired for some reason, you can either explicitly specify a return type for your function (in which case the compiler will try to implicitly convert any non-matching return expressions to the explicit return type), or you can explicitly convert all of your return statements to the same type. 

#### Upsides of return type deduction

The biggest advantage of return type deduction is that having the compiler deduce the function’s return type negates the risk of a mismatched return type (preventing unexpected conversions). This can be particularly useful when a function’s return type is fragile (cases where return type is likely to change if the implementation changes). In such cases, being explicit about the return type means having to update all relevant return types when an impacting change is made to the implementation. If we’re lucky, the compiler will error until we update the relevant return types. If we’re not lucky, we’ll get implicit conversions where we don’t desire them.

In other cases, the return type of a function may either be long and complex, or not be that obvious. In such cases, `auto` can be used to simplify:

```cpp
// let compiler determine the return type of unsigned short + char
auto add(unsigned short x, char y) {
    return x + y;
}
```

We discuss this case a bit more (and how to express the actual return type of such a function) in lesson [11.8 -- Function templates with multiple template types](https://www.learncpp.com/cpp-tutorial/function-templates-with-multiple-template-types/).

#### Downside of return type deduction

Functions that use an `auto` return type must be fully defined before they can be used (a forward declaration is not sufficient). This makes sense: a forward declaration does not have enough information for the compiler to deduce the function’s return type. For example:

```cpp
auto foo();
int main() {
    std::cout << foo() << '\n'; 
    // the compiler has only seen a forward declaration at this point
}
auto foo() { return 5; }
```

Compiler error: error C3779: 'foo': a function that returns 'auto' cannot be used before it is defined.

### Trailing return type syntax

The `auto` keyword can also be used to declare functions using a **trailing return syntax**, where the return type is specified after the rest of the function prototype.

```cpp
int add(int x, int y) {
  return (x + y);
}

// Using the trailing return syntax, this could be equivalently written as:

auto add(int x, int y) -> int {
  return (x + y);
}
```

In this case, `auto` does not perform type deduction,  it is just part of the syntax to use a trailing return type. 

Here are some reasons to use this:

1. For functions with complex return types, a trailing return type can make the function easier to read:
```cpp
#include <type_traits> // for std::common_type
std::common_type_t<int, double> compare(int, double);// harder to read
auto compare(int, double) -> std::common_type_t<int, double>; // easier to read (we don't have to read the return type unless we care)
```

2. The trailing return type syntax can be used to align the names of your functions, which makes consecutive function declarations easier to read:
```cpp
auto add(int x, int y) -> int;
auto divide(double x, double y) -> double;
auto printSomething() -> void;
auto generateSubstring(const std::string &s, int start, int len) -> std::string;
```

3. If we have a function whose return type must be deduced based on the type of the function parameters, a normal return type will not suffice, because the compiler has not yet seen the parameters at that point.

```cpp
#include <type_traits>
// note: decltype(x) evaluates to the type of x

std::common_type_t<decltype(x), decltype(y)> add(int x, double y);         
// Compile error: compiler hasn't seen definitions of x and y yet
auto add(int x, double y) -> std::common_type_t<decltype(x), decltype(y)>; // ok
```

4. The trailing return syntax is also required for some advanced features, such as lambdas (which we cover in lesson [20.6 -- Introduction to lambdas (anonymous functions)](https://www.learncpp.com/cpp-tutorial/introduction-to-lambdas-anonymous-functions/)).


#### Type deduction can’t be used for function parameter types

Many new programmers who learn about type deduction try something like this:

```cpp
#include <iostream>
void addAndPrint(auto x, auto y) {
    std::cout << x + y << '\n';
}

int main() {
    addAndPrint(2, 3); // case 1: call addAndPrint with int parameters
    addAndPrint(4.5, 6.7); // case 2: call addAndPrint with double parameters
}
```

Type deduction doesn’t work for function parameters, and prior to C++20, the above program won’t compile (you’ll get an error about function parameters not being able to have an auto type). In C++20, the `auto` keyword was extended so that the above program will compile and function correctly, however, `auto` is not invoking type deduction in this case. Rather, it is triggering a different feature called `function templates` that was designed to actually handle such cases.

We introduce function templates in lesson [11.6 -- Function templates](https://www.learncpp.com/cpp-tutorial/function-templates/), and discuss use of `auto` in the context of function templates in lesson [11.8 -- Function templates with multiple template types](https://www.learncpp.com/cpp-tutorial/function-templates-with-multiple-template-types/).

---
