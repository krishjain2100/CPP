
The syntax for creating a function pointer is ugly:

```cpp
// fcnPtr is a pointer to a function that takes no arguments and returns an integer
int (*fcnPtr)();
```

`fcnPtr` is a ptr to a function that has no parameters and returns an integer. 
`fcnPtr` can point to any function that matches this type.

The parentheses around  `*fcnPtr` are necessary, as `int* fcnPtr()` would be interpreted as a forward declaration for a function named `fcnPtr` that takes no parameters and returns a pointer to an integer.

To make a const function pointer, the const goes after the asterisk. If you put the const before the int, then that would indicate the function being pointed to would return a const int.

```cpp
int (*const fcnPtr)();
```

Note that the type (parameters and return type) of the function pointer must match the type of the function, else compiler error will result. Function pointers can be assigned `nullptr`.

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

There are two ways to call a function using function pointer.
- via explicit dereference.
- via implicit dereference:

Some older compilers do not support the implicit dereference method, but all modern compilers should.

```cpp
int foo(int x) { return x; }

int main() {
    int (*fcnPtr)(int){ foo }; // Initialize fcnPtr with function foo
    (*fcnPtr)(5); // explicit
    fcnPtr(5); // implicit.
```

---
### Default arguments don’t work for functions called through function pointers 

When the compiler encounters a normal function call to a function with one or more default arguments, it rewrites the function call to include the default arguments. This process happens at compile-time, and thus can only be applied to functions that can be resolved at compile time.

However, when a function is called through a function pointer, it is resolved at runtime. In this case, there is no rewriting of the function call to include default arguments.

This means that we can use a function pointer to disambiguate a function call that would otherwise be ambiguous due to default arguments. In the following example, we show two ways to do this:

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
### Callback Functions

Functions used as arguments to another function are called **callback functions**.
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

---
### Type Aliases for function pointers

Type aliases can be used to make pointers to functions look more like regular variables:

```cpp
using ValidateFunction = bool(*)(int, int); // takes 2 int ad returns a bool
```

Now instead of doing this:

```cpp
bool validate(int x, int y, bool (*fcnPtr)(int, int)); // ugly
```

You can do this:

```cpp
bool validate(int x, int y, ValidateFunction pfcn) // clean
```
---
### `std::function` 

An alternate method of defining and storing function pointers is to use `std::function`, which is part of the standard library `<functional>` header. It can be initialised with any callable.

```cpp
#include <functional>
bool validate(int x, int y, std::function<bool(int, int)> fcn); 
// std::function method that returns a bool and takes two int parameters
```

`std::function` only allows calling the function via implicit dereference (e.g. `fcnPtr()`), not explicit dereference (e.g. `(*fcnPtr)()`). Also, most importantly, it can be reseated.

```cpp
int foo() { return 5; }
int goo() { return 6; }

int main() {
    int (*fcnPtr)(){ &foo }; // fcnPtr points to function foo
    fcnPtr = &goo; // fcnPtr now points to function goo
	std::function fcnPtr2{ &foo };  
	// can also use CTAD to infer template arguments
}
```

Type Aliases:

```cpp
using ValidateFunctionRaw = bool(*)(int, int); 
using ValidateFunction = std::function<bool(int, int)>; 
```

Ofcourse, when defining a type alias, we must explicitly specify any template arguments. We can’t use CTAD in this case since there is no initializer to deduce the template arguments from.

The Callable you pass does not have to match the `std::function` signature _perfectly_; it just has to be **implicitly convertible**. For example, if your `std::function` expects a `double` and returns a `double`, you can  initialize it with a lambda that expects an `int` and returns an `int`.

The _auto_ keyword can  infer the type of a function pointer, just like you'd expect.

---
### Performance Cost 

Let's see what happens when you assign a lambda to a `std::function` in C++,

When you write a lambda, the compiler generates a unique anonymous class (often called a **closure type**). If your lambda captures variables, those variables become **member variables** of this generated class. The code inside the lambda becomes the `operator()` method of that class. Every single lambda has a **completely unique type**, even if two lambdas have the exact same signature and captures.

Because every lambda has a different, unnameable type, you cannot easily store different lambdas in an array or reassign them to a single variable. `std::function<int(int)>` solves this by acting as a universal wrapper for _any_ callable object that takes an `int` and returns an `int`. But how does it store an object whose type it doesn't know at compile time? It uses a technique called **Type Erasure**.

Suppose you do:
`std::function<int(int)> func = [x](int y) { return x + y; };` 
This is what happens now:

#### Step A: Templated Constructor is Invoked

`std::function` has a templated constructor that accepts any type `T` (in this case, our hidden lambda class). Because it is a template, the constructor _temporarily_ knows the exact type and size of the lambda.

#### Step B: Memory Allocation (and Small Object Optimization)

`std::function` needs to store the lambda instance. It has two ways to do this, depending on the **size** of the lambda (which is dictated by how many variables you captured).

- **Small Object Optimization (SOO):** Most implementations of `std::function` have a small internal byte array (typically 16 to 32 bytes). If your lambda's captured state is small enough to fit in this buffer, `std::function`  constructs the lambda _directly inside_ the `std::function` object. **No heap allocation occurs.**

- **Heap Allocation:** If you captured a lot of variables (e.g., several large objects by value), the lambda will be too big for the internal buffer. In this case, `std::function` calls `new` to allocate memory on the heap, constructs the lambda there, and stores a pointer to it internally.

#### Step C: Erasing the Type

Now that the lambda is stored in memory, `std::function` needs a way to invoke it, copy it, and destroy it later, but it can't store the exact type `T` as a class member (otherwise `std::function` wouldn't be generic).

It achieves this by storing pointers to "manager" or "trampoline" functions (often implemented via a hidden virtual table or raw function pointers).

The templated constructor instantiates these helper functions for the specific lambda type `T`:

1. **Invoker:** A function that casts the stored raw memory back to the lambda type `T` and calls `operator()`.
2. **Destroyer:** A function that casts the memory back to `T` and calls its destructor (and frees the heap memory if it was dynamically allocated).
3. **Copier:** A function that knows how to safely copy `T`.

`std::function` saves pointers to these helper functions. The knowledge of the original lambda type `T` is now erased from the data structure and lives only inside these specialized function pointers.

### 4. What Happens at Invocation

When you finally call `func(5);`, the following occurs:
1. `std::function` looks up its internal **Invoker** function pointer.
2. It passes the raw memory (either the internal small buffer or the heap pointer) and the argument (`5`) to this invoker.
3. The invoker casts the raw memory back into the `__Lambda_Hidden_Name_123` type.
4. It executes the lambda's `operator()`.

### Summary of the Overhead

Assigning a lambda to `std::function` is incredibly powerful, but it comes with a cost compared to using raw lambdas or function pointers:
- **Allocation Cost:** If the lambda captures too much data, it triggers a heap allocation (slow).
- **Size Cost:** A `std::function` object is typically 32-64 bytes (to hold the small buffer and function pointers), whereas a capture-less lambda is 1 byte.
- **Invocation Cost:** Calling a `std::function` requires an indirect call (through a function pointer or vtable), which usually prevents the compiler from inlining the lambda's code.

---
