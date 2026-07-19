Unlike functions, type definitions can’t be overloaded. The compiler will give a redefinition error if you declare a type (e.g., a class type) twice with different members.
A class type is a struct, class, or union.

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

Example: Non-member function `max()` being implemented as a function template:

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


 Consider the following version of `print()`:
 
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

When we define `Pair` as a type template parameter, it shadows other uses of the name `Pair` within the global scope. So within the function template, `Pair` refers to the template parameter `Pair`, not the class type `Pair`.  So use simple template parameter names, like `T`, `U`, `N`, as they are less likely to shadow a class type name.

Just like function templates, class templates are typically defined in header files so they can be included into any code file that needs them. Both template definitions and type definitions are exempt from the one-definition rule.

---
### Class template argument deduction (CTAD)

Starting in C++17, when instantiating an object from a class template, the compiler can deduce the template types from the types of the object’s initializer (this is called **class template argument deduction** or **CTAD** for short). For example:

```cpp
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
You may be surprised to find that the following program doesn’t compile in C++17:

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

Let’s take a closer look at how it works.

```cpp
template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;
```

- We use the same template type definition as in our `Pair` class. 
- On the right hand side of the arrow, we have the type that we’re helping the compiler to deduce. 
- On the left side of the arrow, we tell the compiler what kind of declaration to look for. In this case, we’re telling it to look for a declaration of some object named `Pair` with two arguments (one of type `T`, the other of type `U`). We could also write this as `Pair(T t, U u)` (where `t` and `u` are the names of the parameters, but since we don’t use `t` and `u`, we don’t need to give them names).

Here’s a similar example for a Pair that takes a single template type:

```cpp
template <typename T>
struct Pair {
    T first{};
    T second{};
};

// Deduction Guide
template <typename T>
Pair(T, T) -> Pair<T>;

int main() {
    Pair<int> p1{ 1, 2 }; 
    Pair p2{ 1, 2 };  
}
```

C++20 added the ability for the compiler to automatically generate deduction guides for aggregates, so deduction guides should only need to be provided for C++17 compatibility.

`std::pair` (and other standard library template types) come with pre-defined deduction guides, which is why our example above that uses `std::pair` compiles fine in C++17 without us having to provide deduction guides ourselves.

Non-aggregates don’t need deduction guides in C++17 because the presence of a constructor serves the same purpose.

Template parameters can be given default values. These will be used when the template parameter isn’t explicitly specified and can’t be deduced. Example:

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
    Pair p3;   
    // uses default Pair<int, int>
}
```

Our definition for `p3` does not explicitly specify types for the type template parameters, nor is there an initializer for these types to be deduced from. Therefore, the compiler will use the types specified in the defaults, which means `p3` will be of type `Pair<int, int>`.

When initializing the member of a class type using non-static member initialization, CTAD will not work in this context. All template arguments must be explicitly specified:

```cpp
struct Foo {
    std::pair<int, int> p1{ 1, 2 }; 
    std::pair p2{ 1, 2 };  // compile error, CTAD can't be used in this context
};

int main() {
    std::pair p3{ 1, 2 }; // ok, CTAD can be used here
}
```

CTAD stands for class template _argument_ deduction, not class template _parameter_ deduction, so it will only deduce the type of template arguments, not template parameters. Therefore, CTAD can’t be used in function parameters.

```cpp
void print(std::pair p)  { // compile error, CTAD can't be used here
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
- Prior to C++20, we must explicitly specify the template arguments when we instantiate an object using an alias template. As of C++20, we can use **alias template deduction**, which will deduce the type of the template arguments from an initializer in cases where the aliased type would work with CTAD.
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
### Class templates with member functions

Type template parameters defined as part of a class template parameter declaration can be used both as the type of data members and as the type of member function parameters.
Note that the entire class template is not instantiated at the moment of definition for a particular type. The member functions are instantiated by the compiler as and when they are encountered.

Example:

```cpp
template <typename T>
class Pair {
private:
    T m_f{};
    T m_s{};
public:
    Pair(const T& f, const T& s) : m_f{ f } , m_s{ s } {}
    bool isEqual(const Pair<T>& pair);
	Pair(const Pair&) = delete;
    Pair& operator=(const Pair&) = delete;
};

