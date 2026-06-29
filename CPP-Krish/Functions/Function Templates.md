In C++, the template system was designed to simplify the process of creating functions (or classes) that are able to work with different data types.

Just like a normal definition, a **template** definition describes what a function or class looks like. Unlike a normal definition (where all types must be specified), in a template we can use one or more placeholder types. A placeholder type represents some type that is not known at the time the template is defined, but that will be provided later (when the template is used).

Once a template is define/d, the compiler can use the template to generate as many overloaded functions (or classes) as needed, each using different actual types.

The end result is, we end up with a bunch of mostly-identical functions or classes (one for each set of different types). But we only have to create and maintain a single template, and the compiler does all the hard work to create the rest for us.

Templates can work with types that didn’t even exist when the template was written. This helps make template code both flexible and future proof.

---
### Function templates

The initial function template that is used to generate other functions is called the **primary template**, and the functions generated from the primary template are called **instantiated functions**.

When we create a primary function template, we use **placeholder types** (technically called **type template parameters**, informally called **template types**) for any parameter types, return types, or types used in the function body that we want to be specified later, by the user of the template.

C++ supports 3 different kinds of template parameters:
- Type template parameters (where the template parameter represents a type).
- Non-type template parameters (where the template parameter represents a constexpr value).
- Template template parameters (where the template parameter represents a template).

```cpp
template <typename T>  // this is the template parameter declaration defining T as a type template parameter
T max(T x, T y){ // this is the function template definition for max<T>
    return (x < y) ? y : x;
}
```

The scope of a template parameter declaration is strictly limited to the function template (or class template) that follows. Therefore, each function template or class template needs its own template parameter declaration.

In our template parameter declaration, we start with the keyword `template`, which tells the compiler that we’re creating a template. Next, we specify all of the template parameters that our template will use inside angled brackets (`<>`). For each type template parameter, we use the keyword `typename` (preferred) or `class`, followed by the name of the type template parameter (e.g. `T`).

There is no difference between the `typename` and `class` keywords in this context. You will often see people use the `class` keyword since it was introduced into the language earlier. However, we prefer the newer `typename` keyword, because it makes it clearer that the type template parameter can be replaced by any type (such as a fundamental type), not just class types.

When the compiler encounters the function call `max<int>(1, 2)`, it will determine that a function definition for `max<int>(int, int)` does not already exist. Consequently, the compiler will implicitly use our `max<T>` function template to create one.

---
#### Template Instantiation

The process of creating functions (with specific types) from function templates (with template types) is called **function template instantiation**. 
When a function is instantiated due to a function call, it’s called **implicit instantiation**. A function that is instantiated from a template is called a **function instance**.
The template from which a function instance is produced is called a **primary template**. 
Function instances are normal functions in all regards.

Example, showing how the code looks after all instantiations are done by the compiler

```cpp
// a declaration for our function template 
// (we don't need the definition any more)
template <typename T>
T max(T x, T y);

template<>//ignore this for now
int max<int>(int x, int y) { // the generated function max<int>(int, int)
    return (x < y) ? y : x;
}

int main() {
    std::cout << max<int>(1, 2) << '\n'; 
    // instantiates and calls function max<int>(int, int)
}
```

A function template is only instantiated the first time a function call is made in each translation unit. Further calls to the function are routed to the already instantiated function.
Conversely, if no function call is made to a function template, the function template won’t be instantiated in that translation unit.

---
#### Template Argument Deduction

In cases where the type of the arguments match the actual type we want, we do not need to specify the actual type(in angular bracket), instead, we can use **template argument deduction** to have the compiler deduce the actual type that should be used from the argument types in the function call.

```cpp
std::cout << max<int>(1, 2) << '\n'; 
// the bottom two can replace the call above
std::cout << max<>(1, 2) << '\n';
std::cout << max(1, 2) << '\n';
```

- In `max<>(1, 2)`, the compiler will only consider `max<int>` template function overloads when determining which overloaded function to call.
- In max(1, 2), it will consider both `max` non-template function overloads and  `max<int>` template function overloads.  When the bottom case results in both a template function and a non-template function that are equally viable, the non-template function will be preferred.

In most cases, this normal function call syntax will be the one we use to call functions instantiated from a function template. There are a few reasons for this:
- It’s rare that we’ll have both a matching non-template function and a function template.
- If we do have a matching non-template function and a matching function template, we will usually prefer the non-template function to be called.

---
### Function templates with non-template parameters

It’s possible to create function templates that have both template parameters and non-template parameters. The type template parameters can be matched to any type, and the non-template parameters work like the parameters of normal functions. Note that the return type can also be any type. 

