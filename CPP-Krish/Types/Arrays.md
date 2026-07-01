Fixed-size arrays require that the length of the array be known at the point of instantiation, and that length cannot be changed afterward. Example: C-style arrays and `std::array`.
Use `std::array` for constexpr arrays, and `std::vector` for non-constexpr arrays.

The length of a `std::array` must be a constant expression.
Perhaps surprisingly, a `std::array` can be defined with a length of 0.

```cpp
std::array<int, 6> fibonnaci = { 0, 1, 1, 2, 3, 5 }; 
// copy-list initialization using braced list
std::array<int, 5> prime { 2, 3, 5, 7, 11 };         
// list initialization using braced list (preferred)

std::array<int, 5> a;   
// Members default initialized (int elements are left uninitialized)
std::array<int, 5> b{}; 
// Members value initialized (int elements are zero initialized) (preferred)
std::vector<int> v(5);  
// Members value initialized (int elements are zero initialized) (for comparison)

std::array<int, 4> a { 1, 2, 3, 4, 5 }; // compile error: too many initializers
std::array<int, 4> b { 1, 2 };  // b[2] and b[3] are value initialized

```

---
### Const and constexpr `std::array`

```cpp
const std::array<int, 5> prime { 2, 3, 5, 7, 11 };
constexpr std::array<int, 5> prime { 2, 3, 5, 7, 11 };
```

Even though the elements of a `const std::array` are not explicitly marked as const, they are still treated as const (because the whole array is const).

Define your `std::array` as constexpr whenever possible. If your `std::array` is not constexpr, consider using a `std::vector` instead.

---
### Class template argument deduction (CTAD) for `std::array` C++17

Using CTAD (class template argument deduction) in C++17, we can have the compiler deduce both the element type and the array length of a `std::array` from a list of initializers:

```cpp
constexpr std::array a1 { 9, 7, 5, 3, 1 }; 
// The type is deduced to std::array<int, 5>
constexpr std::array a2 { 9.7, 7.31 };     
// The type is deduced to std::array<double, 2>
```

CTAD does not support partial omission of template arguments (as of C++23), so there is no way to use a core language feature to omit just the length or just the type of a `std::array`:

```cpp
constexpr std::array<int> a2 { 9, 7, 5, 3, 1 };     
// error: too few template arguments (length missing)
constexpr std::array<5> a2 { 9, 7, 5, 3, 1 };       
// error: too few template arguments (type missing)
```

However, TAD (template argument deduction, used for function template resolution) does support partial omission of template arguments. Since C++20, it is possible to omit the array length of a `std::array` by using the `std::to_array` helper function:


```cpp
constexpr auto myArray1 { std::to_array<int, 5>({ 9, 7, 5, 3, 1 }) }; 
// Specify type and size
constexpr auto myArray2 { std::to_array<int>({ 9, 7, 5, 3, 1 }) };    
// Specify type only, deduce size
constexpr auto myArray3 { std::to_array({ 9, 7, 5, 3, 1 }) };         
// Deduce type and size
```

Unfortunately, using `std::to_array` is more expensive than creating a `std::array` directly, because it involves creation of a temporary `std::array` that is then used to copy initialize our desired `std::array`. For this reason, `std::to_array` should only be used in cases where the type can’t be effectively determined from the initializers.

---
### Length, Size, Indexing

**The length of a `std::array` has type `std::size_t`**.

`std::array` is implemented as a template struct whose declaration looks like this:

```cpp
template<typename T, std::size_t N> // N is a non-type template parameter
struct array;
```

Because this value must be constexpr, we don’t run into sign conversion issues when we use a signed integral value for `N`.

Prior to C++23, C++ didn’t even have a literal suffix for `std::size_t` (`UZ`), as the implicit compile-time conversion from `int` to `std::size_t` typically suffices for cases where we need a constexpr `std::size_t`.The suffix was added primarily for type deduction purposes usiing `auto`.


**The length and indices of `std::array` have type `size_type`, which is _always_ `std::size_t` .**