// When we define a member function outside the class definition,
// we need to resupply a template parameter declaration
template <typename T>
bool Pair<T>::isEqual(const Pair<T>& pair) {
    return m_f == pair.m_f && m_s == pair.m_s;
}

int main() {
    Pair p1{5,6}; // uses CTAD to infer type Pair<int>
    cout << p1.isEqual(Pair{5,6}) << '\n';
    cout << p1.isEqual(Pair{5,7}) << '\n';
}
```

Note that when we define a member function inside the class template definition, we don’t need to provide a template parameter declaration for the member function. Such member functions implicitly use the class template parameter declaration like the copy constructor and the assignment operator above.

Since `isEqual` function definition is separate from the class template definition, we need to resupply a template parameter declaration (`template <typename T>`) so the compiler knows what `T` is. Also, when we define a member function outside of the class, we need to qualify the member function name with the fully templated name of the class template (`Pair<T>::isEqual`, not `Pair::isEqual`).

---
#### Injected class names

In a prior lesson, we noted that the name of a constructor must match the name of the class. But in our class template for `Pair<T>` above, we named our constructor `Pair`, not `Pair<T>`. Somehow this still works, even though the names don’t match.

Within the scope of a class, the unqualified name of the class is called an **injected class name**. In a class template, the injected class name serves as shorthand for the fully templated name.

Because `Pair` is the injected class name of `Pair<T>`, within the scope of our `Pair<T>` class template, any use of `Pair` will be treated as if we had written `Pair<T>` instead. Therefore, although we named the constructor `Pair`, the compiler treats it as if we had written `Pair<T>` instead. The names now match.

This means that we can also define our `isEqual()` member function like this:

```cpp
template <typename T>
// note the parameter has type Pair, not Pair<T>
bool Pair<T>::isEqual(const Pair& pair)  { 
    return m_f == pair.m_f && m_s == pair.m_s;
}
```

 We noted that CTAD doesn’t work with function parameters (as it is argument deduction, not parameter deduction). However, using an injected class name as a function parameter is okay, as it is shorthand for the fully templated name, not a use of CTAD.

---
#### Where to define member functions for class templates outside the class

With non-template classes, we put the class definition in a header file, and the member function definitions in a similarly named code file. However, with templates, this does not work. 

Remember that C++ compiles files individually. When `main.cpp` is compiled, the contents of the  Class.h header are copied into main.cpp. When the compiler sees that we need two template instances, `Class<int>`, and `Class<double>`,  it will instantiate these, and compile them as part of the main.cpp translation unit. Because the member function has a declaration inside the template class definition, the compiler will accept a call to it, assuming it will be defined elsewhere. When `Class.cpp` is compiled separately, the contents of the `Class.h` header are copied into it, but the compiler won’t find any code in `Class.cpp` that requires the  class template or  function template to be instantiated, so it won’t instantiate anything. Thus, when the program is linked, we’ll get a linker error, because `main.cpp` made a call to member function but that template function was never instantiated.

There are quite a few ways to work around this:

1. Define the function template _inside_ the class definition. This makes things easy at the cost of cluttering our class definition.

2. Put all of your template class code (including member functions defined outside the class) in the header file.  Functions implicitly instantiated from templates are implicitly inline. This includes both non-member and member function templates. But if the template class is used in many files, it can increase your compile and link time. 

3. An alternative is to move the contents of `Class.cpp` to a new file named `Class.inl` (stands for inline), and then include `Class.inl` at the bottom of the `Class.h` header (inside the header guard). That yields the same result as putting all the code in the header, but helps keep things a little more organised. If you get a compiler error about duplicate definitions, your compiler is most likely compiling the `.inl` file as part of the project as if it were a code file. This results in the contents of the `.inl` getting compiled twice: once when your compiler compiles the `.inl`, and once when the `.cpp` file that includes the `.inl` gets compiled. If the `.inl` file contains any non-inline functions (or variables), then we violate the one definition rule. If this happens, you’ll need to exclude the `.inl` file from being compiled as part of the build.

4. Other solutions involve including `.cpp` files, but these are not recommended because of the non-standard usage of `#include`.