```cpp
// T is a type template parameter
// double is a non-template parameter
// We don't need to provide names for these parameters since they aren't used
template <typename T>
int someFcn(T, double) { return 5; }

int main() {
    someFcn(1, 3.4); // someFcn(int, double)
    someFcn(1, 3.4f); // someFcn(int, double), float is promoted to double
    someFcn(1.2, 3.4); // someFcn(double, double)
    someFcn(1.2f, 3.4); // someFcn(float, double)
    someFcn(1.2f, 3.4f); // someFcn(float, double), float is promoted to a double
}
```

Just like normal functions, function templates can have default arguments for non-template parameters. Each function instantiated from the template will use the same default argument.

---

**Instantiated function may give compile errors and thus not compile.**
**Instantiated functions may not always make sense semantically**
The compiler will successfully compile an instantiated function template as long as it makes sense syntactically.

```cpp
template <typename T>
T addOne(T x) { return x + 1; }

std::cout << addOne("Hello, world!") << '\n';
// "ello, world!"

std::string hello{"Hello, world!"};
std::cout << addOne(hello); //compiler error, std::string cannot be added to int
```

Perhaps surprisingly, because C++ syntactically allows addition of an integer value to a string literal (we cover this in future lesson [17.9 -- Pointer arithmetic and subscripting](https://www.learncpp.com/cpp-tutorial/pointer-arithmetic-and-subscripting/)), the above example compiles

---
### Beware function templates with modifiable static local variables

When a static local variable is used in a function template, each function instantiated from that template will have a separate version of the static local variable. This is rarely a problem if the static local variable is const. But consider,


```cpp
template <typename T>
void printIDAndValue(T value) {
    static int id{ 0 };
    std::cout << ++id << ") " << value << '\n';
}

int main() {
    printIDAndValue(12); // 1) 12
    printIDAndValue(13); // 2) 13
    printIDAndValue(14.5); // 1) 14.5
}
```

Note that `printIDAndValue<int>` and `printIDAndValue<double>` each have their own independent static local variable named `id`, not one that is shared between them. So `14.5` starts from 0 again.

---
#### Function templates with multiple template types

Consider the following program:

```cpp
template <typename T>
T max(T x, T y) {
    return (x < y) ? y : x;
}

int main() {
    std::cout << max(2, 3.5) << '\n';  // compile error
}
```

This program won’t compile. Instead, the compiler will issue (probably crazy looking) error messages. On Visual Studio:

```
Project3.cpp(11,18): error C2672: 'max': no matching overloaded function found
Project3.cpp(11,28): error C2782: 'T max(T,T)': template parameter 'T' is ambiguous
Project3.cpp(4): message : see declaration of 'max'
Project3.cpp(11,28): message : could be 'double'
Project3.cpp(11,28): message : or       'int'
Project3.cpp(11,28): error C2784: 'T max(T,T)': could not deduce template argument for 'T' from 'double'
Project3.cpp(4): message : see declaration of 'max'
```

We’re passing arguments of two different types: one `int` and one `double`. 
Because we’re making a function call without using angled brackets to specify an actual type, the compiler will first see if there is a non-template match for `max(int, double)` (not found). 
Next, the compiler will see if it can find a function template match (using template argument deduction). This will also fail, as: `T` can only represent a single type.  Because both parameters in the function template are of type `T`, they must resolve to the same actual type.

Why the compiler didn’t generate function `max<double>(double, double)` and then use numeric conversion to type convert the `int` argument to a `double`. 
**The answer is: type conversion is done only when resolving function overloads, not when performing template argument deduction.**

This lack of type conversion is intentional for at least two reasons:
1) It helps keep things simple: we either find an exact match between the function call arguments and template type parameters, or we don’t. 
2) Second, it allows us to create function templates for cases where we want to ensure that two or more parameters have the same type (as in the example above).

Now rather than using one template type parameter `T`, we’ll now use two (`T` and `U`):

```cpp
template <typename T, typename U>
T max(T x, U y) {
    return (x < y) ? y : x; 
    // uh oh, we have a narrowing conversion problem here
}

int main() {
    std::cout << max(2, 3.5) << '\n'; // resolves to max<int, double>
}
```

However, this example doesn’t work right. If you compile and run the program (with “treat warnings as errors” turned off), it will produce the following result: 3

The conditional operator (?:) requires its (non-condition) operands to be the same common type. The common type of `int` and `double` is `double`, so when the (non-condition) operands of our conditional operator are an `int` and a `double`, the value produced by the conditional operator will be of type `double`. In this case, that’s the value `3.5`, which is correct.

However, the declared return type of our function is `T`. When `T` is an `int` and `U` is a `double`, the return type of the function is `int`. Our value `3.5` is undergoing a narrowing conversion to `int` value `3`, resulting in a loss of data (and possibly a compiler warning).

