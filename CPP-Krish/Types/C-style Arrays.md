
C-style arrays were inherited from the C language, and are built-in to  C++ (unlike the rest of the array types, which are standard library container classes). This means we don’t need to `#include` a header file to use them.

Because they are the only array type natively supported by the language, the standard library array container types (e.g. `std::array` and `std::vector`) are typically implemented using a C-style array.

```cpp
int arr1[5]; // Members default init (int elements are left uninitialized)
int arr2[5] {}; // Members value init (int elements are zero uninitialized) 
```

Inside the square brackets, we can optionally provide the length of the array, which is an integral value of type `std::size_t` that tells the compiler how many elements are in the array. The length of a C-style array must be at least 1. The compiler will error if the array length is zero, negative, or a non-integral value. But C-style arrays dynamically allocated on the heap are allowed to have length 0.

Just like `std::array`, when declaring a C-style array, the length of the array must be a constant expression (of type `std::size_t`, though this typically doesn’t matter).

Some compilers may allow creation of arrays with non-constexpr lengths, for compatibility with a C99 feature called variable-length arrays (VLAs) **(read about this).**
Variable-length arrays are not valid C++, and should not be used in C++ programs. If your compiler allows these arrays, you probably forgot to disable compiler extensions.

C-style arrays can also be indexed using the subscript operator (`operator[]`). 
Unlike the standard library container classes (which use unsigned indices of type `std::size_t` only), the index of a C-style array can be a value of any integral type (signed or unsigned) or an un-scoped enumeration.

C-style arrays are also aggregates, which means they can be initialized using aggregate initialization.

```cpp
int a[4] { 1, 2, 3, 4, 5 }; // compile error: too many initializers
int b[4] { 1, 2 };          // arr[2] and arr[3] are value initialized
```

One downside of using a C-style array is that the element’s type must be explicitly specified. CTAD doesn’t work because C-style arrays aren’t class templates. And using `auto` to try to deduce the element type of an array from the list of initializers doesn’t work either:

```cpp
auto squares[5] { 1, 4, 9, 16, 25 }; // compile error: can't use type deduction 
```


When we initialize a C-style array with an initializer list, we can omit the length (in the array definition) and let the compiler deduce the length of the array from the number of initializers.

```cpp
const int prime1[5] { 2, 3, 5, 7, 11 }; 
const int prime2[] { 2, 3, 5, 7, 11 };  
// prime2 deduced by compiler to have length 5
```

Applied to a C-style array, `sizeof()` returns the number of bytes used by the entire array:

```cpp
const int prime[] { 2, 3, 5, 7, 11 };
std::cout << sizeof(prime); // prints 20 (assuming 4 byte ints)
```

Note that there is no overhead here. An C-style array object contains its elements and nothing more.


In C++17, we can use `std::size()`, which returns the array length as an unsigned integral value (of type `std::size_t`). 
In C++20, we can also use `std::ssize()`, which returns the array length as a signed integral value (of a large signed integral type, probably `std::ptrdiff_t`).

Prior to C++17, you can use this function instead:

```cpp
template <typename T, std::size_t N>
constexpr std::size_t length(const T(&)[N]) noexcept {
// unnamed parameter with &, also brackets becuase of precendence issues
	return N;
}

int main() {
	int array[]{ 1, 1, 2, 3, 5, 8, 13, 21 };
	std::cout << "The array has: " << length(array) << " elements\n";
}
```

In much older codebases, you may see the length of a C-style array determined by dividing the size of the entire array by the size of an array element:

```cpp
int array[8] {};
std::cout <<  sizeof(array) / sizeof(array[0]) << " elements\n";
```

However, you will see below, that this formula can fail quite easily (when passed a decayed array), leaving the program unexpectedly broken. C++17’s `std::size()` and the `length()` function template shown above will both cause compilation errors in this case, so they are safe to use.

Perhaps surprisingly, C++ arrays don’t support assignment:

```cpp
int arr[] { 1, 2, 3 }; // okay: initialization is fine
arr[0] = 4;            // assignment to individual elements is fine
arr = { 5, 6, 7 };     // compile error: array assignment not valid
```

