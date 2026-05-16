### 1. Function Pointers 

A function pointer is a raw, primitive pointer that holds the memory address of a block of compiled code.

**The Syntax:** The syntax for function pointers is  ugly. You read it inside-out.

```cpp
void printNumber(int x) {
    std::cout << x;
}

int main() {
    // Declares a pointer named 'funcPtr' that points to a function
    // taking an 'int' and returning 'void'.
    void (*funcPtr)(int) = &printNumber;

    // Call it
    funcPtr(5); 
}
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

### 2. `std::function` 

Because function pointers couldn't handle capturing lambdas, C++11 introduced `std::function` (A wrapper on function pointers)

**The Syntax:** The syntax is vastly cleaner: `std::function<ReturnType(Arguments)>`.


```cpp
#include <functional>
#include <iostream>

void normalFunction(int a) { std::cout << a << "\n"; }

int main() {
    int multiplier = 2;

    // std::function can hold a normal function
    std::function<void(int)> myTask = normalFunction;

    // std::function can ALSO hold a capturing lambda!
    myTask = [multiplier](int a) { 
        std::cout << a * multiplier << "\n"; 
    };

    myTask(5); // Prints 10
}
```

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
