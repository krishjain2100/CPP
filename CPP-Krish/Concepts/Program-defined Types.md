Program-defined types must be defined and given a name before they can be used. 
The definition for a program-defined type is called a **type definition**.
Type definitions end with a semicolon. Example:

```cpp
struct Fraction {
	int numerator {};
	int denominator {};
};
```


Every code file that uses a program-defined type needs to see the full type definition before it is used. A forward declaration is not sufficient. This is required so that the compiler knows how much memory to allocate for objects of that type. Therefore type definitions are partially exempt from the one-definition rule (ODR): a given type is allowed to be defined in multiple code files. This is the reason why we can import `<iostream>` in multiple files as we are importing all of the  input/output type definitions into multiple files.

There are two caveats that are worth knowing about:
1. You can still only have one type definition per code file
2. All of the type definitions for a given type must be identical, otherwise undefined behaviour will result.

A program-defined type used in multiple code files should be defined in a header file with the same name as the program-defined type and then `#included` into each code file as needed.

---
### Nomenclature: user-defined types vs program-defined types

The term “user-defined type” is not defined in the C++ language standard. In casual conversation, the term tends to mean “a type defined within your own programs” (such as the Fraction type example above).

The C++ language standard uses the term “user-defined type” in a non-conventional manner. In the language standard, a “user-defined type” is any class type or enumerated type that is defined by you, the standard library, or the implementation (e.g. types defined by the compiler to support language extensions). Perhaps counter-intuitively, this means `std::string` (a class type defined in the standard library) is considered to be a user-defined type!

To provide additional differentiation, the C++20 language standard helpfully defines the term “program-defined type” to mean class types and enumerated types that are not defined as part of the standard library, implementation, or core language. In other words, “program-defined types” only include class types and enum types that are defined by us (or a third-party library).

| Type            | Meaning                                                                                                                                                                    | Examples                             |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Fundamental     | A basic type built into the core C++ language                                                                                                                              | int, std::nullptr_t                  |
| Compound        | A type defined in terms of other types                                                                                                                                     | int&, double*, std::string, Fraction |
| User-defined    | A class type or enumerated type  <br>(Includes those defined in the standard library or implementation)  <br>(In casual use, typically used to mean program-defined types) | std::string, Fraction                |
| Program-defined | A class type or enumerated type  <br>(Excludes those defined in standard library or implementation)                                                                        | Fraction                             |

---