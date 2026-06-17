
---

**Global variable initialization**
Unlike local variables, which are uninitialized by default, variables with static duration are zero-initialized by default.

Non-constant global variables can be optionally initialized:
```cpp
int g_x;       // no explicit initializer (zero-initialized by default)
int g_y {};    // value initialized (resulting in zero-initialization)
int g_z { 1 }; // list initialized with specific value
```

That syntax is called **Brace Initialization**. It was introduced in C++11.
An empty set of braces `{}` commands the compiler to perform **Value Initialization**. For a fundamental type like an `int`, value initialization guarantees the memory is set exactly to `0`.

### 1. Uniformity

Before C++11, you had to use different syntax depending on what you were building:

- `int x = 0;` (For basic types)
- `int arr[] = {1, 2, 3};` (For arrays)
- `std::vector<int> v;` (For objects)
- `Player p(10);` (For classes)

The C++ committee introduced `{}` so you could initialize **absolutely everything** using the exact same syntax:

```cpp
int x {0};
int arr[] {1, 2, 3};
std::vector<int> v {1, 2, 3};
Player p {10};
```

### 2. Preventing Narrowing Conversions

 Brace initialization actively protects your memory from accidental truncation.
```cpp
// OLD C-STYLE (Silent Data Loss)
int a = 4.5; // Compiler says: "Fine." It truncates the .5. 'a' is now 4.
// MODERN C++ BRACE INITIALIZATION (Fatal Error)
int b { 4.5 }; // Compiler says: "FATAL ERROR. You are losing data."
```

---

GCC and Clang support the flag `-Wshadow` that will generate warnings if a variable is shadowed. 