### Three sets of types:
The first two are built-in to the language itself (and do not require the inclusion of a `.h` to use):

- The **fundamental data types** provide the most the basic and essential data types.
- The **compound data types** provide more complex data types and allow for the creation of custom (user-defined) types. 
- The third (and largest) set of types is provided by the C++ standard library.

---
### Fundamental Data Types

The C++ language comes with many predefined data types and are called the **fundamental data types**. Example:

| Types                                                                              | Category            | Meaning                                          | Example |
| ---------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------ | :-----: |
| float  <br>double  <br>long double                                                 | Floating Point      | a number with a fractional part                  | 3.14159 |
| bool                                                                               | Integral (Boolean)  | true or false                                    |  true   |
| char  <br>wchar_t  <br>char8_t (C++20)  <br>char16_t (C++11)  <br>char32_t (C++11) | Integral(Character) | a single character of text                       |   ‘c’   |
| short int  <br>int  <br>long int  <br>long long int (C++11)                        | Integral (Integer)  | positive and negative whole numbers, including 0 |   64    |
| std::nullptr_t (C++11)                                                             | Null Pointer        | a null pointer                                   | nullptr |
| void                                                                               | Void                | no type                                          |   n/a   |

- A notable omission from the table of fundamental types above is a data type to handle **strings**. This is because in modern C++, strings are part of the standard library.
- The **standard integer types** are `short`, `int`, `long`, `long long` (including their signed and unsigned variants).
- The **integral types** are `bool`, the various char types, and the standard integer types. In the context of C++, "integral" is used to mean “like an integer”.  These types are considered to be integral types because these types store their values as integer values.

---
### Void

Void means no type.
- Void is our first example of an incomplete type. 
- An **incomplete type** is a type that has been declared but not yet defined.
- The compiler knows about the existence of such types, but does not have enough information to determine how much memory to allocate for objects of that type. `void` is intentionally incomplete since it represents the lack of a type, and thus cannot be defined.
- Incomplete types can not be instantiated

```cpp
void value; // won't work, variables can't be defined with incomplete type void
```

Most commonly, _void_ is used to indicate that a function does not return a value:
In C, void is used as a way to indicate that a function does not take any parameters: eg int getValue(void){}, obviously no need to put the void in C++ codes
The void keyword has a use in Void Pointers.

---
### Fundamental data type sizes

When we talk about the size of a type, we really mean the size of an instantiated object of that type. Perhaps surprisingly, the C++ standard does not define the exact size (in bits) of any of the fundamental types.

Instead, the standard says the following:
- An object must occupy at least 1 byte (so that each object has a distinct memory address).
- A byte must be at least 8 bits.
- The integral types `char`, `short`, `int`, `long`, and `long long` have a minimum size of 8, 16, 16, 32, and 64 bits respectively.
- `char` and `char8_t` are exactly 1 byte (at least 8 bits).

In this tutorial series, we will present a simplified view, by making some reasonable assumptions that are generally true for modern architectures:

- A byte is 8 bits.
- Memory is byte addressable (we can access every byte of memory independently).
- Floating point support is IEEE-754 compliant.
- We are on a 32-bit or 64-bit architecture.

Given the above assumptions, we can reasonably state the following:

| Category       | Type           | Minimum Size     | Typical Size       |
| -------------- | -------------- | ---------------- | ------------------ |
| Boolean        | bool           | 1 byte           | 1 byte             |
| Character      | char           | 1 byte (exactly) | 1 byte             |
|                | wchar_t        | 1 byte           | 2 or 4 bytes       |
|                | char8_t        | 1 byte           | 1 byte             |
|                | char16_t       | 2 bytes          | 2 bytes            |
|                | char32_t       | 4 bytes          | 4 bytes            |
| Integral       | short          | 2 bytes          | 2 bytes            |
|                | int            | 2 bytes          | 4 bytes            |
|                | long           | 4 bytes          | 4 or 8 bytes       |
|                | long long      | 8 bytes          | 8 bytes            |
| Floating point | float          | 4 bytes          | 4 bytes            |
|                | double         | 8 bytes          | 8 bytes            |
|                | long double    | 8 bytes          | 8, 12, or 16 bytes |
| Pointer        | std::nullptr_t | 4 bytes          | 4 or 8 bytes       |
**Tip**: For maximum portability, you shouldn’t assume that objects are larger than the specified minimum size.

#### The sizeof operator
- The **sizeof operator** is a unary operator that takes either a type or a variable, and returns the size of an object of that type (in bytes).
- Trying to use `sizeof` on an incomplete type (such as `void`) will result in a compilation error.
- `sizeof` does not include dynamically allocated memory used by an object. 

