`noexcept` is a specifier you put on a function signature.

```cpp
void func() noexcept {
    // I promise I will not throw
}
```

**The Promise (Runtime Behavior)** This function will never throw an exception. If an exception _does_ try to escape a `noexcept` function, the program **does not catch it**. It immediately calls `std::terminate()` and kills the entire program because you broke your promise.
    
**The Optimisation (Compile Time)** Because the compiler knows the function won't throw:
- It doesn't generate "stack unwinding" code (the heavy machinery needed to handle exceptions).
- It enables functions like `std::vector::resize` to use `std::move_if_noexcept` to choose the faster path.