Technically, this doesn’t work because assignment requires the left-operand to be a modifiable lvalue, and **C-style arrays are not considered to be modifiable lvalues**.

You can assign new values to a C-style array element-by-element, or use `std::copy` to copy an existing C-style array:

```cpp
#include <algorithm> // for std::copy
int arr[] { 1, 2, 3 };
int src[] { 5, 6, 7 };
// Copy src into arr
std::copy(std::begin(src), std::end(src), std::begin(arr));
```

---
### Const and constexpr C-style arrays

C-style arrays can also be `const` or `constexpr`. Just like other const variables, const arrays must be initialized, and the value of the elements cannot be changed afterward.

---
### C-style array decay

Consider the following similar program, which uses a C-style int array:

```cpp
void printElementZero(int arr[1000]) {
    std::cout << arr[0];
}

int main() {
    int x[1000] { 5 } 
     // define an array with 1000 elements, x[0] is initialized to 5
    printElementZero(x);
}
```

This program compiles and prints the expected value (`5`) to the console. 
But mechanically it works a bit different than you might expect. 

C doesn’t have references, so using pass by reference to avoid making a copy of function arguments wasn’t an option. Also we want to be able to write a single function that can accept array arguments of different lengths. C has no syntax to specify “any length” arrays, nor does it support templates, nor can arrays of one length be converted to arrays of another length (presumably because doing so would involve making an expensive copy).

The designers of the C language came up with a clever solution (inherited by C++ for compatibility reasons) that solves for both of these issues:

```cpp
void printElementZero(int arr[1000]) { // doesn't make a copy
    std::cout << arr[0]; 
}

int main() {
    int x[7] { 5 };     // define an array with 7 elements
    printElementZero(x); // somehow works!
}
```

Somehow, the above example passes a 7 element array to a function expecting a 1000 element array, without any copies being made. In this lesson, we’ll explore how this works.

---
#### Array to pointer conversions (array decay)

In most cases, when a C-style array is used in an expression, the array will be implicitly converted into a pointer to the element type, initialized with the address of the first element (with index 0). This is called **array decay**.

You can see this in the following program:

```cpp
int arr[5]{ 9, 7, 5, 3, 1 }; // our array has elements of type int
auto ptr{ arr }; 
std::cout << (typeid(ptr) == typeid(int*)) << '\n';  // 1
std::cout << std::boolalpha << (&arr[0] == ptr) << '\n'; // 1
```

Similarly, a const array (e.g. `const int arr[5]`) decays into a pointer-to-const (`const int*`).

In C++, there are a few common cases where an C-style array doesn’t decay:
1. When used as an argument to `sizeof()` or `typeid()`.
2. When taking the address of the array using `operator&`.
3. When passed as a member of a class type.
4. When passed by reference.

Because C-style arrays decay into a pointer in most cases, it’s a common fallacy to believe arrays _are_ pointers. This is not the case. An array object is a sequence of elements, whereas a pointer object just holds an address. Also the type information of an array and a decayed array is different. In the example above, the array `arr` has type `int[5]`, whereas the decayed array has type `int*`.

Because a C-style arrays decays to a pointer when evaluated, when a C-style array is subscripted, the subscript is actually operating on the decayed array pointer:

```cpp
const int arr[] { 9, 7, 5, 3, 1 };
std::cout << arr[2]; // subscript decayed array to get element 2, prints 5
```

We can also use `operator[]` directly on a pointer. If that pointer is holding the address of the first element, the result will be identical:

```cpp
const int arr[] { 9, 7, 5, 3, 1 };
const int* ptr{ arr };  // arr decays into a pointer
std::cout << ptr[2];    // subscript ptr to get element 2, prints 5
```

When passing a C-style array as an argument, although it looks like we’re passing the C-style array by value, we’re actually passing it by address. This is how making a copy of the C-style array argument is avoided. Two different arrays of the same element type but different lengths are distinct types, incompatible with each other. However, they will both decay into the same pointer type (e.g. `int*`). Dropping the length information from the type allows us to pass arrays of different lengths without a type mismatch.