You might assume that types that use less memory would be faster than types that use more memory. This is not always true. CPUs are often optimised to process data of a certain size (e.g. 32 bits), and types that match that size may be processed quicker. On such a machine, a 32-bit int could be faster than a 16-bit short or an 8-bit char.

---
### Integer types

C++ has _4_ primary fundamental integer types available for use:

| Type          | Minimum Size | Note                                      |
| ------------- | ------------ | ----------------------------------------- |
| short int     | 16 bits      |                                           |
| int           | 16 bits      | Typically 32 bits on modern architectures |
| long int      | 32 bits      |                                           |
| long long int | 64 bits      |                                           |
Prefer the shorthand types that do not use the `int` suffix or `signed` prefix.
Example: `short `instead of `short int`(no difference, just looks good)

**Overflow**: 
- Signed: Undefined behaviour occurs(may vary system to system)
- Unsigned: Oddly, the C++ standard explicitly says “a computation involving unsigned operands can never overflow”.  The number wraps around. But we will call this overflow only

During a operation, if unsigned and signed both used, typically signed gets converted to unsigned before operation. Example : 

```cpp
unsigned int u{ 2 };
signed int s{ 3 };
cout << u - s << '\n'; // 2 - 3 = 4294967295
```

There are still a few cases in C++ where it’s okay / necessary to use unsigned numbers.
1. They  are preferred when dealing with bit manipulation. They are also useful when well-defined wrap-around behavior is required (useful in some algorithms like encryption and random number generation).
2. Use of unsigned numbers is still unavoidable in some cases, mainly those having to do with array indexing. 
3. Also note that if you’re developing for an embedded system (e.g. an Arduino) or some other processor/memory limited context, use of unsigned numbers is more common and accepted (and in some cases, unavoidable) for performance reasons.

**Why isn’t the size of the integer types fixed?**

When computers were slow and performance was of the utmost concern, C opted to intentionally leave the size of an integer open so that the compiler implementers could pick a size for `int` that performs best on the target computer architecture. That way, programmers could just use `int` without having to worry about whether they could be using something more performant. By modern standards, the lack of consistent ranges for the various integral types sucks (especially in a language designed to be portable).

---
### Fixed-width integers

To address the above issues, C++11 provides an alternate set of integer types that are guaranteed to be the same size on any architecture called **fixed-width integers**:
std::int(8, 16, 32, 64)_t, std::uint(8, 16, 32, 64)_t

The fixed-width integers actually don’t define new types , they are just aliases for existing integral types with the desired size. For each fixed-width type, the implementation (the compiler and standard library) gets to determine which existing type is aliased. eg if `int` is 32-bits, `std::int32_t` will be an alias for `int`.

In most cases, `std::int8_t` is an alias for `signed char` because it is the only available 8-bit signed integral type.  However, in rare cases, if a platform has an implementation-specific 8-bit signed integral type, the implementation may decide to make `std::int8_t` an alias for that type instead.

Other fixed-width downsides
- First, the fixed-width integers are not guaranteed to be defined on all architectures. They only exist on systems where there are fundamental integral types that match their widths and following a certain binary representation. however this is unlikely to be a problem.
- It may be slower than a wider type on some architectures. For example, you might decide to use `std::int32_t`, but your CPU might actually be faster at processing 64-bit integers. but you still might use it due to memory constraints.

Fast and least integral types
- The fast types (`std::int_fast#_t` and` std::uint_fast#_t`) provide the fastest signed/unsigned integer type with a width of at least # bits (where # = 8, 16, 32, or 64).
- The least types (`std::int_least#_t` and `std::uint_least#_t`) provide the smallest signed/unsigned integer type with a width of at least # bits (where # = 8, 16, 32, or 64).
- These might not be portable. Avoid the fast and least integral types because they may exhibit different behaviours on architectures where they resolve to different sizes.

---
### `std::size_t`

`std::size_t` is an alias for an implementation-defined unsigned integral type. It is used within the standard library to represent the byte-size or length of objects. It is actually a typedef.

It is defined in a number of headers, but it is best to use ```<cstddef>``` (contains least no. of other defined identifiers)
 
 Using `sizeof` does not require a header (even though it returns a value whose type is `std::size_t`, remember `std::size_t `is nothing just a alias for one of the integer types therefore no header required until you specifically use the word).
 
 `std::size_t` also varies in size. guaranteed to be unsigned and at least 16 bits, but on most systems will be equivalent to the address-width of the application.

