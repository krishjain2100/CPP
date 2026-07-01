### Type Aliases

```cpp
using Distance = double; // define Distance as an alias for type double
```
 
 When the compiler encounters a type alias name, it will substitute in the aliased type.
**Best practice:** Name your type aliases starting with a capital letter and do not use a suffix (unless you have a specific reason to do otherwise).

Type aliases can also be templated. We cover this in lesson [13.14 -- Class template argument deduction (CTAD) and deduction guides](https://www.learncpp.com/cpp-tutorial/class-template-argument-deduction-ctad-and-deduction-guides/).

An alias does not actually define a new, distinct type (one that is considered separate from other types), it just introduces a new identifier for an existing type. A type alias is completely interchangeable with the aliased type.

This allows us to do things that are syntactically valid but semantically meaningless. For example:

```cpp
int main() {
    using Miles = long; // define Miles as an alias for type long
    using Speed = long; // define Speed as an alias for type long
    Miles distance { 5 }; // distance is actually just a long
    Speed mhz  { 3200 };  // mhz is actually just a long
    // The following is syntactically valid (but semantically meaningless)
    distance = mhz;
}
```

Because the compiler does not prevent these kinds of semantic errors for type aliases, we say that aliases are not **type safe**. In spite of that, they are still useful.

Some languages support the concept of a **strong typedef** (or strong type alias). A strong typedef actually creates a new type that has all the original properties of the original type, but the compiler will throw an error if you try to mix values of the aliased type and the strong typedef. As of C++20, C++ does not directly support strong typedefs (though enum classes, covered in [[Enumerations]], are similar), but there are quite a few 3rd party C++ libraries that implement strong typedef-like behavior.
#### The scope of a type alias

Because scope is a property of an identifier, type alias identifiers follow the same scoping rules as variable identifiers: a type alias defined inside a block has block scope and is usable only within that block, whereas a type alias defined in the global namespace has global scope and is usable to the end of the file. If you need to use one or more type aliases across multiple files, they can be defined in a header file and `#included` into any code files that needs to use the definition.

Note: The compiler replaces all occurrences of type alias and linker has no role here so, there is no linkage associated with type alias

---
### Typedefs

A **typedef** is an older way of creating an alias for a type. 
```cpp
// The following aliases are identical
typedef long Miles;
using Miles = long;
```

The syntax for typedefs can get ugly with more complex types. For example, here is a hard-to-read typedef, along with an equivalent (and slightly easier to read) type alias:

```cpp
typedef int (*FcnType)(double, char); // FcnType hard to find
using FcnType = int(*)(double, char); // FcnType easier to find
```


#### Using type aliases for platform independent coding

On some platforms, an `int` is 2 bytes, and on others, it is 4 bytes. Thus, using `int` to store more than 2 bytes of information can be potentially dangerous when writing platform independent code. In order to make sure each aliased type resolves to a type of the right size, type aliases of this kind are typically used in conjunction with preprocessor directives:

```cpp
#ifdef INT_2_BYTES
using int8_t = char;
using int16_t = int;
using int32_t = long;
#else
using int8_t = char;
using int16_t = short;
using int32_t = int;
#endif
```

On machines where integers are only 2 bytes, `INT_2_BYTES` can be `#defined` (as a compiler/preprocessor setting), and the program will be compiled with the top set of type aliases.

The fixed-width integer types (such as `std::int16_t` and `std::uint32_t`) and the `size_t` type are actually just type aliases to various fundamental types.

This is also why when you print an 8-bit fixed-width integer using `std::cout`, you’re likely to get a character value. For example:

```cpp
std::int8_t x{ 97 }; // int8_t is usually a typedef for signed char
std::cout << x << '\n'; // prints a
```

---