One problem with declaring the function parameter as `int* arr` is that it’s not obvious that `arr` is supposed to be a pointer to an array of values rather than a pointer to a single integer. For this reason, when passing a C-style array, its preferable to use the alternate declaration form `int arr[]`. Note that no length information is required between the square brackets (since it is not used anyway). If a length is provided, it will be ignored.

---
### Downsides of decay

The loss of array length information makes it easy for several types of mistakes to be made.
First, `sizeof()` will return different values for arrays and decayed arrays:

```cpp
void printArraySize(int arr[]){
    std::cout << sizeof(arr) << '\n'; // prints 4 (assuming 32-bit addresses)
}

int main() {
    int arr[]{ 3, 2, 1 };
    std::cout << sizeof(arr) << '\n'; // prints 12 (assuming 4 byte ints)
    printArraySize(arr);
}
```

We mentioned that `sizeof(arr)/sizeof(*arr)` is dangerous because if `arr` has decayed, `sizeof(arr)` will return the size of a pointer rather than the size of the array, producing the wrong array length as a result, likely causing the program to malfunction. Fortunately, C++17’s  `std::size()` (and C++20’s `std::ssize()`) will fail to compile if passed a pointer value.

Second, without length information, it is difficult to sanity check the length of the array.

```cpp
void printElement2(int arr[]) {
    // How do we ensure that arr has at least three elements?
    std::cout << arr[2] << '\n';
}
```

Third, when traversing the array, how do we know when we’ve reached the end.

---
### Solution to length problem

First, we can pass in both the array and the array length as separate arguments:
However, this still has issues:

- Caller has to ensure they are consistent
- There are potential sign conversion issues.
- We’d want to use `static_assert` for compile-time validation of the array length of constexpr arrays, but there’s no easy way to do this (as function parameters can’t be constexpr).
- This method only works if we’re making an explicit function call. If the function call is implicit (e.g. we’re calling an operator with the array as an operand), then we can't pass in length

Second, if there is an element value that is not semantically valid (e.g. a test score of `-1`), we can  mark the end of the array using an element of that value. This works even with implicit function calls (like `operator<<` overloading) as we don't need an extra parameter.

---
### Modern Usage

In C++, arrays can be passed by reference, in which case the array argument won’t decay when passed to a function (but the reference to the array will still decay when evaluated). However, it’s easy to forget to apply this consistently, and one missed reference will lead to decaying arguments. Also, array reference parameters must have a fixed length, meaning the function can only handle arrays of one particular length. If we want a function that can handle arrays of different lengths, then we also have to use function templates. But if you’re going to do both of those things to “fix” C-style arrays, then you might as well just use `std::array`.

In modern C++, C-style arrays are typically used in two cases:
1. To store constexpr global (or constexpr static local) program data. Because such arrays can be accessed directly from anywhere in the program, we do not need to pass the array, which avoids decay-related issues. Preferred because they don't have sign issues.
2. As parameters to functions or classes that want to handle non-constexpr C-style string arguments directly (rather than requiring a conversion to `std::string_view`). There are two possible reasons for this: 
	- Converting a non-constexpr C-style string to a `std::string_view` requires traversing the C-style string to determine its length. If the length isn’t needed (e.g. because the function is going to traverse the string anyway) then avoiding the conversion may be useful.
	- If the function (or class) calls other functions that expect C-style strings, converting to a `std::string_view` just to convert back may be suboptimal (unless you have other reasons for wanting a `std::string_view`).

---
### Pointer arithmetic and subscripting

**Pointer arithmetic** is a feature that allows us to apply certain integer arithmetic operators (addition, subtraction, increment, or decrement) to a pointer to produce a new memory address.

Given some pointer `ptr`, `ptr + 1` returns the address of the _next object_ in memory (based on the type being pointed to). So if `ptr` is an `int*`, and an `int` is 4 bytes, `ptr + 1` will return the memory address that is 4 bytes after `ptr`, and `ptr + 2` will return the memory address that is 8 bytes after `ptr`.

According to the C++ standard, pointer arithmetic is only defined behavior when the pointer and the result are within the same array (or one-past-the-end). However, modern C++ implementations generally do not enforce this, and will typically not disallow you from using pointer arithmetic outside of arrays.