Just like a `std::vector`, `std::array` defines a nested typedef member named `size_type`, which is an alias for the type used for the length (and indices, if supported) of the container. In the case , `size_type` is _always_ an alias for `std::size_t`.

Note that the non-type template parameter is explicitly defined as `std::size_t` rather than `size_type` because `size_type` is a member of `std::array`, and isn’t defined at that point. This is the only place that uses `std::size_t` explicitly, everywhere else uses `size_type`.

**Getting the length of a `std::array`**

There are three common ways to get the length of a `std::array` object:

1. `size()` member function , returns the length as unsigned `size_type`. Does not have `length()` like most other container types

2. In C++17,  `std::size()` non-member function, which for `std::array` just calls the `size()` member function, thus returning the length as unsigned `size_type`.

3. In C++20, `std::ssize()` non-member function, which returns the length as a large _signed_ integral type (usually`std::ptrdiff_t`)

Because the length of a `std::array` is constexpr, each of the above functions will return the length of a `std::array` as a constexpr value (even when called on a non-constexpr `std::array` object). 

**Subscripting**

The most common way to index a `std::array` is to use the subscript operator (`operator[]`). No bounds checking is done in this case, and passing in an invalid index will result in undefined behavior. Just like `std::vector`, `std::array` also has an `at()` member function that does subscripting with runtime bounds checking. Both of these functions expect the index to be of type `size_type` (`std::size_t`). If either of these functions are called with a constexpr value, the compiler will do a constexpr conversion to `std::size_t`. 

Since the length of a `std::array` is constexpr, if our index is also a constexpr value, then the compiler should be able to validate it at compile-time  (and stop compilation if the constexpr index is out of bounds).  However, `operator[]` does no bounds checking by definition, and the `at()` member function only does runtime bounds checking. And function parameters can’t be constexpr (even for constexpr or consteval functions), so we use non-type templates, the `std::get()` function template. 

Since template arguments must be constexpr, `std::get()` can only be called with constexpr indices.

```cpp
constexpr std::array prime{ 2, 3, 5, 7, 11 };
std::cout << std::get<3>(prime); // print the value of element with index 3
std::cout << std::get<9>(prime); // invalid index (compile error)
```

---
### Passing and returning std::array

```cpp
void passByRef(const std::array<int, 5>& arr)  {}
// we must explicitly specify <int, 5> here as CTAD does not work on parameters.
```

To write a function that can accept `std::array` with any kind of element type or any length, we can create a function template that parameterizes both the element type and length.
Since `std::array` is defined like this:

```cpp
template<typename T, std::size_t N> 
struct array;
```

We can create a function template that uses the same template parameter declaration:

```cpp
template <typename T, std::size_t N> 
void passByRef(const std::array<T, N>& arr) {
    static_assert(N != 0); // fail if this is a zero-length std::array
    std::cout << arr[0] << '\n';
}
```

Note: if you use `int` as the type of the non-type template parameter, the compiler will be unable to match the argument of type `std::array<T, std::size_t>` with the parameter of type `std::array<T, int>` (and templates won’t do conversions).

#### Auto non-type template parameters C++20

In C++20, we can use `auto` in a template parameter declaration to have a non-type template parameter deduce its type from the argument:

```cpp
template <typename T, auto N> 
void passByRef(const std::array<T, N>& arr) {
    static_assert(N != 0); // fail if this is a zero-length std::array
    std::cout << arr[0] << '\n';
}
```

One advantage that template parameters have over function parameters is that template parameters are compile-time constants. So we can `static_assert` the indices ourselves or use `std::get()` inside the function

Unlike `std::vector`, `std::array` is not move-capable, so returning a `std::array` by value will make a copy of the array. The elements inside the array will be moved if they are move-capable, and copied otherwise. In cases where return by value is too expensive, we can use an out-parameter instead. 

`std::vector` is move-capable . If you’re returning a `std::array` by value, your `std::array` probably isn’t constexpr, and you should consider using (and returning) `std::vector` instead.