5. Another alternative is to use a three-file approach. The template class definition goes in the header. The template class member functions goes in the code file. Then you add a third file, which contains _all_ of the instantiated classes you need:

templates.cpp:

```cpp
#include "Class.h"
#include "Class.cpp" // we're breaking best practices, but only once

// #include other .h and .cpp template definitions you need here
template class Class<int>; // Explicitly instantiate template Array<int>
template class Class<double>; // Explicitly instantiate template Array<double>
// instantiate other templates here
```

The `template class` command causes the compiler to explicitly instantiate the template class and all of it's member functions too . Other code files that want to use these types can include Class.h (to satisfy the compiler), and the linker will link in these explicit type definitions from `template.cpp`. This method may be more efficient (depending on how your compiler and linker handle templates and duplicate definitions), but requires maintaining the `templates.cpp` file for each program.

---
### Class Template Specialization

Class template specializations are treated as completely independent classes, even though they are instantiated in the same way as the templated class. This means that we can change anything and everything about our specialization class, including the way it’s implemented and even the functions it makes public, just as if it were an independent class.

Just like all templates, the compiler must be able to see the full definition of a specialization to use it. Also, defining a class template specialization requires the non-specialized class to be defined first.

```cpp
template <typename T>
class Storage8 {
private:  T m_array[8];
public:
    void set(int index, const T& value) { m_array[index] = value; }
    const T& get(int index) const { return m_array[index];}
};

// A specializtion to save space
template <> // the following is a template class with no templated parameters
class Storage8<bool> {
private: std::uint8_t m_data{};
public:
    void set(int index, bool value) {
        auto mask{ 1 << index };
        if (value) m_data |= mask;   
        else m_data &= ~mask; 
	}
    bool get(int index) {
        auto mask{ 1 << index };
        return (m_data & mask);
    }
};
```

Now, when we instantiate an object type `Storage<T>`, where `T` is not a `bool`, we’ll get a version stenciled from the generic templated `Storage8<T>` class. When we instantiate an object of type `Storage8<bool>`, we’ll get the specialized version we just created.

---
### Specializing member functions

Consider the following class template:

```cpp
template <typename T>
class Storage {
private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; } 
};

int main() {
    Storage i { 5 };
    Storage d { 6.7 };
    i.print();
    d.print();
}
```

Note that `i.print()` calls `Storage<int>::print()` and `d.print()` calls `Storage<double>::print()`. Therefore, if we want to change the behavior of this function when `T` is a double, we need to specialize `Storage<double>::print()`, which is a class template specialization, not a function template specialization.

```cpp
template <typename T>
class Storage {
private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; } 
};

template <>
class Storage<double> {
private: double m_value {};
public: 
	Storage(double value) : m_value { value } {}
    void print();
};

// Now Storage<double> is a regular class now 
// This is a normal member function definition
void Storage<double>::print() { cout << std::scientific << m_value << '\n';}

int main() {
    Storage i { 5 };
    Storage d { 6.7 }; // uses explicit specialization Storage<double>
    i.print(); // calls Storage<int>::print (instantiated from Storage<T>)
    d.print(); // calls Storage<double>::print
}
```

However, note the redundancy. We’ve duplicated an entire class definition just so that we can change one member function. Fortunately, we can let the compiler implicitly specialize `Storage<double>` from `Storage<T>`, and provide an explicit specialization of just `Storage<double>::print()`:

```cpp
template <typename T>
class Storage {
private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; } 
};

// This is a specialized member function definition and requires template<>
template<>
void Storage<double>::print() { cout << std::scientific << m_value << '\n'; }

int main() {
    Storage i { 5 };
    Storage d { 6.7 }; //  Storage<double> is implicitly instantiated
    i.print(); // calls Storage<int>::print (instantiated from Storage<T>)
    d.print(); // calls Storage<double>::print (called from explicit specialization of Storage<double>::print())
}
```

As noted earlier, function specializations are not implicitly inline, so we should mark our specialization of `Storage<double>::print()` as inline if it is defined it in a header file.