In such cases, return type deduction (via `auto`) can be useful

```cpp
template <typename T, typename U>
auto max(T x, U y) {
    return (x < y) ? y : x;
}
```

A function with an `auto` return type needs to be fully defined before it can be used (a forward declaration won’t suffice), since the compiler has to inspect the function implementation to determine the return type.

If we need a function that can be forward declared, we have to be explicit about the return type. Since our return type needs to be the common type of `T` and `U`, we can use `std::common_type_t` to fetch the common type of `T` and `U` to use as our explicit return type:

```cpp
#include <type_traits> // for std::common_type_t
template <typename T, typename U>
auto max(T x, U y) -> std::common_type_t<T, U>; 
// returns the common type of T and U

int main() {
    std::cout << max(2, 3.5) << '\n';

    return 0;
}

template <typename T, typename U>
auto max(T x, U y) -> std::common_type_t<T, U> {
    return (x < y) ? y : x;
}
```

---
#### `auto` parameters C++-20

C++20 introduces a new use of the `auto` keyword: When the `auto` keyword is used as a parameter type in a normal function, the compiler will automatically convert the function into a function template with each auto parameter becoming an independent template type parameter. This method for creating a function template is called an **abbreviated function template**.

```cpp
auto max(auto x, auto y) {
    return (x < y) ? y : x;
}
```

There isn’t a concise way to use abbreviated function templates when you want more than one auto parameter to be the same type. That is, there isn’t an easy abbreviated function template for something like this:

---
### Function templates may be overloaded

Just like functions may be overloaded, function templates may also be overloaded. Such overloads can have a different number of template types and/or a different number or type of function parameters:

```cpp
// Add two values with matching types
template <typename T>
auto add(T x, T y) {
    return x + y;
}

// Add two values with non-matching types
template <typename T, typename U>
auto add(T x, U y) {
    return x + y;
}

// Add three values with any type
template <typename T, typename U, typename V>
auto add(T x, U y, V z) {
    return x + y + z;
}

int main() {
    std::cout << add(1.2, 3.4) << '\n'; 
    // instantiates and calls add<double>()
    std::cout << add(5.6, 7) << '\n';   
    // instantiates and calls add<double, int>()
    std::cout << add(8, 9, 10) << '\n'; 
    // instantiates and calls add<int, int, int>()
}
```

One interesting note here is that for the call to `add(1.2, 3.4)`, the compiler will prefer `add<T>(T, T)` over `add<T, U>(T, U)` even though both could possibly match.

The rules for determining which of multiple matching function templates should be preferred are called “partial ordering of function templates”. In short, whichever function template is more restrictive/specialized will be preferred. `add<T>(T, T)` is the more restrictive function template in this case (since it only has one template parameter), so it is preferred.

If multiple function templates can match a call and the compiler can’t determine which is more restrictive, the compiler will error with an ambiguous match.

---
### Non-type template parameters

A **non-type template parameter** is a template parameter with a fixed type that serves as a placeholder for a constexpr value passed in as a template argument.

A non-type template parameter can be any of the following types:
- An integral type
- An enumeration type
- `std::nullptr_t`
- A floating point type (since C++20)
- A pointer or reference to an object
- A pointer or reference to a function
- A pointer or reference to a member function
- A literal class type (since C++20)

In the case of `std::bitset`, the non-type template parameter is used to tell the `std::bitset` how many bits we want it to store.

```cpp
template <int N> // declare a non-type template parameter of type int named N
void print() {  std::cout << N << '\n';  }

int main() {
    print<5>(); 
}
```

Instantiation:

```cpp
template <>
void print<5>() { std::cout << 5 << '\n'; }
```

Function parameters cannot be constexpr. This is true for normal functions, constexpr functions (which makes sense, as they must be able to be run at runtime), and surprisingly, even consteval functions.

```cpp
double getSqrt(double d) {
    assert(d >= 0.0 && "getSqrt(): d must be non-negative");
    if (d >= 0) return std::sqrt(d);
    return 0.0;
}

int main() {
    std::cout << getSqrt(5.0) << '\n';
    std::cout << getSqrt(-5.0) << '\n';
}
```

When run, the call to `getSqrt(-5.0)` will runtime assert out. Because `-5.0` is a literal (and implicitly constexpr), it would be better if we could static_assert so that errors would be caught at compile-time. However, static_assert requires a constant expression, and function parameters can’t be constexpr

However, if we change the function parameter to a non-type template parameter instead, then we can do exactly as we want:

