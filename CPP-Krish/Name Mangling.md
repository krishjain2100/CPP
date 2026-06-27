When the compiler compiles a function, it performs **name mangling**, which means the compiled name of the function is altered (“mangled”) based on various criteria, such as the number and type of parameters, so that the linker has unique names to work with.

For example, a function with prototype `int fcn()` might compile to mangled name `__fcn_v`, whereas `int fcn(int)` might compile to mangled name `__fcn_i`. So while in the source code, the two overloaded functions share the name `fcn()`, in compiled code, the mangled names are unique (`__fcn_v` vs `__fcn_i`).

There is no standardization on how names should be mangled, so different compilers will produce different mangled names.

In **C**, you can only have one function with a specific name. If you write `void print(int x)`, the compiler literally names it `print` in the final compiled binary. The Linker (the tool that stitches your program together) just looks for `print`.

In **C++**, we have **Function Overloading**. You can write this:

```cpp
void print(int x);
void print(double x);
void print(std::string x);
```

If the C++ compiler just exported all three of these as `print` to the compiled binary, the Linker would crash. 

To fix this, the C++ compiler takes your clean, human-readable names and **"mangles"** them. It squishes the function's name together with encoded information about its namespaces, classes, and parameter types to create a 100% unique string. The Linker never sees your C++ code; it only sees these mangled, unique IDs.

 Let's look at the **Itanium ABI** Name Mangling, which is the standard used by GCC and Clang.

```cpp
void calculate(int x);
```

The compiler mangles it into: **`_Z9calculatei`**

Here is how to decode that:
- **`_Z`**: This is the standard prefix. It tells the system, "Hey, this is a mangled C++ name."
- **`9`**: This is the number of characters in the function's name.
- **`calculate`**: The actual name of the function.
- **`i`**: The code for the parameter type (`i` stands for `int`).

A More Complex Example:
```cpp
class Math {
public:
    void calculate(double x, char y);
};
```

This mangles to: **`_ZN4Math9calculatedcE`**
- **`_Z`**: C++ Mangled name prefix.
- **`N`**: Stands for "Nested" (indicating it belongs to a class or namespace).
- **`4Math`**: A 4-letter name: `Math`.
- **`9calculate`**: A 9-letter name: `calculate`.
- **`d`**: First parameter is a `double`.
- **`c`**: Second parameter is a `char`.
- **`E`**: End of the nested name block.


Because C++ changes the names of everything, it creates a massive problem if you want to use a library written in C. If you include a C library that has a function `void connect();`, your C++ compiler will assume it is a C++ function and look for the mangled name `_Z7connectv` in the library. But because the library was compiled in C, the function is just named `connect`. The Linker will throw an `undefined reference` error. To stop the C++ compiler from mangling a name, you wrap it in `extern "C"`.

```cpp
extern "C" {
    void connect(); // The compiler leaves this alone. It stays exactly "connect"
}
```
---
### Summary

- **What is it?** A string-encoding process performed by the compiler.
- **Why do we need it?** To support function overloading, namespaces, and classes by ensuring every single function and object has a universally unique name in the final binary.
- **Can you reverse it?** Yes. Reversing it is called **demangling**. Command-line tools like `c++filt` or built-in functions like `abi::__cxa_demangle` can translate `_ZN4Math9calculatedcE` back into `Math::calculate(double, char)` so humans can read error logs.

---