---
### `std::array` of class types, and brace elision

We will use this
```cpp
struct House {
    int number{};
    int stories{};
    int roomsPerStory{};
};
```

We can do this:
```cpp
std::array<House, 3> houses{};
houses[0] = { 13, 1, 7 };
houses[1] = { 14, 2, 5 };
houses[2] = { 15, 2, 4 };
// The compiler knows that each element of houses is a House
// so it will implicitly convert the right hand side of each assignment to a House
```

Or this:
```cpp
constexpr std::array houses { // use CTAD to deduce <House, 3>
	House{ 13, 1, 7 },
	House{ 14, 2, 5 },
	House{ 15, 2, 4 }
};
```

So you might think to try something like this, but it doesn't work:

```cpp
// doesn't work
constexpr std::array<House, 3> houses { 
// we're telling the compiler that each element is a House
	{ 13, 1, 7 }, // but not mentioning it here
	{ 14, 2, 5 },
	{ 15, 2, 4 }
};
```

A `std::array` is defined as a struct that contains a single C-style array member (whose name is implementation defined), like this:

```cpp
template<typename T, std::size_t N>
struct array {
    T implementation_defined_name[N]; 
    // a C-style array with N elements of type T
}
```

So when we try to initialize `houses` per the above, the compiler interprets the initialization like this:
```cpp
constexpr std::array<House, 3> houses { // initializer for houses
    { 13, 1, 7 }, // initializer for C-style array member
    { 14, 2, 5 }, // ?
    { 15, 2, 4 }  // ?
};
```

The compiler will interpret `{ 13, 1, 7 }` as the initializer for the first member of `houses`, which is the C-style array with the implementation defined name. This will initialize the C-style array element 0 with `{ 13, 1, 7 }` and the rest of the members will be zero-initialized. Then the compiler will discover we’ve provided two more initialization values (`{ 14, 2, 7 }` and `{ 15, 2, 5 }`) and produce a compilation error telling us that we’ve provided too many initialization values.

Now add an extra set of braces as follows:

```cpp
// This works as expected
constexpr std::array<House, 3> houses { // initializer for houses
    { // extra set of braces to initialize the C-style array member 
        { 13, 4, 30 }, // initializer for array element 0
        { 14, 3, 10 }, // initializer for array element 1
        { 15, 3, 40 }, // initializer for array element 2
    }
};
```

Note the extra set of braces that are required (to begin initialization of the C-style array member inside the `std::array` struct). Within those braces, we can then initialize each element individually, each inside its own set of braces.

Given the explanation above, you may be wondering why the above case requires double braces, but all other cases we’ve seen only require single braces:

```cpp
constexpr std::array<int, 5> arr { 1, 2, 3, 4, 5 }; // single braces
```

It turns out that you can supply double braces for such arrays:

```cpp
constexpr std::array<int, 5> arr {{ 1, 2, 3, 4, 5 }}; // double braces
```

However, aggregates in C++ support a concept called **brace elision**, which lays out some rules for when multiple braces may be omitted. Generally, you can omit braces when initializing a `std::array` with scalar (single) values, or when initializing with class types or arrays where the type is explicitly named with each element.

There is no harm in always initializing `std::array` with double braces, as it avoids having to think about whether brace-elision is applicable in a specific case or not. Alternatively, you can try to single-brace init, and the compiler will generally complain if it can’t figure it out. In that case, you can quickly add an extra set of braces.

---
### Arrays of references via `std::reference_wrapper`

We’ll use `std::array` in the examples, but this is equally applicable to all array types.

You can make an array of pointers, however, because references are not objects, you cannot make an array of references. The elements of an array must also be assignable, and references can’t be reseated.

```cpp
int x { 1 }, y { 2 };

std::array<int&, 2> refarr { x, y }; 
// compile error: cannot define array of references

int& ref1 { x }, ref2 { y };
std::array valarr { ref1, ref2 };
 // ok: this is actually a std::array<int, 2>, not an array of references
```

#### `std::reference_wrapper`

