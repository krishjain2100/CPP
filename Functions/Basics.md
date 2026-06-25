### Nested functions

A function whose definition is placed inside another function is a **nested function**.  Nested functions are not supported in C++.

---
### `main()`

When the program is executed, the operating system makes a function call to `main()`. Execution then jumps to the top of `main()`. The statements in `main()` are executed sequentially. Finally, `main()` returns an integer value (usually `0`), and your program terminates.

In C++, there are two special requirements for `main()`:
- `main()` is required to return an `int`.
- Explicit function calls to `main()` are disallowed.

Key insights:
- C does allow `main()` to be called explicitly, so some C++ compilers will allow this for compatibility reasons.
- It is a common misconception that `main` is always the first function that executes. Global variables are initialized prior to the execution of `main`. If the initializer for such a variable invokes a function, then that function will execute prior to `main`. 

#### Status codes

The return value from `main()` is sometimes called a **status code**. The status code is used to signal whether your program was successful or not.
By convention, a status code of `0` means the program ran normally (meaning the program executed and behaved as expected).
A non-zero status code is often used to indicate some kind of failure (and while this works fine on most operating systems, strictly speaking, it’s not guaranteed to be portable).

**Note:** The C++ standard only defines the meaning of 3 status codes: `0`, `EXIT_SUCCESS`, and `EXIT_FAILURE`. `0` and `EXIT_SUCCESS` both mean the program executed successfully. `EXIT_FAILURE` means the program did not execute successfully.

`EXIT_SUCCESS` and `EXIT_FAILURE` are preprocessor macros defined in the`<cstdlib>` header

If you want to maximiwe portability, you should only use `0` or `EXIT_SUCCESS` to indicate a successful termination, or `EXIT_FAILURE` to indicate an unsuccessful termination.


**Note**: The only exception to the rule that a value-returning function must return a value via a return statement is for function `main()`. The function `main()` will implicitly return the value `0` if no return statement is provided. 

---
### Unnamed Parameters

In a function definition, the name of a function parameter is optional. Therefore, in cases where a function parameter needs to exist but is not used in the body of the function, you can simply omit the name. A parameter without a name is called an **unnamed parameter**:

```cpp
void doSomething(int) {
	// ok: unnamed parameter will not generate warning
}
```

You’re probably wondering why we’d write a function that has a parameter whose value isn’t used. This happens most often in cases similar to the following:

1. Let’s say we have a function with a single parameter. Later, the function is updated in some way, and the value of the parameter is no longer needed. If the now-unused function parameter were simply removed, then every existing call to the function would break (because the function call would be supplying more arguments than the function could accept). This would require us to find every call to the function and remove the unneeded argument. This might be a lot of work (and require a lot of retesting). It also might not even be possible (in cases where we did not control all of the code calling the function). So instead, we might leave the parameter as it is, and just have it do nothing.
2. Operators `++` and `--` have prefix and postfix variants (e.g. `++foo` vs `foo++`). An unreferenced function parameter is used to differentiate whether an overload of such an operator is for the prefix or postfix case. We cover this in lesson [21.8 -- Overloading the increment and decrement operators](https://www.learncpp.com/cpp-tutorial/overloading-the-increment-and-decrement-operators/).(didnt understand now)
3. When we need to determine something from the type (rather than the value) of a type template parameter (did not understand).

---

Functions have their own function type . Much like variables, functions live at an assigned address in memory (making them lvalues).

```cpp
// code for foo starts at memory address 0x002717f0
int foo() { return 5; }
int main() {
    cout << foo << '\n';  
    // we meant to call foo(), but instead we're printing foo itself!
}
```

When a function is referred to by name (without parenthesis), C++ converts the function into a function pointer (holding the address of the function). Then `operator<<` tries to print the function pointer, which it fails at because `operator<<` does not know how to print function pointers. The standard says that in this case, `foo` should be converted to a `bool` (which `operator<<` does know how to print). And since the function pointer for `foo` is a non-null pointer, it should always evaluate to Boolean `true`. Thus, this should print: 1

----

