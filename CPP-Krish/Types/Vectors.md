### Initializing a `std::vector` with a list of values

We can do this by using list initialization:

```cpp
// List construction (uses list constructor)
std::vector<int> primes{ 2, 3, 5, 7 };          
std::vector vowels { 'a', 'e', 'i', 'o', 'u' }; 
// Uses CTAD (C++17) to deduce element type char (preferred).
```

Containers typically have a special constructor called a **list constructor** that allows us to construct an instance of the container using an initializer list. The list constructor does three things:
- Ensures the container has enough storage to hold all the initialization values (if needed).
- Sets the length of the container to the number of elements in the initializer list (if needed).
- Initializes the elements to the values in the initializer list (in sequential order).

We discuss adding list constructors to your own program-defined classes in lesson [23.7 -- std::initializer_list](https://www.learncpp.com/cpp-tutorial/stdinitializer_list/).

`std::vector` has an explicit constructor (`explicit std::vector<T>(std::size_t)`) that takes a single `std::size_t` value defining the length of the `std::vector` to construct:

```cpp
std::vector<int> data( 10 ); 
// vector containing 10 int elements, value-initialized to 0
```

Each of the created elements are value-initialized, which for `int` does zero-initialization (and for class types calls the default constructor). However, there is one non-obvious thing about using this constructor: it must be called using direct initialization. To understand why, consider:

```cpp
std::vector<int> data{ 10 }; // what does this do?
```

There are two different constructors that match this initialization:
- `{ 10 }` can be interpreted as an initializer list, and matched with the list constructor to construct a vector of length 1 with value 10.
- `{ 10 }` can be interpreted as a single braced initialization value, and matched with the `std::vector<T>(std::size_t)` constructor to construct a vector of length 10 with elements value-initialized to 0.

Normally when a class type definition matches more than one constructor, the match is considered ambiguous and a compilation error results. However, C++ has a special rule for this case: **When an initializer list is non-empty, a matching list constructor will be selected over other matching constructors. Without this rule, a list constructor would result in an ambiguous match with any constructor that took arguments of a single type.**

Since `{ 10 }` can be interpreted as an initializer list and `std::vector` has a list constructor, the list constructor takes precedence in this case.

**When constructing a class type object using a initializer list:**
- **If the initializer list is empty, the default constructor is preferred over the list constructor.**
- **If the initializer list is non-empty, a matching list constructor is preferred over other matching constructors.**

Let’s look at similar cases using copy, direct, and list initialization:

```cpp
// Copy init
std::vector<int> v1 = 10; // 10 not an initializer list, copy init won't match explicit constructor: compilation error

// Direct init
std::vector<int> v2(10);  // 10 not an initializer list, matches explicit single-argument constructor

// List init
std::vector<int> v3{ 10 }; // { 10 } interpreted as initializer list, matches list constructor

// Copy list init
std::vector<int> v4 = { 10 }; // { 10 } interpreted as initializer list, matches list constructor
std::vector<int> v5({ 10 });  // { 10 } interpreted as initializer list, matches list constructor

// Default init
std::vector<int> v6 {}; // {} is empty initializer list, matches default constructor
std::vector<int> v7 = {};  // {} is empty initializer list, matches default constructor
```

In case `v1`, the initialization value of `10` is not an initializer list, so the list constructor isn’t a match. The single-argument constructor `explicit std::vector<T>(std::size_t)` won’t match either because copy initialization won’t match explicit constructors. Since no constructors match, this is a compilation error.

This is one of the warts of C++ initialization: `{ 10 }` will match a list constructor if one exists, or a single-argument constructor if a list constructor doesn’t exist. This means which behavior you get depends on whether a list constructor exists! You can generally assume containers have list constructors.

When a `std::vector` is a member of a class type, it is not obvious how to provide a default initializer that sets the length of a `std::vector` to some initial value:

```cpp
struct Foo {
    std::vector<int> v1(8); // compile error: direct initialization not allowed for member default initializers
};
```

This doesn’t work because direct (parenthesis) initialization is disallowed for member default initializers. When providing a default initializer for a member of a class type:
- We must use either copy initialization or list initialization (direct or copy).
- CTAD is not allowed (so we must explicitly specify the element type).

The answer is as follows:

```cpp
struct Foo {
    std::vector<int> v{ std::vector<int>(8) }; // ok
};
```

---
### Const and Constexpr Vectors

Objects of type `std::vector` can be made `const`:

```cpp
const std::vector<int> prime { 2, 3, 5, 7, 11 }; 
// its elements cannot be modified
```

A `const std::vector` must be initialized, and then cannot be modified. 
The elements of such a vector are treated as if they were const. 
The element type of a `std::vector` must not be defined as const (e.g. `std::vector<const int>` is disallowed).

The standard library containers were not designed to have const elements.
A containers const-ness comes from const-ing the container itself, not the elements. 

One of the biggest downsides of `std::vector` is that it cannot be made `constexpr`. 
If you need a `constexpr` array, use `std::array`.

---
### Indexing `std::vector`

Each of the standard library container classes defines a nested typedef member named `size_type` (sometimes written as `T::size_type`), which is an alias for the type used for the length (and indices, if supported) of the container.

`size_type` is almost always an alias for `std::size_t`, but can be overridden (in rare cases) to use a different type. `std::size_t` is itself a typedef for some large unsigned integral type, usually `unsigned long` or `unsigned long long`.

When accessing the `size_type` member of a container class, we must scope qualify it with the fully templated name of the container class. For example, `std::vector<int>::size_type`.

In C++17, we can also use the `std::size()` non-member function (which for container classes just calls the `size()` member function).

C++20 introduces the `std::ssize()` non-member function, which returns the length as a large _signed_ integral type (usually `std::ptrdiff_t`, which is the type normally used as the signed counterpart to `std::size_t`):

The array container classes support another method for accessing an array. The `at()` member function can be used to do array access with runtime bounds checking. When the `at()` member function encounters an out-of-bounds index, it actually throws an exception of type `std::out_of_range`. If the exception is not handled, the program will be terminated.

```cpp
std::vector prime{ 2, 3, 5, 7, 11 };
int index { 3 };  // non-constexpr
std::cout << prime[index] << '\n'; // possible warning: index implicitly converted to std::size_t, narrowing conversion
```

You can give constexpr indices and they wont considered narrowing due to constexpr clause.
Also, the warning is a little useless becuase positive signed integer can fit in their unsigned versions

Another good alternative is instead of indexing the `std::vector` itself, index the result of the `data()` member function:

```cpp
std::vector prime{ 2, 3, 5, 7, 11 };
int index { 3 };                          // non-constexpr signed value
std::cout << prime.data()[index] << '\n'; // okay: no sign conversion warnings
```

Under the hood, `std::vector` holds its elements in a C-style array. The `data()` member function returns a pointer to this underlying C-style array, which we can then index. Since C-style arrays allow indexing with both signed and unsigned types, we don’t run into any sign conversion issues.

---
### Passing `std::vector`

```cpp
void passByRef(const std::vector<int>& arr) {}
 // we must explicitly specify <int> here
```

Can't pass this into above:  `std::vector dbl{ 1.1, 2.2, 3.3 };`

```cpp
void passByRef(const std::vector& arr) {}
// compile error: CTAD can't be used to infer function parameters
```

Even the above doesn't work. So use templates

```cpp
template <typename T>
void passByRef(const std::vector<T>& arr) {}
```

Even generaller:

```cpp
template <typename T>
void passByRef(const T& arr) {
	// will accept any type of object that has an overloaded operator[]
    std::cout << arr[0] << '\n';
}
```

In C++20, we can use an abbreviated function template (via an `auto` parameter) to do the same thing:

```cpp
void passByRef(const auto& arr) {
	// abbreviated function template
	std::cout << arr[0] << '\n';
}
```

Accessing index like this is risky, so assert the size first, but `std::vector::size()` is a non-constexpr function, we can only do a runtime assert here. A better option is to avoid using `std::vector`in cases where you need to assert on array length. Using a type that supports `constexpr` arrays (e.g. `std::array`) is probably a better choice, as you can `static_assert` on the length of a constexpr array.

---
### Returning `std::vector`

**Copy semantics** refers to the rules that determine how copies of objects are made. When we say a type supports copy semantics, we mean that objects of that type are copyable, because the rules for making such copies have been defined. When we say copy semantics are being invoked, that means we’ve done something that will make a copy of an object.

For class types, copy semantics are typically implemented via the copy constructor (and copy assignment operator), which defines how objects of that type are copied. Typically this results in each data member of the class type being copied. 

When copy semantics is not optimal:

```cpp

std::vector<int> generate() {      // return by value
    // We're  using a named object here so mandatory copy elision doesn't apply
    std::vector arr1 { 1, 2, 3, 4, 5 }; // copies { 1, 2, 3, 4, 5 } into arr1
    return arr1;
}

int main() {
    std::vector arr2 { generate() }; 
    // A temporary object is created which is then copied into `arr2`
}
```

Instead, what if there was a way for `arr2` to steal the temporary’s data instead of copying it? `arr2` would then be the new owner of the data, and no copy of the data would need to be made. When ownership of data is transferred from one object to another, we say that data has been **moved**. The cost of such a move is typically trivial (usually just two or three pointer assignments). As an added benefit, when the temporary was then destroyed at the end of the expression, it would no longer have any data to destroy, so we wouldn’t have to pay that cost either.

This is the essence of **move semantics**, which refers to the rules that determine how the data from one object is moved to another object. When move semantics is invoked, any data member that can be moved is moved, and any data member that can’t be moved is copied

When all of the following are true, move semantics will be invoked instead:
- The type of the object supports move semantics.
- The object is being initialized with (or assigned) an rvalue (temporary) object of the same type.
- The move isn’t elided.

Here’s the sad news: not that many types support move semantics. However, `std::vector` and `std::string` both do. Therefore we can return move-capable types like `std::vector` by value


One of the most common things we do in C++ is pass a value to some function, and get a different value back. When the passed values are class types, that process involves 4 steps:

1. Construct the value to be passed.
2. Actually pass the value to the function.
3. Construct the value to be returned.
4. Actually pass the return value back to the caller.

```cpp
std::vector<int> doSomething(std::vector<int> v2) {
    std::vector v3 { v2[0] + v2[0] }; 
    // 3 -- construct value to be returned to caller
    return v3; // 4 -- actually return value
}

int main() {
    std::vector v1 { 5 }; // 1 -- construct value to be passed to function
    std::cout << doSomething(v1)[0] << '\n'; // 2 -- actually pass value
    std::cout << v1[0] << '\n';
}
```

First, let’s assume `std::vector` is not move-capable. In that case, the above program makes 4 copies:

1. Constructing the value to be passed copies the initializer list into `v1`
2. Actually passing the value to the function copies argument `v1` into function parameter `v2`.
3. Constructing the value to be returned copies the initializer into `v3`
4. Actually returning the value to the caller copies `v3` back to the caller.

We have many tools at our disposal here to optimize:
pass by reference or address, elision, move semantics, and out parameters.

We can’t optimize copies 1 and 3 at all. We need a `std::vector` to pass to the function, and we need a `std::vector` to return, these objects have to be constructed. `std::vector` is an owner of its data, so it necessarily makes a copy of its initializer.

Copy 2 is made because we’re passing by value from the caller to the called function. What other options do we have?

- Can we pass by reference or address? Yes.
- Can this copy be elided? No. Elision only works when we’re making a redundant copy or move. There’s no redundant copy or move here.
- Can we use an out parameter here? No. We’re passing a value to the function, not getting a value back.
- Can we use move semantics here? No. The argument is an lvalue. If we moved data from `v1` to `v2`, `v1` would become an empty vector, and subsequently printing `v1[0]` would lead to undefined behavior.

Copy 4 is made because we’re passing by value from the called function back to the caller. What other options do we have here?

- Can we return by reference or address? No, it will result in the caller receiving a dangling pointer or reference.
- Can this copy be elided? Yes, possibly. Compiler mayl realize that we’re constructing an object in the scope of the called function and returning it. By rewriting the code (under the as-if rule) so that `v3` is constructed in the scope of the caller instead. However, we are reliant upon the compiler realizing it can do this, so it is not guaranteed.
- Can we use an out parameter here? Yes. We can pass an empty `v3` it to the function by non-const reference. This avoids the copy, but also has some significant downsides and constraints:  doesn’t work with objects that don’t support assignment, and it is challenging to write such functions that can work with both lvalue and rvalue arguments.
- Can we use move semantics here? Yes.

---
### Arrays, loops, and sign challenge solutions

#### Unsigned iteration
Conisder: 
```cpp
template <typename T>
void printReverse(const std::vector<T>& arr) {
    for(std::size_t index{arr.size()-1}; index>=0; --index) {
	     // index is unsigned
        std::cout << arr[index] << ' ';
    }
}
```

When we decrement `index` when it has value `0`, it will wrap around to a large positive value, which we then use to index the array on the next iteration. This is an out-of-bounds index, and will cause undefined behavior. 

We noted that the standard library container classes define nested typedef `size_type`, which is an unsigned integral type used for array lengths and indices. The `size()` member function returns `size_type`, and `operator[]` uses `size_type` as an index, so using `size_type` as the type of your index is technically the most consistent and safe unsigned type to use. 

To use it we have to explicitly prefix the name with the fully templated name of the container (meaning we have to type `std::vector<int>::size_type` rather than just `std::size_type`). 
Also, still doesn't solve reverse iteration cleanly.


You may occasionally see the array type aliased to make the loop easier to read:

```cpp
using arrayi = std::vector<int>;
for (arrayi::size_type index { 0 }; index < arr.size(); ++index)
```

A more general solution is to have the compiler fetch the type of the array type object for us, so that we don’t have to explicitly specify the container type or template arguments. To do so, we can use the **decltype** keyword, which returns the type of its parameter.

```cpp
// arr is some non-reference type
for (decltype(arr)::size_type index { 0 }; index < arr.size(); ++index) // decltype(arr) resolves to std::vector<int>
```

However, if `arr` is a reference type (e.g. an array passed by reference), the above doesn’t work. We need to first remove the reference from `arr`:

```cpp
for (typename std::remove_reference_t<decltype(arr)>::size_type index { 0 }; index < arr.size(); ++index) {
		std::cout << arr[index] << ' ';
}
```

---
#### Signed Iteration

If we have decided to use signed type, there are three (sometimes four) good options here:
1. Unless you are working with a very large array, using `int` should be fine (particularly on architectures where int is 4 bytes). `int` is the default signed integral type we use for everything when we don’t really care about the type otherwise, and there’s little reason to do otherwise here.
2. If you are dealing with very large arrays, or if you want to be a bit more defensive, you can use `std::ptrdiff_t`. This typedef is often used as  signed counterpart to `std::size_t`.
3. Because `std::ptrdiff_t` has a weird name, another good approach is to define your own type alias for indices:

```cpp
using Index = std::ptrdiff_t;
for (Index index{ 0 }; index < static_cast<Index>(arr.size()); ++index)
```

4. In cases where you can derive the type of your loop variable from the initializer, you can use `auto` to have the compiler deduce the type:

```cpp
for (auto index{ static_cast<std::ptrdiff_t>(arr.size())-1 }; index >= 0; --index)
```

In C++23, the `Z` suffix can be used to define a literal of the type that is the signed counterpart to `std::size_t` (probably `std::ptrdiff_t`):

```cpp
for (auto index{ 0Z }; index < static_cast<std::ptrdiff_t>(arr.size()); ++index)
```

Getting the length of an array as a signed value

1. Pre-C++20, the best option is to `static_cast` the return value of the `size()` member function or `std::size()` to a signed type:
2. In C++20, use `std::ssize()`:  This function returns the size of an array type as a signed type (likely `ptrdiff_t`).

```cpp
for (auto index{ std::ssize(arr)-1 }; index >= 0; --index) {}
```

We need some way to convert our signed loop variable to an unsigned value wherever we intend to use it as an index, to avoid warnings.
1. `static_cast`
2. Use a conversion function with a short name:

```cpp
using Index = std::ptrdiff_t;

// Helper function to convert `value` into an object of type std::size_t
// UZ is the suffix for literals of type std::size_t.
template <typename T>
constexpr std::size_t toUZ(T value) {
    // make sure T is an integral type
    static_assert(std::is_integral<T>() || std::is_enum<T>());
    return static_cast<std::size_t>(value);
}

auto length { static_cast<Index>(arr.size()) }; // in C++20, prefer std::ssize()
for (auto index{ length-1 }; index >= 0; --index)
    std::cout << arr[toUZ(index)] << ' '; 
```

3. Use a custom view: While we can’t modify the standard library containers to accept a signed integral index, we can create our own custom view class to “view” a standard library container class. And in doing so, we can define our own interface to work however we want.

In the following example, we define a custom view class that can view any standard library container that supports indexing. Our interface will do two things:
- Allow us to access elements using `operator[]` with a signed integral type.
- Get the length of the container as a signed integral type (since `std::ssize()` is only available on C++20).


```cpp
template <typename T>
class SignedArrayView { // requires C++17
private:
    T& m_array;
public:
    using Index = std::ptrdiff_t;
    SignedArrayView(T& array) : m_array{ array } {}
    // Overload operator[] to take a signed index
    constexpr auto& operator[](Index index) { 
	    return m_array[static_cast<typename T::size_type>(index)]; 
    }
    constexpr const auto& operator[](Index index) const { 
	    return m_array[static_cast<typename T::size_type>(index)]; 
	}
    constexpr auto ssize() const { 
	    return static_cast<Index>(m_array.size()); 
	}
};
```

```cpp
std::vector arr{ 9, 7, 5, 3, 1 };
SignedArrayView sarr{ arr }; // Create a signed view of our std::vector
for (auto index{ sarr.ssize() - 1 }; index >= 0; --index)
	std::cout << sarr[index] << ' '; // index using a signed type
```


We noted that instead of indexing the standard library container, we can instead call the `data()` member function and index that instead. 

```cpp
auto length { static_cast<Index>(arr.size()) }; // in C++20, prefer std::ssize()
for (auto index{ length - 1 }; index >= 0; --index)
    std::cout << arr.data()[index] << ' '; 
```

---
### Range-based for loops in reverse 

Range-based for loops (`auto x : arr`) only iterate in forwards order. As of C++20, you can use the `std::views::reverse` capability of the Ranges library to create a reverse view of the elements that can be traversed:

```cpp
#include <ranges> // C++20
std::vector<std::string_view> words{ "Alex", "Bobby", "Chad", "Dave" }; 
for (const auto& word : std::views::reverse(words)) // create a reverse view
	std::cout << word << ' ';
std::cout << '\n';
```

---
### Array indexing and length using enumerators

Since un-scoped enumerations will implicitly convert to a `std::size_t`, this means we can use un-scoped enumerations as array indices to help document the meaning of the index:

```cpp
namespace Students {
    enum Names {
        kenny, // 0
        kyle, // 1
        stan, // 2
        butters, // 3
        cartman, // 4
	    // add future enumerators here
        max_students // 5
    };
}

std::vector testScores { 78, 94, 66, 77, 14 };
testScores[Students::stan] = 76; 
// we are now updating the test score belonging to stan
```

Because enumerators are implicitly constexpr, conversion of an enumerator to an unsigned integral type is not considered a narrowing conversion, thus avoiding signed/unsigned indexing problems.

However, if we define a non-constexpr variable of the enumeration type, and then try to index our `std::vector` with that, we may get sign conversion warnings on any platform that defaults un-scoped enumerations to a signed type:

```cpp
#include <vector>

std::vector testScores { 78, 94, 66, 77, 14 };
Students::Names name { Students::stan }; // non-constexpr
testScores[name] = 76; // may trigger a sign conversion warning if Student::Names defaults to a signed underlying type
```

In this particular case, we could make `name` constexpr (so that the conversion from a constexpr signed integral type to `std::size_t` is non-narrowing). However, that won’t work when our initializer isn’t a constant expression.

An alternate option is to explicitly specify the underlying type of the enumeration to be an unsigned int.

Because enum classes don’t have an implicit conversion to integral types, we run into a problem when we try to use their enumerators as array indices. Most obviously, we can `static_cast` the enumerator to an integer.

A better option is to use the helper function that we introduced in enums, which allows us to convert the enumerators of enum classes to integral values using unary `operator+`.

```cpp
#include <type_traits> // for std::underlying_type_t
// Overload the unary + operator to convert StudentNames to the underlying type
constexpr auto operator+(StudentNames a) noexcept {
    return static_cast<std::underlying_type_t<StudentNames>>(a);
}

int main() {
    std::vector<int> testScores(+StudentNames::max_students);
    testScores[+StudentNames::stan] = 76;
    std::cout << +StudentNames::max_students;
}
```

---
### Length and Capcity

Ccapacity** is how many elements the `std::vector` has allocated storage for, and **length** is how many elements are currently being used.

`.resize(x)` (changes length (and capacity, if necessary))
- When we resized the vector, the existing element values are preserved. T
- The new elements are value-initialized (which performs default-initialization for class types, and zero-initialization for other types). 
- Vectors may also be resized to be smaller.

Obviously, the valid indices for the subscript operator (`operator[]`) and `at()` member function is based on the vector’s length, not the capacity.

When a `std::vector` changes the amount of storage it is managing, this process is called **reallocation**. Informally, the reallocation process goes something like this:
- The `std::vector` acquires new memory with capacity for the desired number of elements. These elements are value-initialized.
- The elements in the old memory are copied (or moved, if possible) into the new memory. The old memory is then returned to the system.
- The capacity and length of the `std::vector` are set to the new values.


`std::vector` has a member function called `shrink_to_fit()` that requests that the vector shrink its capacity to match its length. This request is non-binding, meaning the implementation is free to ignore it. Depending on what the implementation thinks is best, an implementation may decide to fulfill the request, may partially fulfill it (e.g. reduce the capacity but not all the way), or may completely ignore the request.


`.reserve(x)` (increases capacity only )
- When reserving more than current length, then reallocation takes place
- When reserving less than current length, then nothing happens.

---
### `push_back()` vs `emplace_back()`

In cases where we are creating a temporary object (of the same type as the vector’s element) for the purpose of pushing it onto the vector, `emplace_back()` can be more efficient:

```cpp

class Foo {
private:
    std::string m_a{};
    int m_b{};
public:
    Foo(std::string_view a, int b) : m_a { a }, m_b { b } {}
    explicit Foo(int b) : m_a {}, m_b { b } {};
};

int main() {
	std::vector<Foo> stack{};
	// When we already have an object, they are similar in efficiency
	Foo f{ "a", 2 };
	stack.push_back(f);    // prefer this one
	stack.emplace_back(f);

	// When we need to create a temp obj to push, emplace_back is more efficient
	stack.push_back({ "a", 2 }); // a temp obj, and then copies it into the vec
	stack.emplace_back("a", 2);  
// forwards the args so the obj can be created directly in the vec (no copy made)

	// push_back won't use explicit constructors, emplace_back will
	stack.push_back({ 2 }); // compile error: Foo(int) is explicit
	stack.emplace_back(2);  // ok
}
```

With `emplace_back()`, we pass the arguments that would be used to create the temporary object, and `emplace_back()` forwards them (using a feature called perfect forwarding) to the vector, where they are used to create and initialize the object inside the vector. This avoids a copy that would have otherwise been made.

`push_back()` won’t use explicit constructors, whereas `emplace_back()` will. This makes `emplace_back` more dangerous, as it is easier to accidentally invoke an explicit constructor to perform some conversion that doesn’t make sense.

Prior to C++20, `emplace_back()` doesn’t work with aggregate initialization.

Prefer `emplace_back()` when creating a new temporary object to add to the container, or when you need to access an explicit constructor. Prefer `push_back()` otherwise.

---
### `std::vector<bool>`

It has some tradeoffs that users should be aware of:
- `std::vector<bool>` has a fairly high amount of overhead (`sizeof(std::vector<bool>)` is 40 bytes on the author’s machine).
- the performance of `std::vector<bool>` is highly dependent upon the implementation. A highly optimized implementation can be significantly faster than alternatives. However, a poorly optimized implementation will be slower.
- `std::vector<bool>` is not a vector (it is not required to be contiguous in memory), nor does it hold `bool` values (it holds a collection of bits), nor does it meet C++’s definition of a container.

Although `std::vector<bool>` behaves like a vector in most cases, it is not fully compatible with the rest of the standard library. Code that works with other element types may not work with `std::vector<bool>`.

For example, the following code works when `T` is any type except `bool`:

```cpp
template<typename T>
void foo( std::vector<T>& v ) {
    T& first = v[0]; // get a reference to the first element
}
```

The optimizing version of `std::vector<bool>` is enabled by default, and there is no way to disable it in favour of a non-optimized version that is actually a container. There have been calls to deprecate `std::vector<bool>`, and work is underway to determine what a replacement compacted vector of `bool` might look like (perhaps as a future `std::dynamic_bitset`).

Best practice:
- Use (constexpr) `std::bitset` when the number of bits you need is known at compile-time, you don’t have more than a moderate number of Boolean values to store (e.g. under 64k).
- Prefer `std::vector<char>` when you need a resizable container of Boolean values and space-savings isn’t a necessity. This type behaves like a normal container.
- Favour a 3rd party implementation of a dynamic bitset (such as `boost::dynamic_bitset`) when you need a dynamic bitset to do bit operations on.

---