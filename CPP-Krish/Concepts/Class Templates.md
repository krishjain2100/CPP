
Unlike functions, type definitions can’t be overloaded. The compiler will give a redefinition error if you declare a class twice with different members

A class type is a struct, class, or union type. Although we’ll be demonstrating class templates on struct, everything here applies equally well to classes.

```cpp
template <typename T>
struct Pair {
    T first{};
    T second{};
};

int main() {
    Pair<int> p1{ 5, 6 }; 
     // instantiates Pair<int> and creates object p1
    cout << p1.first << ' ' << p1.second << '\n';
    Pair<double> p2{ 1.2, 3.4 }; 
    // instantiates Pair<double> and creates obj p2
    cout << p2.first << ' ' << p2.second << '\n';
    Pair<double> p3{ 7.8, 9.0 }; 
    // creates object p3 using prior defenition for Pair<double>
    cout << p3.first << ' ' << p3.second << '\n';
}
```

What the compiler actually compiles after all template instantiations are done:

```cpp

// A declaration for our Pair class template
// (we don't need the definition any more since it's not used)
template <typename T>
struct Pair;

// Explicitly define what Pair<int> looks like
template <> // tells the compiler this is a template type with no template parameters
struct Pair<int> {
    int first{};
    int second{};
};

// Explicitly define what Pair<double> looks like
template <> // tells the compiler this is a template type with no template parameters
struct Pair<double> {
    double first{};
    double second{};
};

int main() {
    Pair<int> p1{ 5, 6 }; 
    cout << p1.first << ' ' << p1.second << '\n';
    Pair<double> p2{ 1.2, 3.4 }; 
    cout << p2.first << ' ' << p2.second << '\n';
    Pair<double> p3{ 7.8, 9.0 }; 
    cout << p3.first << ' ' << p3.second << '\n';
}
```

Example: `max()` being implemented as a function template:

```cpp
template <typename T>
struct Pair {
    T first{};
    T second{};
};

template <typename T>
constexpr T max(Pair<T> p) {
    return (p.first < p.second ? p.second : p.first);
}

int main() {
    Pair<int> p1{ 5, 6 };
    std::cout << max<int>(p1) << " is larger\n"; 
    // explicit call to max<int>
    Pair<double> p2{ 1.2, 3.4 };
    std::cout << max(p2) << " is larger\n"; 
    // call to max<double> using template argument deduction (prefer this)
}
```

When the `max()` function is called with a `Pair<int>` argument, the compiler will instantiate the function:
```cpp
template <>
constexpr int max(Pair<int> p) {
    return (p.first < p.second ? p.second : p.first);
}
```

As with all calls to a function template, we can either be explicit about the template type argument (e.g. `max<int>(p1)`) or we can be implicit (e.g. `max(p2)`) and let the compiler use template argument deduction to determine what the template type argument should be.

Class templates can have some members using a template type and other members using a normal (non-template) type.  Also, they can have multiple template types.

In some cases, we may write function templates that we want to use with any type that will successfully compile. To do that, we simply use a type template parameter as the function parameter instead.

```cpp
template <typename T, typename U>
struct Pair {
    T first{};
    U second{};
};

struct Point{
    int first{};
    int second{};
};

template <typename T>
void print(T p) { // type template parameter will match anything
    std::cout << '[' << p.first << ", " << p.second << ']'; 
    // will only compile if type has first and second members
}
```

There is one case that can be misleading. Consider the following version of `print()`:

```cpp
template <typename T, typename U>
struct Pair  { // defines a class type named Pair
    T first{};
    U second{};
};

template <typename Pair> // defines a type template parameter named Pair (shadows Pair class type)
void print(Pair p) {
	 // this refers to template parameter Pair, not class type Pair
    std::cout << '[' << p.first << ", " << p.second << ']';
}
```

The issue here is that when we define `Pair` as a type template parameter, it shadows other uses of the name `Pair` within the global scope. So within the function template, `Pair` refers to the template parameter `Pair`, not the class type `Pair`.  This is a good reason to stick to simple template parameter names, such a `T`, `U`, `N`, as they are less likely to shadow a class type name.


Just like function templates, class templates are typically defined in header files so they can be included into any code file that needs them. Both template definitions and type definitions are exempt from the one-definition rule.

---
### Class template argument deduction (CTAD)

