The three most common fake C++ features that compilers let you get away with, which `CMAKE_CXX_EXTENSIONS OFF` will strictly forbid.

### 1. Variable Length Arrays (VLAs) in C++

In strict C++, the size of a standard array _must_ be known at compile time (it must be a constant). But GCC  allows you to size an array using a runtime variable:


```cpp
// ALLOWED IN GNU C++ (Extension)
// FATAL ERROR IN STRICT C++ (MSVC will crash here)
int size;
std::cin >> size;
int arr[size]; 
```

If you turn extensions `OFF`, CMake forces you to write standard, cross-platform C++. You must use dynamic memory allocation instead:

```cpp
// STRICT ISO C++ STANDARD
int size;
std::cin >> size;
std::vector<int> arr(size); // The correct, cross-platform way
```

### 2. Case Ranges in Switch Statements

If you are writing a parser and want to check if a character is a letter, GNU gives you a shortcut using `...`:


```cpp
char c = 'b';
switch (c) {
    // ALLOWED IN GNU C++ (Extension)
    case 'a' ... 'z': 
        std::cout << "It is lowercase";
        break;
}
```

If you turn extensions `OFF`, the compiler will throw an error and force you to write it the standard way


```cpp
// STRICT ISO C++ STANDARD
switch (c) {
    case 'a': case 'b': case 'c': /* ... all the way to z */:
        std::cout << "It is lowercase";
        break;
}
```

### 3. Statement Expressions

GNU allows you to put a multi-line block of code inside parentheses `({ ... })` and have it evaluate to a single return value. This is heavily used in the Linux Kernel source code.


```cpp
// ALLOWED IN GNU C++ (Extension)
int result = ({ 
    int a = 5; 
    int b = 10; 
    a + b; // The last statement becomes the return value
});
```

If you turn extensions `OFF`, you are forced to use **Lambda Functions**.

```cpp
// STRICT ISO C++ STANDARD
int result = []() {
    int a = 5;
    int b = 10;
    return a + b;
}(); // Notice the () at the end to immediately invoke it
```