```cpp
const int arr[] { 9, 7, 5, 3, 1 };
const int* ptr{ arr }; // a normal pointer holding the address of element 0
std::cout << ptr[2];   // subscript ptr to get element 2, prints 5
```

`ptr[n]` is a concise syntax equivalent to `*((ptr) + (n)).

**LOL** because the compiler converts `ptr[n]` into `*((ptr) + (n))` when subscripting a pointer, this means we can also subscript a pointer as `n[ptr]`. The compiler converts this into `*((n) + (ptr))`, which is behaviourally identical to `*((ptr) + (n))`.


We mentioned that (unlike the standard library container classes) the index of a C-style array can be either an unsigned integer or a signed integer. ,  so it’s actually possible to index a C-style array with a negative subscript.

```cpp
const int arr[] { 9, 8, 7, 6, 5 };
const int* ptr { &arr[3] };
std::cout << *ptr << ptr[0] << '\n'; // prints 66
std::cout << *(ptr-1) << ptr[-1] << '\n'; // prints 77
```

Iterating using pointer arithmetic:
```cpp
constexpr int arr[]{ 9, 7, 5, 3, 1 };
const int* begin{ arr };
const int* end{ arr + std::size(arr) }; 
for (; begin != end; ++begin) 
	std::cout << *begin << ' ';
```

For a pointer that is pointing to a C-style array element, pointer arithmetic is valid so long as the resulting address is the address of a valid array element, or one-past the last element. If pointer arithmetic results in an address beyond these bounds, it is undefined behavior (even if the result is not dereferenced). It is just the way it is **(I don't know why)**

Internally, range-based for loop are  implemented exactly like we did using `begin` and `end`.
```cpp
constexpr int arr[]{ 9, 7, 5, 3, 1 };
for (auto e : arr)  // iterate from `begin` up to (but excluding) `end`
	std::cout << e << ' ';
	// dereference our loop variable to get the current element
```

---
### Multidimensional C-style Arrays

We noted that the elements of an array can be of any object type. This means the element type of an array can be another array.

```cpp
int a[3][5]; // a 3-element array of 5-element arrays of int
```

Memory is linear (1-dimensional), so multidimensional arrays are actually stored as a sequential list of elements. There are two possible ways for the following array to be stored in memory:

```cpp
// col 0   col 1   col 2   col 3   col 4
// [0][0]  [0][1]  [0][2]  [0][3]  [0][4]  row 0
// [1][0]  [1][1]  [1][2]  [1][3]  [1][4]  row 1
// [2][0]  [2][1]  [2][2]  [2][3]  [2][4]  row 2
```

C++ uses **row-major order**, where elements are sequentially placed in memory row-by-row, ordered from left to right, top to bottom. Some other languages (like Fortran) use **column-major order**, elements are sequentially placed in memory column-by-column, from top to bottom, left to right:.

In C++, when initializing an array, elements are initialized in row-major order. And when traversing an array, it is most efficient to access elements in the order they are laid out in memory.

To initialize a two-dimensional array, it is easiest to use nested braces, with each set of numbers representing a row:

```cpp
int array[3][5] {
  { 1, 2, 3, 4, 5 },     // row 0
  { 6, 7, 8, 9, 10 },    // row 1
  { 11, 12, 13, 14, 15 } // row 2
};
```

Although some compilers will let you omit the inner braces, we recommend you include them anyway for readability purposes. When using inner braces, missing initializers will be value-initialized:

```cpp
int array[3][5] {
  { 1, 2 },          // row 0 = 1, 2, 0, 0, 0
  { 6, 7, 8 },       // row 1 = 6, 7, 8, 0, 0
  { 11, 12, 13, 14 } // row 2 = 11, 12, 13, 14, 0
};
```

An initialized multidimensional array can omit (only) the leftmost length specification:

```cpp
int array[][5] {
  { 1, 2, 3, 4, 5 },
  { 6, 7, 8, 9, 10 },
  { 11, 12, 13, 14, 15 }
};
```

In such cases, the compiler can do the math to figure out what the leftmost length is from the number of initializers. Omitting non-leftmost dimensions is not allowed.

Multidimensional arrays can still be initialized to 0 as follows:

```cpp
int array[3][5] {};
```

---