`std::reference_wrapper` is a standard library class template that lives in the `<functional> `header. It takes a type template argument T, and then behaves like a modifiable lvalue reference to T. There are a few things worth noting about `std::reference_wrapper`:
- `Operator=` will reseat a `std::reference_wrapper` (change which object is being referenced).
- `std::reference_wrapper<T>` will implicitly convert to `T&`.
- The `get()` member function can be used to get a `T&`. This is useful when we want to update the value of the object being referenced.

```cpp
int x { 1 }, y { 2 }, z { 3 };
std::array<std::reference_wrapper<int>, 3> arr { x, y, z };
arr[1].get() = 5; // modify the object in array element 1
std::cout << arr[1] << y << '\n'; 
// show that we modified arr[1] and y, prints 55

```

Note that we must use `arr[1].get() = 5` and not `arr[1] = 5`. The latter is ambiguous, as the compiler can’t tell if we intend to reseat the `std::reference_wrapper<int>` to value 5 (something that is illegal anyway) or change the value being referenced. Using `get()` disambiguates this.

When printing `arr[1]`, the compiler will realize it can’t print a `std::reference_wrapper<int>`, so it will implicitly convert it to an `int&`, which it can print. So we don’t need to use `get()` here.

### `std::ref` and `std::cref`

Prior to C++17, CTAD (class template argument deduction) didn’t exist, so all template arguments for a class type needed to be listed explicitly. Thus, to create a `std::reference_wrapper<int>`, you could do either of these:

```cpp
int x { 5 };
std::reference_wrapper<int> ref1 { x };        // C++11
auto ref2 { std::reference_wrapper<int>{ x }}; // C++11
```

To make things easier, the `std::ref()` and `std::cref()` functions were provided as shortcuts to create `std::reference_wrapper` and `const std::reference_wrapper` wrapped objects. Note that these functions can be used with `auto` to avoid having to explicitly specify the template argument.

```cpp
int x { 5 };
auto ref { std::ref(x) };   // C++11, deduces to std::reference_wrapper<int>
auto cref { std::cref(x) }; // C++11, deduces to std::reference_wrapper<const int>
```

Of course, now that we have CTAD in C++17, we can also do this:

```cpp
std::reference_wrapper ref1 { x };        // C++17
auto ref2 { std::reference_wrapper{ x }}; // C++17
```

But since `std::ref()` and `std::cref()` are shorter to type, they are still widely used to create `std::reference_wrapper` objects

---
### std::array and enumerations

When initializing a `constexpr std::array` using CTAD, the compiler will deduce how long the array should be from the number of initializers. If less initializers are provided than there should be, the array will be shorter than expected, and indexing it can lead to undefined behavior.

Whenever the number of initializers in a `constexpr std::array` can be reasonably sanity checked, you can do so using a static assert:

```cpp
enum StudentNames {
    kenny, // 0
    kyle, // 1
    stan, // 2
    butters, // 3
    cartman, // 4
    max_students // 5
};

int main() {
    constexpr std::array testScores { 78, 94, 66, 77 };
    static_assert(std::size(testScores) == max_students); 
    // compile error: static_assert condition failed
    std::cout  << testScores[StudentNames::cartman] << '\n';
}
```

We covered a few ways to input and output the names of enumerators. We had helper functions that converted an enumerator to a string and vice-versa. These functions each had their own (duplicate) set of string literals, and we had to specifically code logic to check each:

```cpp
constexpr std::string_view getPetName(Pet pet) {
    switch (pet) {
    case cat:   return "cat";
    case dog:   return "dog";
    case pig:   return "pig";
    case whale: return "whale";
    default:    return "???";
    }
}

constexpr std::optional<Pet> getPetFromString(std::string_view sv) {
    if (sv == "cat")   return cat;
    if (sv == "dog")   return dog;
    if (sv == "pig")   return pig;
    if (sv == "whale") return whale;
    return {};
}
```