Starting in C++17, when instantiating an object from a class template, the compiler can deduce the template types from the types of the object’s initializer (this is called **class template argument deduction** or **CTAD** for short). For example:

```cpp
#include <utility> // for std::pair

int main() {
    std::pair<int, int> p1{ 1, 2 }; 
    // explicitly specify class template std::pair<int, int> (C++11 onward)
    std::pair p2{ 1, 2 };           
    // CTAD used to deduce std::pair<int, int> from the initializers (C++17)
}
```

CTAD is only performed if no template argument list is present. Therefore, both of the following are errors:

```cpp
std::pair<> p1 { 1, 2 }; 
// error: too few template arguments, both arguments not deduced
std::pair<int> p2 { 3, 4 }; 
// error: too few template arguments, second argument not deduced
```

In most cases, CTAD works right out of the box. However, in certain cases, the compiler may need a little extra help understanding how to deduce the template arguments properly.
You may be surprised to find that the following program doesn’t compile in C++17 (only):

```cpp
// define our own Pair type
template <typename T, typename U>
struct Pair {
    T first{};
    U second{};
};

Pair<int, int> p1{ 1, 2 }; // ok
Pair p2{ 1, 2 };  // compile error in C++17 (okay in C++20)
```

If you compile this in C++17, you’ll likely get some error about “class template argument deduction failed” or “cannot deduce template arguments” or “No viable constructor or deduction guide”. This is because in C++17, CTAD doesn’t know how to deduce the template arguments for aggregate class templates. To address this, we can provide the compiler with a **deduction guide**, which tells the compiler how to deduce the template arguments for a given class template. Here’s the same program with a deduction guide:

```cpp
template <typename T, typename U>
struct Pair {
    T first{};
    U second{};
};

// Here's a deduction guide for our Pair (needed in C++17 only)
// Pair objects initialized with arguments of type T and U should deduce to Pair<T, U>
template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;

int main() {
    Pair<int, int> p1{ 1, 2 }; 
    Pair p2{ 1, 2 };   
    // CTAD used to deduce Pair<int, int> from the initializers (C++17)
}
```

The deduction guide for our `Pair` class is pretty simple, but let’s take a closer look at how it works.

```cpp
// Here's a deduction guide for our Pair (needed in C++17 only)
// Pair objects initialized with arguments of type T and U should deduce to Pair<T, U>
template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;
```

First, we use the same template type definition as in our `Pair` class. This makes sense, because if our deduction guide is going to tell the compiler how to deduce the types for a `Pair<T, U>`, we have to define what `T` and `U` are (template types). Second, on the right hand side of the arrow, we have the type that we’re helping the compiler to deduce. In this case, we want the compiler to be able to deduce template arguments for objects of type `Pair<T, U>`, so that’s exactly what we put here. Finally, on the left side of the arrow, we tell the compiler what kind of declaration to look for. In this case, we’re telling it to look for a declaration of some object named `Pair` with two arguments (one of type `T`, the other of type `U`). We could also write this as `Pair(T t, U u)` (where `t` and `u` are the names of the parameters, but since we don’t use `t` and `u`, we don’t need to give them names).

Putting it all together, we’re telling the compiler that if it sees a declaration of a `Pair` with two arguments (of types `T` and `U` respectively), it should deduce the type to be a `Pair<T, U>`.

So when the compiler sees the definition `Pair p2{ 1, 2 };` in our program, it will say, “oh, this is a declaration of a `Pair` and there are two arguments of type `int` and `int`, so using the deduction guide, I should deduce this to be a `Pair<int, int>`“.

Here’s a similar example for a Pair that takes a single template type:

```cpp
template <typename T>
struct Pair {
    T first{};
    T second{};
};

// Here's a deduction guide for our Pair (needed in C++17 only)
template <typename T>
Pair(T, T) -> Pair<T>;

int main() {
    Pair<int> p1{ 1, 2 }; 
    Pair p2{ 1, 2 };  
    // CTAD used to deduce Pair<int> from the initializers (C++17)
}
```


C++20 added the ability for the compiler to automatically generate deduction guides for aggregates, so deduction guides should only need to be provided for C++17 compatibility.

`std::pair` (and other standard library template types) come with pre-defined deduction guides, which is why our example above that uses `std::pair` compiles fine in C++17 without us having to provide deduction guides ourselves.

Non-aggregates don’t need deduction guides in C++17 because the presence of a constructor serves the same purpose.


