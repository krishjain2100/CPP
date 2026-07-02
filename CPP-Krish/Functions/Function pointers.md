
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

`std::function`comes with a massive cost when you assign a lambda to a `std::function`. 
It needs to store that lambda object inside itself. It will copy the object if it is an lvalue (expensive) or move it, if rvalue. 

- If the lambda only captures couple of `int`, `std::function` can usually fit it inside its own small buffer (Small Object Optimisation).
- But if the lambda captures a lot of variables, `std::function` is forced to  call `new` and allocate memory on the **Heap** to store the objec

It also does **Type Erasure**. It does not remember the type of the Callable that was used to initialize it. It just remembers the signature (arguments, return type) of the function that will be called. Also it stores tables and stuff to figure out what to call at runtime. So inlining is not possible because we don't know what it will pointing to at runtime.


Using Templates:

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
