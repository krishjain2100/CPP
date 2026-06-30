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

---
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

---
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

Type deduction doesn’t work for function parameters, and prior to C++20, the above program won’t compile (you’ll get an error about function parameters not being able to have an auto type). In C++20, the `auto` keyword was extended so that the above program will compile and function correctly, however, `auto` is not invoking type deduction in this case. Rather, it is triggering `function templates` that was designed to actually handle such cases.

---
### Top/Low-Level Const

A **top-level const** is a const qualifier that applies to an object itself. For example:
1. `const int x;` : x is const and can't be changed
2. `int* const ptr` : ptr is const and can't be reseated.

In contrast, a **low-level const** is a const qualifier that applies to the object being referenced or pointed to. For example:
1. `const int& ref` : ref is const always but here is points to a const value
2. `const int* ptr` : ptr can itself be reseated

- References don't have a top-level const syntax, as they are implicitly top-level const. 
- A reference to a const value is always a low-level const.  
- A pointer can have a top-level, low-level, or both kinds of const

---
### Type deduction for references

In addition to dropping const, type deduction will also drop references. Just like with dropped `const`, if you want the deduced type to be a reference, you can reapply the reference at the point of definition:

```cpp
std::string& getRef(); // some function that returns a reference
auto ref1 { getRef() };  // std::string (reference dropped), ref1 is a copy
auto& ref2 { getRef() }; // std::string& (reference dropped, reference reapplied)

```

When we say that type deduction drops const qualifiers, it only drops top-level consts. 
Low-level consts are not dropped. 

**If the initializer is a reference to const, the reference is dropped first (and then reapplied if applicable), and then any top-level const is dropped from the result.**

```cpp
const std::string& getConstRef(); // some function that returns a const reference
int main() {
    auto ref1{ getConstRef() };         // std::string 
    const auto ref2{ getConstRef() };   // const std::string 
    auto& ref3{ getConstRef() };        // const std::string&
    const auto& ref4{ getConstRef() };  // const std::string&
}
```

- For `ref1`, the reference is dropped first, leaving us with a `const std::string`. This const is now a top-level const, so it is also dropped, leaving the deduced type as `std::string`.
- For `ref2`, this is similar to the `ref1` case, except we’re reapplying the `const` qualifier, so the deduced type is `const std::string`.
- For `ref3`,  normally the reference would be dropped first, but since we’ve reapplied the reference, it is not dropped. That means the type is still `const std::string&`. And since this const is a low-level const, it is not dropped. Thus the deduced type is `const std::string&`.
- The `ref4` case works similarly to `ref3`, except we’ve reapplied the `const` qualifier as well. Since the type is already deduced as a reference to const, us reapplying `const` here is redundant. 

Dropping a reference may change a low-level const to a top-level const: `const std::string&` is a low-level const, but dropping the reference yields `const std::string`, which is a top-level const.

---
### Constexpr is not part of an expression’s type, so it is not deduced by `auto`.

When defining a constexpr reference to a const variable (e.g. `constexpr const int&`), we need to apply both `constexpr` (which applies to the reference) and `const` (which applies to the type being referenced).


```cpp
constexpr std::string_view hello { "Hello" };   // implicitly const
constexpr const std::string_view& getConstRef() {
    return hello;
} // function is constexpr, returns a const std::string_view&

int main() {
    auto ref1{ getConstRef() };  
    // std::string_view (reference dropped and top-level const dropped)
    constexpr auto ref2{ getConstRef() };
     // constexpr const std::string_view (reference dropped and top-level const dropped, constexpr applied, implicitly const)

    auto& ref3{ getConstRef() }; 
     // const std::string_view& (reference reapplied, low-level const not dropped)
    constexpr const auto& ref4{ getConstRef() }; 
    // constexpr const std::string_view& (reference reapplied, low-level const not dropped, constexpr applied)
}
```

----
### Type deduction for pointers

Unlike references, type deduction does not drop pointers but we can also use an asterisk in conjunction with pointer type deduction (`auto*`) to make it clearer that the deduced type is a pointer:

```cpp
std::string* getPtr(); // some function that returns a pointer
auto ptr1{ getPtr() };  // std::string*
auto* ptr2{ getPtr() }; // std::string*
```