Let’s improve these functions a bit. In cases where the value of our enumerators start at 0 and proceed sequentially (which is true for most enumerations), we can use an array to hold the name of each enumerator.

```cpp

namespace Color {
    enum Type {
        black,
        red,
        blue,
        max_colors
    };
    using namespace std::string_view_literals; // for sv suffix
    constexpr std::array colorName { "black"sv, "red"sv, "blue"sv };
    // use sv suffix so std::array will infer type as std::string_view
    static_assert(std::size(colorName) == max_colors);
};

constexpr std::string_view getColorName(Color::Type color) {
    return Color::colorName[static_cast<std::size_t>(color)];
}

std::ostream& operator<<(std::ostream& out, Color::Type color) {
    return out << getColorName(color);
}

std::istream& operator>> (std::istream& in, Color::Type& color) {
    std::string input {};
    std::getline(in >> std::ws, input);
    // Iterate through the list of names to see if we can find a matching name
    for (std::size_t index=0; index < Color::colorName.size(); ++index) {
        if (input == Color::colorName[index]) {
            color = static_cast<Color::Type>(index);
            return in;
        }
    }

    // We didn't find a match, so input must have been invalid
    // so we will set input stream to fail state
    in.setstate(std::ios_base::failbit);

    // On an extraction failure, operator>> zero-initializes fundamental types
    // Uncomment the following line to make this operator do the same thing
    // color = {};
    return in;
}

int main() {
    auto shirt{ Color::blue };
    std::cout << "Your shirt is " << shirt << '\n';
    std::cout << "Enter a new color: ";
    std::cin >> shirt;
    if (!std::cin) std::cout << "Invalid\n";
    else std::cout << "Your shirt is now " << shirt << '\n';
}
```


To iterate through the enumerators of an enumeration we require a lot of static casting of the integer index to our enumeration type.

```cpp
for (int i=0; i < Color::max_colors; ++i )
	std::cout << static_cast<Color::Type>(i) << '\n';
```

Unfortunately, range-based for-loops won’t allow you to iterate over the enumerators of an enumeration:

```cpp
for (auto c: Color::Type) // compile error: can't traverse enumeration
	std::cout << c < '\n';
```

Since we can use a range-based for-loop on an array, one of the  solutions is to create a `constexpr std::array` containing each of our enumerators, and then iterate over that. This method only works if the enumerators have unique values.

```cpp
namespace Color {
    enum Type {
        black,     // 0
        red,       // 1
        blue,      // 2
        max_colors // 3
    };

    using namespace std::string_view_literals; // for sv suffix
    constexpr std::array colorName { "black"sv, "red"sv, "blue"sv };
    static_assert(std::size(colorName) == max_colors);

    constexpr std::array types { black, red, blue }; 
    // A std::array containing all our enumerators
    static_assert(std::size(types) == max_colors);
};

constexpr std::string_view getColorName(Color::Type color) {
    return Color::colorName[color];
}

std::ostream& operator<<(std::ostream& out, Color::Type color) {
    return out << getColorName(color);
}

int main() {
    for (auto c: Color::types) // ok: we can do a range-based for on a std::array
        std::cout << c << '\n';
}
```

---
### Multidimensional std::array

There is no standard library multidimensional array class. But we can to something like this:

```cpp
std::array<std::array<int, 4>, 3> arr {{  // note double braces
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
}};
```

- When initializing a multidimensional `std::array`, we need to use double-braces (we discuss why earlier)
- Because of the way templates get nested, the array dimensions are switched, but indexing works the same.

We can also pass a two-dimensional `std::array` to a function just like a one-dimensional `std::array`:

```cpp
template <typename T, std::size_t Row, std::size_t Col>
void printArray(const std::array<std::array<T, Col>, Row> &arr) {
    for (const auto& arow: arr) {
        for (const auto& e: arow) 
            std::cout << e << ' ';
        std::cout << '\n';
    }
}

int main() {
    std::array<std::array<int, 4>, 3>  arr {{
        { 1, 2, 3, 4 },
        { 5, 6, 7, 8 },
        { 9, 10, 11, 12 }}};
    printArray(arr);
}
```