```cpp
template <double D> // requires C++20 for floating point non-type parameters
double getSqrt() {
    static_assert(D >= 0.0, "getSqrt(): D must be non-negative");
    if constexpr (D >= 0) return std::sqrt(D); 
    // strangely, std::sqrt isn't a constexpr function (until C++26)
    return 0.0;
}

int main() {
    std::cout << getSqrt<5.0>() << '\n';
    std::cout << getSqrt<-5.0>() << '\n';
    return 0;
}
```

`getSqrt<-5.0>()` will instantiate and call a function that looks like this:

```cpp
template <>
double getSqrt<-5.0>() {
    static_assert(-5.0 >= 0.0, "getSqrt(): D must be non-negative");
    if constexpr (-5.0 >= 0) return std::sqrt(-5.0);
    return 0.0;
}
```

The static_assert condition is false, so the compiler asserts out.

Non-type template parameters are used primarily when we need to pass constexpr values to functions (or class types) so they can be used in contexts that require a constant expression.

The class type `std::bitset` uses a non-type template parameter to define the number of bits to store because the number of bits must be a constexpr value.

---
#### Implicit conversions for non-type template arguments 

Certain non-type template arguments can be implicitly converted in order to match a non-type template parameter of a different type. For example:

```cpp
template <int N> // int non-type template parameter
void print() { std::cout << N << '\n'; }

int main() {
    print<5>();   // no conversion necessary
    print<'c'>(); // 'c' converted to type int, prints 9
}
```

In this context, only certain types of constexpr conversions are allowed. The most common types of allowed conversions include:
- Integral promotions (e.g. `char` to `int`)
- Integral conversions (e.g. `char` to `long` or `int` to `char`)
- User-defined conversions (e.g. some program-defined class to `int`)
- Lvalue to rvalue conversions (e.g. some variable `x` to the value of `x`)

Note that this list is less permissive than the type of implicit conversions allowed for list initialization. For example, you can list initialize a variable of type `double` using a `constexpr int`, but a `constexpr int` non-type template argument will not convert to a `double` non-type template parameter.

Unlike with normal functions, the algorithm for matching function template calls to function template definitions is not sophisticated (overload resolution is),  and certain matches are not prioritized over others based on the type of conversion required (or lack thereof). This means that if a function template is overloaded for different kinds of non-type template parameters, it can very easily result in an ambiguous match:

```cpp
template <int N> // int non-type template parameter
void print() { std::cout << N << '\n'; }
template <char N> // char non-type template parameter
void print() {  std::cout << N << '\n'; }

int main() {
    print<5>();   // ambiguous match with int N = 5 and char N = 5
    print<'c'>(); // ambiguous match with int N = 99 and char N = 'c'
}
```

Perhaps surprisingly, both of these calls to `print()` result in ambiguous matches.

---
### Type deduction for non-type template parameters using `auto` C++17

As of C++17, non-type template parameters may use `auto` to have the compiler deduce the non-type template parameter from the template argument:

```cpp
template <auto N> // deduce non-type template parameter from template argument
void print() { std::cout << N << '\n'; }
int main() {
    print<5>();   // N deduced as int `5`
    print<'c'>(); // N deduced as char `c`
}
```

Why this example doesn’t produce an ambiguous match like the example above ? 
The compiler looks for ambiguous matches first, and then instantiates the function template if no ambiguous matches exist. In this case, there is only one function template, so there is no possible ambiguity.

After instantiating the function template for the above example, the program looks like this:

```cpp
template <auto N>
void print() { std::cout << N << '\n'; }

template <>
void print<5>() { std::cout << 5 << '\n'; }
 // note that this is print<5> and not print<int>

template <>
void print<'c'>() {  std::cout << 'c' << '\n'; }
 // note that this is print<`c`> and not print<char>
```

----
###  Using function templates in multiple files

As compiler needs to place the definition of the functions derived from the template, it needs the definition of the template to be known at compile time. 
If you just place a forward declaration of the template, it will assume that the derived functions definition exist elsewhere but if the same derived functions are not used elsewhere the linker won't find any definitions and will cause a linker error. 
If the same derived functions are used elsewhere where the definition of the template was placed, it will work(as the compiler would have placed the derived function's definition there) but these solutions are fragile and should be avoided.

The most conventional way to address this issue is to put all your template code in a header (.h) file instead of a source (.cpp) file. That way, any files that need access to the template can `#include` the relevant header, and the template definition will be copied by the preprocessor into the source file. The compiler will then be able to instantiate any functions that are needed. 

The ODR says that types, templates, inline functions, and inline variables are allowed to have identical definitions in different files. So there is no problem if the template definition is copied into multiple files (as long as each definition is identical).

But what about the instantiated functions themselves? **Functions implicitly instantiated from templates are implicitly inline**. And as you know, inline functions can be defined in multiple files, so long as the definition is identical in each. The templates themselves are not inline, as the concept of inline only applies to variables and functions.

---