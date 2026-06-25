### 1. Function Pointers 


The syntax for creating a function pointer is ugly:

```cpp
// fcnPtr is a pointer to a function that takes no arguments and returns an integer
int (*fcnPtr)();
```

`fcnPtr` is a ptr to a function that has no parameters and returns an integer. 
`fcnPtr` can point to any function that matches this type.

The parentheses around  * fcnPtr are necessary, as `int* fcnPtr()` would be interpreted as a forward declaration for a function named fcnPtr that takes no parameters and returns a pointer to an integer.

To make a const function pointer, the const goes after the asterisk:

```cpp
int (*const fcnPtr)();
```

Note that the type (parameters and return type) of the function pointer must match the type of the function. Function pointers can be assigned nullptr.
```cpp
int foo() { return 5; }
int goo() { return 6; }

int main() {
    int (*fcnPtr)(){ &foo }; // fcnPtr points to function foo
    fcnPtr = &goo; // fcnPtr now points to function goo
}
```

Unlike fundamental types, C++ _will_ implicitly convert a function into a function pointer if needed (so you don’t need to use the address-of operator (&) to get the function’s address). However, function pointers will not convert to void pointers, or vice-versa (though some compilers like Visual Studio may allow this anyway).

```cpp
int foo();
int (*fcnPtr5)() { foo }; // okay,implicitly converts to a function pointer
void* vPtr { foo };  // not okay, though some compilers may allow
```

There are two ways to call a function using function pointer."
The first is via explicit dereference.
The second way is via implicit dereference:

```cpp
int foo(int x) { return x; }

int main() {
    int (*fcnPtr)(int){ foo }; // Initialize fcnPtr with function foo
    fcnPtr(5); // call function foo(5) through fcnPtr.
}
// Some older compilers do not support the implicit dereference method, but all modern compilers should.
```
---
#### Default arguments don’t work for functions called through function pointers 

When the compiler encounters a normal function call to a function with one or more default arguments, it rewrites the function call to include the default arguments. This process happens at compile-time, and thus can only be applied to functions that can be resolved at compile time.

However, when a function is called through a function pointer, it is resolved at runtime. In this case, there is no rewriting of the function call to include default arguments

```cpp
void print(int x) { std::cout << "print(int)\n"; }
void print(int x, int y = 10) { std::cout << "print(int, int)\n"; }

int main() {
    // print(1); // ambiguous function call
    using vnptr = void(*)(int); 
    // define a type alias for a function pointer to a void(int) function
    vnptr pi { print }; // initialize our function pointer with function print
    pi(1); // call the print(int) function through the function pointer
    // Concise method
    static_cast<void(*)(int)>(print)(1); 
    // call void(int) version of print with argument 1

}
```

---
#### Callbacks

Functions used as arguments to another function are called callback functions.
One common use case is to pass custom sorting functions to sorting algorithms.

Note: If a function parameter is of a function type, it will be converted to a pointer to the function type. This means:
```cpp
void selectionSort(int* array, int size, bool (*comparisonFcn)(int, int))
```
can be equivalently written as:
```cpp
void selectionSort(int* array, int size, bool comparisonFcn(int, int))
```

This only works for function parameters, and so is of somewhat limited use. On a non-function parameter, the latter is interpreted as a forward declaration:

```cpp
bool (*ptr)(int, int); // definition of function pointer ptr
bool fcn(int, int);    // forward declaration of function fcn
```




**The Issue:** Function pointers are incredibly fast because there is zero overhead; the CPU just jumps to a memory address. However, **a function pointer CANNOT store a lambda that captures variables.**

Remember the hidden class the compiler generates for a capturing lambda? A class object is data. A function pointer can only point to pure code.


```cpp
int x = 10;

// SUCCESS: Empty capture list. The compiler can convert this to a raw function pointer.
void (*goodPtr)() = []() { std::cout << "Hello"; }; 

// ERROR: Capturing lambda. This is a class object now, not a raw function.
void (*badPtr)() = [x]() { std::cout << x; }; 
```

---

### 2. `std::function` 


Because function pointers couldn't handle capturing lambdas, C++11 introduced `std::function` (A wrapper on function pointers)

An alternate method of defining and storing function pointers is to use std::function, which is part of the standard library `<functional>` header. 

```cpp
#include <functional>
bool validate(int x, int y, std::function<bool(int, int)> fcn); // std::function method that returns a bool and takes two int parameters
```

`std::function` only allows calling the function via implicit dereference (e.g. `fcnPtr()`), not explicit dereference (e.g. `(*fcnPtr)()`).

When defining a type alias, we must explicitly specify any template arguments. We can’t use CTAD in this case since there is no initializer to deduce the template arguments from. (Huh)

Much like the _auto_ keyword can be used to infer the type of normal variables, the _auto_ keyword can also infer the type of a function pointer.

---
### 3. Performance Cost 

`std::function`comes with a massive, hidden performance cost called **Type Erasure**.

When you assign a capturing lambda to a `std::function`, the `std::function` needs to store that hidden lambda class inside itself.

- If your lambda only captures one small `int`, `std::function` can usually fit it inside its own small buffer (Small Object Optimisation).

- **But if your lambda captures a lot of variables**, the hidden class becomes too large. `std::function` is forced to  call `new` and allocate memory on the **Heap** to store your lambda.

Heap allocations are incredibly slow. Furthermore, calling a `std::function` requires following pointers and virtual tables (indirection), which ruins the CPU's ability to inline the code for speed. (**Hmmmm, did not understand acche se**)

---
### 4. Templates

```cpp
// The Template Way: Blazing fast, zero heap allocation, perfectly inlined.
template <typename Callable>
void executeTask(Callable myLambda) {
    myLambda();
}

int main() {
    int x = 10;
    // We pass the lambda directly. The compiler perfectly deduces 
    // the hidden class type and compiles a custom version of executeTask.
    executeTask( [x]() { std::cout << x; } ); 
}
```

---