Making two-dimensional `std::array` easier using an alias templates. Example:

```cpp
template <typename T, std::size_t Row, std::size_t Col>
using Array2d = std::array<std::array<T, Col>, Row>;

template <typename T, std::size_t Row, std::size_t Col>
void printArray(const Array2d<T, Row, Col> &arr) {
    for (const auto& arow: arr) {
        for (const auto& e: arow)
            std::cout << e << ' ';
    }
}
```

In order to get the length of the first dimension, we call `size()` on the array. 
To get the length of the second dimension, we first call `arr[0].size()`
To get the length of the third dimension of a 3-dimensional array, we call `arr[0][0].size()`.
But this will produce undefined behavior if any dimension other than the last has a length of 0.
A better option is to use a function template to return the length of the dimension directly from the associated non-type template parameter:

```cpp

template <typename T, std::size_t Row, std::size_t Col>
using Array2d = std::array<std::array<T, Col>, Row>;

template <typename T, std::size_t Row, std::size_t Col>
constexpr int rowLength(const Array2d<T, Row, Col>&) { return Row; }
    
template <typename T, std::size_t Row, std::size_t Col>
constexpr int colLength(const Array2d<T, Row, Col>&) { return Col; }
// you can return std::size_t if you prefer
    
int main() {
    Array2d<int, 3, 4> arr {{
        { 1, 2, 3, 4 },
        { 5, 6, 7, 8 },
        { 9, 10, 11, 12 }
    }};
    std::cout << "Rows: " << rowLength(arr) << '\n'; 
    std::cout << "Cols: " << colLength(arr) << '\n'; 
}
```

This also allows us to easily return the length as an `int` if we desire (no static_cast is required, as converting from a `constexpr std::size_t` to `constexpr int` is non-narrowing, so an implicit conversion is fine).

One approach to make multidimensional arrays easier to work with is to flatten them.
We can also provide an interface that mimics a multidimensional array. This interface will accept two-dimensional coordinates, and then map them to a unique position in the one-dimensional array.

```cpp
// An alias template to allow us to define a one-dimensional std::array using two dimensions
template <typename T, std::size_t Row, std::size_t Col>
using ArrayFlat2d = std::array<T, Row * Col>;

// A modifiable view that allows us to work with an ArrayFlat2d using two dimensions
// This is a view, so the ArrayFlat2d being viewed must stay in scope
template <typename T, std::size_t Row, std::size_t Col>
class ArrayView2d {
private:
    // You might be tempted to make m_arr a reference to an ArrayFlat2d,
    // but this makes the view non-copy-assignable since references can't be reseated.
    // Using std::reference_wrapper gives us reference semantics and copy assignability.
    std::reference_wrapper<ArrayFlat2d<T, Row, Col>> m_arr {};

public:
    ArrayView2d(ArrayFlat2d<T, Row, Col> &arr)
        : m_arr { arr }
    {}

    // Get element via single subscript (using operator[])
    T& operator[](int i) { return m_arr.get()[static_cast<std::size_t>(i)]; }
    const T& operator[](int i) const { return m_arr.get()[static_cast<std::size_t>(i)]; }

    // Get element via 2d subscript (using operator(), since operator[] doesn't support multiple dimensions prior to C++23)
    T& operator()(int row, int col) { 
	    return m_arr.get()[static_cast<std::size_t>(row * cols() + col)]; 
	}
    const T& operator()(int row, int col) const { 
	    return m_arr.get()[static_cast<std::size_t>(row * cols() + col)]; 
	}

// in C++23, you can uncomment these since multidimensional operator[] is supported
//    T& operator[](int row, int col) { return m_arr.get()[static_cast<std::size_t>(row * cols() + col)]; }
//    const T& operator[](int row, int col) const { return m_arr.get()[static_cast<std::size_t>(row * cols() + col)]; }

    int rows() const { return static_cast<int>(Row); }
    int cols() const { return static_cast<int>(Col); }
    int length() const { return static_cast<int>(Row * Col); }
};

int main() {
    ArrayFlat2d<int, 3, 4> arr {
        1, 2, 3, 4,
        5, 6, 7, 8,
        9, 10, 11, 12 
    };

    // Define a two-dimensional view into our one-dimensional array
    ArrayView2d<int, 3, 4> arrView { arr };
    
    std::cout << "Rows: " << arrView.rows() << '\n';
    std::cout << "Cols: " << arrView.cols() << '\n';
    
    for (int i=0; i < arrView.length(); ++i)
        std::cout << arrView[i] << ' ';

    for (int row=0; row < arrView.rows(); ++row) {
        for (int col=0; col < arrView.cols(); ++col)
            std::cout << arrView(row, col) << ' ';
        std::cout << '\n';
    }
}

// Rows: 3
// Cols: 4
// 1 2 3 4 5 6 7 8 9 10 11 12
// 1 2 3 4
// 5 6 7 8
// 9 10 11 12
```