In the first approach too, you will have to make it inline since it is defined outside the class. No issues, if it is defined outisde as such member functions are implicitly inline

#### Where to define class template specializations

In order to use a specialization, the compiler must be able to see the full definition of both the non-specialized class and the specialized class. If the compiler can only see the definition of the non-specialized class, it will use that instead of the specialization. So specialized classes and functions are often defined in a header file just below the definition of the non-specialized class, 

If a specialization is only required in a single translation unit, it can be defined in the source file for that translation unit. Because other translation units will not be able to see the definition of the specialization, they will continue to use the non-specialized version.

You might be tempted to put a specialization in its own separate header file, with the intent of including the specialization’s header in any translation unit where the specialization is desired. It’s a bad idea  changes behavior based on the presence or absence of a header file. For example, if you intend to use the specialization but forget to include the header of the specialization, you may end up using the non-specialized version instead. If you intend to use the non-specialization, you may end up using the specialization anyway if some other header includes the specialization as a transitive include.

---
### Partial template specialization
**(DID NOT UNDERSTAND ONE BIT)**

Static Array class:

```cpp
template <typename T, int size>
class StaticArray {
private:
    T m_array[size]{};
public:
    T* getArray() { return m_array; }
    const T& operator[](int index) const { return m_array[index]; }
    T& operator[](int index) { return m_array[index]; }
};
```

We want to write a function to print out the whole array. We’re going to do it as a non-member function because it will make the successive examples easier to follow.

Using templates, we might write something like this:

```cpp
template <typename T, int size>
void print(const StaticArray<T, size>& array) {
    for (int count{ 0 }; count < size; ++count)
		cout << array[count] << ' ';
}
```

But with char type, we don't want spaces in between the letters.
We can use template specialization to handle this. But the problem with full template specialization is that all template parameters must be explicitly defined. So we could only specialise it for a single `size` at a time.

Partial template specialization allows us to specialize classes (but not individual functions) where some, but not all, of the template parameters have been explicitly defined.

```cpp
// overload of print() function for partially specialized StaticArray<char, size>
template <int size> // size is still a template non-type parameter
void print(const StaticArray<char, size>& array)  {
	for (int count{ 0 }; count < size; ++count)
		std::cout << array[count];
}
```

Partial template specialization can only be used with classes, not template functions (functions must be fully specialized). `void print(StaticArray<char, size> &array)` works because the print function is not partially specialized, it’s just an overloaded template function that happens to have a partially-specialized class parameter.

```cpp
// Error: You cannot partially specialize a function. 
template <int size> 
void print<char, size>(const StaticArray<char, size>& array) {}
// Notice the angled brackets after print
```

---
### Partial template specialization for member functions

The limitation on the partial specialization of functions can lead to some challenges when dealing with member functions. For example:

```cpp
template <typename T, int size>
class StaticArray {
private: T m_array[size]{};
public:
    T* getArray() { return m_array; }
    const T& operator[](int index) const { return m_array[index]; }
    T& operator[](int index) { return m_array[index]; }
    void print() const;
};

template <typename T, int size>
void StaticArray<T, size>::print() const {
    for (int i{ 0 }; i < size; ++i)
        std::cout << m_array[i] << ' ';
    std::cout << '\n';
}
```

You might try this to partially specialize `print()`:

```cpp
template <int size>
void StaticArray<double, size>::print() const {
	for (int i{ 0 }; i < size; ++i)
		std::cout << std::scientific << m_array[i] << ' ';
	std::cout << '\n';
}
```

Unfortunately, this doesn’t work, because we’re trying to partially specialize a function, which is disallowed. One obvious way to handle this  is to partially specialize the entire class:


```cpp
template <typename T, int size>
class StaticArray {
private: T m_array[size]{};
public:
    T* getArray() { return m_array; }
    const T& operator[](int index) const { return m_array[index]; }
    T& operator[](int index) { return m_array[index]; }
    void print() const;
};

template <typename T, int size>
void StaticArray<T, size>::print() const {
    for (int i{ 0 }; i < size; ++i)
        std::cout << m_array[i] << ' ';
    std::cout << '\n';
}

// Partially specialized class
template <int size>
class StaticArray<double, size> {
private: double m_array[size]{};
public:
	double* getArray() { return m_array; }
	const double& operator[](int index) const { return m_array[index]; }
	double& operator[](int index) { return m_array[index]; }
	void print() const;
};

// Member function of partially specialized class
template <int size>
void StaticArray<double, size>::print() const {
	for (int i{ 0 }; i < size; ++i)
		std::cout << std::scientific << m_array[i] << ' ';
	std::cout << '\n';
}
```

This works because `StaticArray<double, size>::print()` is no longer a partially specialized function, it is a non-specialized member of partially specialized class `StaticArray<double, size>`. However, this isn’t a great solution, because there is a lot of redudancy.

If only there were some way to reuse the code in `StaticArray<T, size>` in `StaticArray<double, size>`. Ahha.. Inheritance. First Attempt:

```cpp
template <int size> // size is the expression parameter
class StaticArray<double, size>: public StaticArray<T, size>
```

But this doesn’t work, because we’ve used `T` without defining it. There is no syntax that allows us to inherit in such a manner. Also, even if we were able to define `T` as a type template parameter, when `StaticArray<double, size>` was instantiated, the compiler would need to replace the `T` in `StaticArray<T, size>` with an actual type. What actual type would it use? The only type that makes sense is `T=double`, but that would leave `StaticArray<double, size>` inheriting from itself. Fortunately, there’s a workaround, by using a common base class:

```cpp
template <typename T, int size>
class StaticArray_Base {
protected: T m_array[size]{};
public:
	T* getArray() { return m_array; }
	const T& operator[](int index) const { return m_array[index]; }
	T& operator[](int index) { return m_array[index]; }
	void print() const {
		for (int i{ 0 }; i < size; ++i)
			std::cout << m_array[i] << ' ';
		std::cout << '\n';
	}
// Don't forget a virtual destructor if you're going to use virtual function resolution
};

template <typename T, int size>
class StaticArray: public StaticArray_Base<T, size> {};

template <int size>
class StaticArray<double, size>: public StaticArray_Base<double, size> {
public:
	void print() const {
		for (int i{ 0 }; i < size; ++i)
			std::cout << std::scientific << this->m_array[i] << ' ';
// note: The this-> prefix in the above line is needed.
// See https://stackoverflow.com/a/6592617 or https://isocpp.org/wiki/faq/templates#nondependent-name-lookup-members for more info on why.
		std::cout << '\n';
	}
};

int main() {
	StaticArray<int, 6> intArray{};
	for (int count{ 0 }; count < 6; ++count)
		intArray[count] = count;
	intArray.print();
	StaticArray<double, 4> doubleArray{};
	for (int count{ 0 }; count < 4; ++count)
		doubleArray[count] = (4.0 + 0.1 * count);
	doubleArray.print();
}
```

---
### Partial template specialization for pointers

Recap:

```cpp
template <typename T>
class Storage {
private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; } 
};

// This is a specialized member function definition and requires template<>
template<>
void Storage<double>::print() { cout << std::scientific << m_value << '\n'; }

int main() {
    double d { 1.2 };
    double *ptr { &d };
    Storage s { ptr };
    s.print(); // 0x7ffe164e0f50
}

```

This compiles but malfunctions when `T` is a pointer type. `m_value` has type `double*`. It is the address that is printed when the `print()` member function is called. So how do we fix this?

You might think to to try creating a template function overloaded on type `T*`:

```cpp
// doesn't work
template<typename T>
void Storage<T*>::print() {
    if (m_value)
	    std::cout << std::scientific << *m_value << '\n';
}
```

Such a function is a partially specialized template function because it’s restricting what type `T` can be (to a pointer type), but `T` is still a type template parameter. So, this doesn’t as functions cannot be partially specialized. As noted above, only classes can be partially specialized.
So let’s partially specialize the `Storage` class instead:

```cpp

template <typename T>
class Storage {
private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; }
};

template <typename T> // we still have a type template parameter
class Storage<T*> {// This is partially specialized for T*
private: T* m_value {};
public:
    Storage(T* value) : m_value { value } {}
    void print();
};

template <typename T>
// This is a non-specialized function of partially specialized class Storage<T*>
void Storage<T*>::print() {
    if (m_value) std::cout << std::scientific << *m_value << '\n';
}

int main() {
    double d { 1.2 };
    double *ptr { &d };
    Storage s { ptr }; // instantiates Storage<double*> 
    s.print(); // calls Storage<double*>::print()
}
```

We’ve defined `Storage<T*>::print()` outside the class just to show how it’s done, and to show that the definition is identical to the partially specialized function `Storage<T*>::print()` that did not work above. 

It’s  worth a reminder that the partial specialization `Storage<T*>` needs to be defined after the primary template class `Storage<T>`.

#### Ownership and lifetime issues

Because `m_value` is a `T*`, it is a pointer to the object that is passed in. If that object is then destroyed, our `Storage<T*>` will be left dangling. The core problem is that our implementation of `Storage<T>` has copy semantics (meaning it makes a copy of its initializer) but `Storage<T*>` has reference semantics (meaning it’s a reference to its initializer). This inconsistency can lead to bugs.

There are a few different ways we can deal with such issues (in order of increasing complexity):

1. Make it clear that `Storage<T*>` is a view class (with reference semantics), so it’s on the caller to ensure the object being pointed to stays valid for as long as the `Storage<T*>` does. 

2. Prevent use of `Storage<T*>` altogether. We probably don’t need `Storage<T*>` to exist, as the caller can always dereference the pointer at the point of instantiation to use `Storage<T>` and make a copy of the value. However, while you can delete an overloaded function, C++ (as of C++23) won’t let you delete a class. The obvious solution is to partially specialize `Storage<T*>` and then do something to make it not compile (e.g. `static_assert`) when the template is instantiated, this approach has one major downside: `std::nullptr_t` is not a pointer type, so `Storage<std::nullptr_t>` won’t match `Storage<T*>`.

3. A better solution is to avoid partial specialization altogether, and use a `static_assert` on our primary template to ensure `T` is a type that we’re okay with.  Example:

```cpp
#include <iostream>
#include <type_traits> // for std::is_pointer_v and std::is_null_pointer_v

template <typename T>
class Storage {
    static_assert(!std::is_pointer_v<T> && !std::is_null_pointer_v<T>, "Storage<T*> and Storage<nullptr> disallowed");

private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; }
};

int main() {
    double d { 1.2 };
    Storage s1 { d }; // ok
    s1.print();
    Storage s2 { &d }; // static_assert because T is a pointer
    s2.print();
    Storage s3 { nullptr }; // static_assert because T is a nullptr
    s3.print();
}
```

4. Have `Storage<T*>` make a copy of the object on the heap. Doing all the heap memory management yourself requires overloading the constructor, copy constructor, copy assignment, and destructor. An easier alternative is to just use `std::unique_ptr`

```cpp
#include <iostream>
#include <type_traits> // for std::is_pointer_v and std::is_null_pointer_v
#include <memory>

template <typename T>
class Storage {
    // Make sure T isn't a pointer or a std::nullptr_t
    static_assert(!std::is_pointer_v<T> && !std::is_null_pointer_v<T>, "Storage<T*> and Storage<nullptr> disallowed");

private: T m_value {};
public:
    Storage(T value) : m_value { value } {}
    void print() { std::cout << m_value << '\n'; }
};

template <typename T>
class Storage<T*> {
private:
    std::unique_ptr<T> m_value {}; 
    // use std::unique_ptr to automatically deallocate when Storage is destroyed
public:
    Storage(T* value) : m_value { std::make_unique<T>(value ? *value : 0) } {}
	 // or throw exception when !value
    void print() {
        if (m_value) std::cout << *m_value << '\n';
    }
};

int main() {
    double d { 1.2 };
    Storage s1 { d }; // ok
    s1.print();
    Storage s2 { &d }; // ok, copies d on heap
    s2.print();
}
```

Using partial template class specialization to create separate pointer and non-pointer implementations of a class is extremely useful when you want a class to handle both differently.

---
