
A **Functor/Function Objects** is just a fancy name for a standard C++ `class` or `struct` that overrides the **`operator()`** (the parentheses operator). This trick allows an _instance of an object_ to be invoked exactly like a function.

```cpp
class Accumulator {
private:
    int current_sum = 0; // Internal State!

public:
    // Overriding the function call operator
    int operator()(int value) {
        current_sum += value;
        return current_sum;
    }
};

int main() {
    Accumulator acc; // Create an instance of the class (the object)
    
    // We treat the object like a function!
    std::cout << acc(5)  << "\n"; // Prints 5
    std::cout << acc(10) << "\n"; // Prints 15 (It remembered the 5!)
}
```