The size of `std::size_t`(maximum value storable in it) imposes a strict mathematical upper limit on an object’s size. In practice, the largest creatable object may be smaller than this amount (perhaps significantly so). 

The C++20 standard says: “Constructing a type such that the number of bytes in its object representation exceeds the maximum value representable in the type `std::size_t `is ill-formed.”

Some compilers limit the largest creatable object to half the maximum value of `std::size_t`
On a **32-bit system**, `std::size_t` maxes out at 4 Gigabytes. You can hit this limit easily today.
On a **64-bit system**, `std::size_t` maxes out at **16 Exabytes** (16 Billion Gigabytes), which is obviously more memory than any single computer has.
 
Even dynamically allocated objects are bounded by this limit as when you ask the operating system for dynamic memory on the heap, you use either C++'s `new` keyword or C's `malloc()` function.
- The official signature for C's allocation function is: `void* malloc(size_t size);`
- When you write C++ code like `int* myArray = new int[5000];`, the compiler calculates the total bytes needed and passes that number to the allocator using a `std::size_t` integer.

Because the very functions that hand out dynamic memory require you to specify the amount using `std::size_t`, it is mathematically impossible to request a single, contiguous block of memory larger than the maximum value of `std::size_t`. 

But obviously, you could create a class storing pointers to dynamically allocated objects(the pointers are small in size only) and have a class which stores more memory than the limit imposed

---
###  Bool

When we print Boolean values, `std::cout` prints `0` for `false`, and `1` for `true`:
Use `std::cout << std::boolalpha`to print `true` or `false`
You can use `std::cout << std::noboolalpha` to turn it back off.

```cpp
std::cout << true << '\n'; // 1
std::cout << false << '\n'; // 0
std::cout << std::boolalpha; // print bools as true or false
std::cout << true << '\n'; // true
std::cout << false << '\n'; // false
```

It default initialises to false

By default, `std::cin` only accepts numeric input for Boolean variables: `0` is `false`, and `1` is `true`. Any other numeric value will be interpreted as `true`, and will cause `std::cin` to enter failure mode. Any non-numeric value will be interpreted as `false` and will cause `std::cin` to enter failure mode.