Because `operator[]` can only accept a single subscript prior to C++23, there are two alternate approaches:
- Use `operator()` instead, which can accept multiple subscripts. This lets you use `[]` for single index indexing and `()` for multiple-dimension indexing. We’ve opted for this approach above.
- Have `operator[]` return a view that also overloads `operator[]` so that you can chain `operator[]`. This is more complex and doesn’t scale well to higher dimensions (kinda makes sense).

In C++23, `operator[]` was extended to accept multiple subscripts, so you can overload it to handle both single and multiple subscripts (instead of using `operator()` for multiple subscripts).

---
### `std::mdspan`

Introduced in C++23, `std::mdspan` is a modifiable view that provides a multidimensional array interface for a contiguous sequence of elements. By modifiable view,  if the underlying sequence of elements is non-const, those elements can be modified.

The following example prints the same output as the prior example, but uses `std::mdspan` instead of our own custom view:

```cpp
#include <mdspan>

template <typename T, std::size_t Row, std::size_t Col>
using ArrayFlat2d = std::array<T, Row * Col>;

int main() {
    // Define a one-dimensional std::array of int (with 3 rows and 4 columns)
    ArrayFlat2d<int, 3, 4> arr {
        1, 2, 3, 4,
        5, 6, 7, 8,
        9, 10, 11, 12 
    };

    // Define a two-dimensional span into our one-dimensional array
    // We must pass std::mdspan a pointer to the sequence of elements
    // which we can do via the data() member function of std::array or std::vector
    std::mdspan mdView { arr.data(), 3, 4 };

    // print array dimensions
    // std::mdspan calls these extents
    std::size_t rows { mdView.extents().extent(0) };
    std::size_t cols { mdView.extents().extent(1) };
    std::cout << "Rows: " << rows << '\n';
    std::cout << "Cols: " << cols << '\n';

    // print array in 1d
    // The data_handle() member gives us a pointer to the sequence of elements
    // which we can then index
    for (std::size_t i=0; i < mdView.size(); ++i)
        std::cout << mdView.data_handle()[i] << ' ';
    std::cout << '\n';

    // print array in 2d
    // We use multidimensional [] to access elements
    for (std::size_t row=0; row < rows; ++row) {
        for (std::size_t col=0; col < cols; ++col)
            std::cout << mdView[row, col] << ' ';
        std::cout << '\n';
    }
}
```

`std::mdspan` let us define a view with as many dimensions as we want.
The first parameter to the constructor of `std::mdspan` should be a pointer to the array data. This can be a decayed C-style array, or we can use the `data()` member function of `std::array` or `std::vector` to get this data.
To index a `std::mdspan` in one-dimension, we must fetch the pointer to the array data, which we can do using the `data_handle()` member function. We can then subscript that.
In C++23, `operator[]` accepts multiple indices, so we use `[row, col]` as our index instead of `[row][col]`.

C++26 will include `std::mdarray`, which essentially combines `std::array` and `std::mdspan` into an owning multidimensional array.

---