The reason that references are dropped during type deduction but pointers are not dropped is because references and pointers have different semantics. When we evaluate a reference, we’re really evaluating the object being referenced. Therefore, when deducing a type, it makes sense that we should deduce the type of the thing being referenced, not the reference itself. On the other hand, pointers hold the address of an object. When we evaluate a pointer, we are evaluating the pointer, not the object being pointed to. Therefore, it makes sense that we should deduce the type of the pointer, not the thing being pointed to.

---
#### The difference between auto and auto*

When we use `auto` with a pointer type initializer, the type deduced for `auto` includes the pointer. So for `ptr1` above, the type substituted for `auto` is `std::string*`.

When we use `auto*` with a pointer type initializer, the type deduced for auto does _not_ include the pointer , the pointer is reapplied afterward after the type is deduced. So for `ptr2` above, the type substituted for `auto` is `std::string`, and then the pointer is reapplied.


There are a couple of difference between `auto` and `auto*` in practice. 
First, `auto*` must resolve to a pointer initializer, otherwise a compile error will result:

```cpp

std::string* getPtr(); // some function that returns a pointer
auto ptr3{ *getPtr() }; // std::string (because we dereferenced getPtr())
auto* ptr4{ *getPtr() }; // does not compile (initializer not a pointer)
```

This makes sense: in the `ptr4` case, `auto` deduces to `std::string`, then the pointer is reapplied. Thus `ptr4` has type `std::string*`, and we can’t initialize a `std::string*` with an initializer that is not a pointer.

Second, there are differences in how `auto` and `auto*` behave when we introduce `const` into the equation. 

---
#### Type deduction for const pointers 

Just like with references, only top-level const is dropped during pointer type deduction.

```cpp
std::string* getPtr(); // some function that returns a pointer
int main() {
    const auto ptr1{ getPtr() };  // std::string* const
    auto const ptr2 { getPtr() }; // std::string* const
    const auto* ptr3{ getPtr() }; // const std::string*
    auto* const ptr4{ getPtr() }; // std::string* const
}
```

When we use either `auto const` or `const auto`, we’re saying, “make the deduced pointer a const pointer”. So in the case of `ptr1` and `ptr2`, the deduced type is `std::string*`, and then const is applied, making the final type `std::string* const`. This is similar to how `const int` and `int const` mean the same thing.

However, when we use `auto*`, the order of the const qualifier matters. A `const` on the left means “make the deduced pointer a pointer to const”, whereas a `const` on the right means “make the deduced pointer type a const pointer”. Thus `ptr3` ends up as a pointer to const, and `ptr4` ends up as a const pointer.


Now let’s look at an example where the initializer is a const pointer to const.

```cpp
std::string s{};
const std::string* const ptr { &s };

auto ptr1{ ptr };  // const std::string*
auto* ptr2{ ptr }; // const std::string*

auto const ptr3{ ptr };  // const std::string* const
const auto ptr4{ ptr };  // const std::string* const

auto* const ptr5{ ptr }; // const std::string* const
const auto* ptr6{ ptr }; // const std::string*

const auto const ptr7{ ptr };  // error: const qualifer can not be applied twice
const auto* const ptr8{ ptr }; // const std::string* const
```

For `ptr1` and `ptr2` , the top-level const (the const on the pointer itself) is dropped. The low-level const on the object being pointed to is not dropped. So in both cases, the final type is `const std::string*`.

For `ptr3` and `ptr4`, the top-level const is dropped, but we’re reapplying it. The low-level const on the object being pointed to is not dropped. So in both cases, the final type is `const std::string* const`.

The `ptr5` and `ptr6` cases are analogous to the cases we showed in the prior example. In both cases, the top-level const is dropped. For `ptr5`, the `auto* const` reapplies the top-level const, so the final type is `const std::string* const`. For `ptr6`, the `const auto*` applies const to the type being pointed to (which in this case was already const), so the final type is `const std::string*`.

In the `ptr7` case, we’re applying the const qualifier twice, which is disallowed, and will cause a compile error. 

And finally, in the `ptr8` case, we’re applying const on both sides of the pointer (which is allowed since `auto*` must be a pointer type), so the resulting types is `const std::string* const`.

Best practice; Consider using `auto*` when deducing a pointer type. Using `auto*` in this case makes it clearer that we are deducing a pointer type, enlists the compiler’s help to ensure we don’t deduce a non-pointer type, and gives you more control over const.

---