To allow `std::cin` to accept the words `false` and `true` as inputs, you must first use to `std::cin >> std::boolalpha;` However, when `std::boolalpha` is enabled for input, numeric values will no longer be accepted (they evaluate to `false` and cause `std::cin` to enter failure mode), also it is case sensitive so only `true` works (True or TRUE won't work).

---
### Char

The integer stored by a `char` variable are interpreted as an `ASCII character`
You can initialize chars with their ASCII values as well, `char ch1{ 97 };`

Char is defined by C++ to always be 1 byte in size. By default, a char may be signed or unsigned (though it’s usually signed). If you’re using chars to hold ASCII characters, you don’t need to specify a sign and 0-127 can be held in both signed and unsigned.

If you’re using a char to hold small integers (something you should not do unless you’re explicitly optimising for space), you should always specify (using prefix) whether it is signed or unsigned. A signed char can hold a number between -128 and 127. An unsigned char can hold a number between 0 and 255.
#### Escape sequences
An escape sequence starts with a ‘\’ (backslash) character, and then a following letter or number.
Here’s a table of all of the escape sequences:

| Name            | Symbol     | Meaning                                                                                |
| --------------- | ---------- | -------------------------------------------------------------------------------------- |
| Alert           | \a         | Makes an alert, such as a beep                                                         |
| Backspace       | \b         | Moves the cursor back one space                                                        |
| Formfeed        | \f         | Moves the cursor to next logical page                                                  |
| Newline         | \n         | Moves cursor to next line                                                              |
| Carriage return | \r         | Moves cursor to beginning of line                                                      |
| Horizontal tab  | \t         | Prints a horizontal tab                                                                |
| Vertical tab    | \v         | Prints a vertical tab                                                                  |
| Single quote    | \’         | Prints a single quote                                                                  |
| Double quote    | \”         | Prints a double quote                                                                  |
| Backslash       | \\         | Prints a backslash.                                                                    |
| Question mark   | \?         | Prints a question mark.  <br>No longer relevant. You can use question marks unescaped. |
| Octal number    | \(number)  | Translates into char represented by octal                                              |
| Hex number      | \x(number) | Translates into char represented by hex number                                         |

a, b, f, r, v- do not do anything useful now, they were useful before when actual mechanical typewriters hooked to the computer were used

For backwards compatibility reasons, many C++ compilers support **multi-character literals**, which are char literals that contain multiple characters (e.g. `'56'`). If supported, these have an implementation-defined value (meaning it varies depending on the compiler). Because they are not part of the C++ standard, and their value is not strictly defined, multi-character literals should be avoided.

Note: While inputting `std::cin` will let you enter multiple characters. Only the first input character is extracted into the variable. The rest is left in the input buffer that `std::cin` uses.

---
### `wchar_t`, `char8_t`, `char16_t`, and `char32_t`

Other character encoding standards exist to map integers (of varying sizes) to characters in other languages. The most well-known mapping outside of ASCII is the Unicode standard, which maps over 144,000 integers to characters in many different languages. Because Unicode contains so many code points, a single Unicode code point needs 32-bits to represent a character (called UTF-32). However, Unicode characters can also be encoded using multiple 16-bit or 8-bit characters (called UTF-16 and UTF-8 respectively).

- `char16_t` and `char32_t` were added to C++11 to provide explicit support for 16-bit and 32-bit Unicode characters. These char types have the same size as `std::uint_least16_t` and `std::uint_least32_t` respectively (but are distinct types). 
- `char8_t` was added in C++20 to provide support for 8-bit Unicode (UTF-8). It is a distinct type that uses the same representation (in memory) as `unsigned char`.
- `wchar_t` (wide-character , 16 bits on Windows, 32 bits on Linux/macOS (GCC/Clang)) should be avoided in almost all cases (except when interfacing with the Windows API), as its size is implementation-defined.

---
### Floating Point Numbers

Floating point data types are always signed.
There are 3 floating point types in C++ (typical sizes mentioned)
- `float`: 4 bytes
- `double`: 8 bytes
- `long double`: 8, 12, or 16 bytes

Floating-point types are conventionally implemented using one of the floating-point formats defined in the IEEE 754 standard. As a result, `float` is almost always 4 bytes, and `double` is almost always 8 bytes.
`long double` is a strange type. On different platforms, its size can vary between 8 and 16 bytes, and it may or may not use an IEEE 754 compliant format. We recommend avoiding `long double`.

 Because floating point numbers tend to be inexact, comparing floating point numbers is generally problematic.

```cpp
double d{0.1};
cout << d << '\n'; // 0.1
cout << std::setprecision(17); // 0.10000000000000001
cout << d << '\n';
```

#### NaN and Inf

IEEE 754 compatible formats additionally support some special values:
- `Inf` represents infinity. `Inf` is signed, and can be positive (`+Inf`) or negative (`-Inf`). Example:  `cout << 5.0/0.0;`, will output `Inf`, answer will be random if you do `5/0` as `int `is not IEEE754 compatible.
- `NaN` stands for “Not a Number”. Example: `0.0/0.0`.
- There are separate representations for positive zero (+0.0) and negative zere (-0.0) in IEEE 754.

#### Outputting floating point values

`std::cout` has a default precision of 6.
We can override this default precision by using an `output manipulator` function named `std::setprecision()`. 
**Output manipulators** alter how data is output, and are defined in the `<iomanip>` header
Output manipulators (and input manipulators) are sticky, meaning if you set them, they will remain set. The one exception is `std::setw`. Some IO operations reset `std::setw`.
Use `std::cout << std::fixed` to avoid auto conversions to scientific notation.

#### Floating point range

| Format                                  | Range                             | Precision                              |
| --------------------------------------- | --------------------------------- | -------------------------------------- |
| IEEE 754 single-precision (4 bytes)     | ±1.18e-38 to ±3.4e38 and 0.0      | 6-9 significant digits, typically 7    |
| IEEE 754 double-precision (8 bytes)     | ±2.23e-308 to ±1.80e308 and 0.0   | 15-18 significant digits, typically 16 |
| x87 extended-precision (80 bits)        | ±3.36e-4932 to ±1.18e4932 and 0.0 | 18-21 significant digits               |
| IEEE 754 quadruple-precision (16 bytes) | ±3.36e-4932 to ±1.18e4932 and 0.0 | 33-36 significant digits               |

---
### Compound Data Types

Compound data type  are defined in terms of other existing data types.

Every data type is either a fundamental type or a compound type. The C++ language standard explicitly defines which category each type falls into.

C++ supports the following compound types:

- Functions
- C-style Arrays
- Pointer types:
    - Pointer to object
    - Pointer to function
- Pointer to member types:
    - Pointer to data member
    - Pointer to member function
- Reference types:
    - L-value references
    - R-value references
- Enumerated types:
    - Un-scoped enumerations
    - Scoped enumerations
- Class types:
    - Structs
    - Classes
    - Unions

---