Just like function parameters can have default arguments, template parameters can be given default values. These will be used when the template parameter isn’t explicitly specified and can’t be deduced.

Here’s a modification of our `Pair<T, U>` class template program above, with type template parameters `T` and `U` defaulted to type `int`:

```cpp
template <typename T=int, typename U=int> // default T and U to type int
struct Pair {
    T first{};
    U second{};
};

template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;

int main(){
    Pair<int, int> p1{ 1, 2 };
    // explicitly specify class template Pair<int, int> (C++11 onward)
    Pair p2{ 1, 2 }; 
    // CTAD used to deduce Pair<int, int> from the initializers (C++17)
    Pair p3;   // uses default Pair<int, int>
}
```

Our definition for `p3` does not explicitly specify types for the type template parameters, nor is there an initializer for these types to be deduced from. Therefore, the compiler will use the types specified in the defaults, which means `p3` will be of type `Pair<int, int>`.


When initializing the member of a class type using non-static member initialization, CTAD will not work in this context. All template arguments must be explicitly specified:

```cpp
#include <utility> // for std::pair

struct Foo {
    std::pair<int, int> p1{ 1, 2 }; 
    std::pair p2{ 1, 2 };  // compile error, CTAD can't be used in this context
};

int main() {
    std::pair p3{ 1, 2 };           // ok, CTAD can be used here
}
```

CTAD stands for class template _argument_ deduction, not class template _parameter_ deduction, so it will only deduce the type of template arguments, not template parameters. Therefore, CTAD can’t be used in function parameters.

```cpp
void print(std::pair p)  {// compile error, CTAD can't be used here
    std::cout << p.first << ' ' << p.second << '\n';
}

int main() {
    std::pair p { 1, 2 }; // p deduced to std::pair<int, int>
    print(p);
}
```

In such cases, you should use a template instead:

```cpp
template <typename T, typename U>
void print(std::pair<T, U> p) {
    std::cout << p.first << ' ' << p.second << '\n';
}

int main() {
    std::pair p { 1, 2 }; // p deduced to std::pair<int, int>
    print(p);
}
```

---
### Alias templates

Creating a type alias for a class template where all template arguments are explicitly specified works just like a normal type alias:

```cpp
template <typename T>
struct Pair {
    T first{};
    T second{};
};

template <typename T>
void print(const Pair<T>& p) {
    std::cout << p.first << ' ' << p.second << '\n';
}

int main() {
    using Point = Pair<int>; // create normal type alias
    Point p { 1, 2 }; // compiler replaces this with Pair<int>
    print(p);
}
```

Such aliases can be defined locally (e.g. inside a function) or globally.

An **alias template** can be used to instantiate type aliases. Just like type aliases do not define distinct types, alias templates do not define distinct types. 

```cpp
template <typename T>
struct Pair {
    T first{};
    T second{};
};

template <typename T>
using Coord = Pair<T>; // Coord is an alias for Pair<T>

// Our print function template needs to know that Coord's template parameter T is a type template parameter
template <typename T>
void print(const Coord<T>& c) {
    std::cout << c.first << ' ' << c.second << '\n';
}

int main() {
    Coord<int> p1 { 1, 2 }; // Pre C++-20: We must explicitly specify all type template argument
    Coord p2 { 1, 2 }; // In C++20, we can use alias template deduction to deduce the template arguments in cases where CTAD works
    std::cout << p1.first << ' ' << p1.second << '\n';
    print(p2);
}
```

Things worth noting about this example:
- Unlike normal type aliases (which can be defined inside a block), alias templates must be defined in the global scope (as all templates must).
- rior to C++20, we must explicitly specify the template arguments when we instantiate an object using an alias template. As of C++20, we can use **alias template deduction**, which will deduce the type of the template arguments from an initializer in cases where the aliased type would work with CTAD.
- Because CTAD doesn’t work on function parameters, when we use an alias template as a function parameter, we must explicitly define the template arguments used by the alias template. In other words, we do this:

```cpp
template <typename T>
void print(const Coord<T>& c) {
    std::cout << c.first << ' ' << c.second << '\n';
}
```

Not this:

```cpp
void print(const Coord& c)  {
	// won't work, missing template arguments
    std::cout << c.first << ' ' << c.second << '\n';
}
```

This is no different than if we’d used `Pair` or `Pair<T>` instead of `Coord` or `Coord<T>`.

---
