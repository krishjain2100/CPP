### Const Variables

A `const` variable cannot be changed after initialisation. It **must** be initialised when declared.
The keyword can be written before or after the type, but the usual convention is to write before the type (like shown below).
```cpp
const int maxScore = 100;
// maxScore = 200; // ERROR: Cannot assign to variable 'maxScore'
```

---
### Const Member Functions 

Putting `const` at the end of a function signature promises that the function will **not modify any member variables**. It mainly helps in maintaining correctness but also helps in minor compiler optimisations (some getters called multiple times can be called once only because we know it didn't get changed). If you do not write `const` at the end of the function name, the compiler assumes the function **WILL** change the object, even if the body is empty. If you have a `const Object`, you can **only** call `const` functions on it.

```cpp
class Wallet {
    int money;
public:
    void addMoney(int m) { money += m; } // 1. Mutator

    // 2. Const function (Accessor)
    int checkBalance() const {
        // money = 0; // ERROR: Cannot modify members in a const function
        return money; 
    }
};

void printWallet(const Wallet& w) {
    // w.addMoney(10);     // ERROR: 'w' is const!
    cout << w.checkBalance(); // OK: checkBalance is const-safe
}
```

---
### `mutable` Keyword 

Sometimes, you need to modify a variable inside a `const` function (e.g., for debugging, logging, or thread locks (you can then lock the mutex inside a const function)).  The `mutable` keyword allows a specific member variable to be changed **even if the object is const** ( const class objects can only call const functions).

```cpp
class Database {
    // "mutable" means: "I can change even in a const function"
    mutable int debugAccessCount = 0; 
    string data;
public:
    string getData() const {
        debugAccessCount++; // OK because of 'mutable'
        return data;
    }
};
```

---
### 4. Const Function Parameters (Pass by Const Reference)

```cpp
void processImage(const Image& img) {
    // efficient (no copy)
    // safe (cannot modify img)
}
```

---
### Const Return Types

For returning by value, the `const` qualifier on a return type is simply ignored (your compiler may generate a warning), because the temporary copy being const is useless. Though `const` is useful for returning references to internal data safely.

Returning a const value can also impede certain kinds of compiler optimizations (involving move semantics), which can result in better performance.

```cpp
class Student {
    string name;
public:
    // Allows read access to 'name' without making a copy.
    // The caller cannot use this reference to change the name.
    const string& getName() const {
        return name;
    }
};

int main() {
    Student s;
    const string& n = s.getName(); // OK
    // s.getName() = "Bob";        // ERROR: Cannot assign to const reference
}
```

---
### `volatile` Variables

In C++23, there are only two type qualifiers (`const` and `volatile`).
`volatile` tells the C++ compiler: _"This variable's value can change at any given microsecond from completely outside the program. Do not trust it, and do not optimise it."

**Why we use it (The Optimisation Trap):** When you compile with optimisations (`-O2` or `-O3`), the compiler tries to make your code extremely fast by caching variables inside the CPU registers.

```cpp
int button_state = 0; // Hardware changes this to 1 when pressed

void waitForButton() {
    while (button_state == 0) {
        // Wait
    }
}
```

- **Without `volatile`:** The compiler sees the loop and thinks, _"Nothing inside this loop changes `button_state`. I will load it into the CPU register once, and just check the register forever."_ Even if the physical hardware changes the RAM to `1`, your program is stuck in an infinite loop looking at the cached `0`.

- **With `volatile`:** By writing `volatile int button_state = 0;`, you legally forbid the compiler from caching it. You force the CPU to physically fetch the bits from main RAM on every single iteration of the loop.

**Where we encounter it:** You will almost never see `volatile` in standard desktop or web backend C++. It is strictly used in low-level systems programming:

1. **Embedded Systems & Hardware:** Reading memory-mapped hardware registers (e.g., Arduino pins, engine sensors, graphics card memory).
2. **Interrupt Service Routines (ISRs):** When hardware interrupts the CPU to pause your program and change a state (like a keyboard press).
3. **Signal Handlers:** Catching OS-level signals like `CTRL+C` (`SIGINT`) to gracefully shut down a program.

Note: **The Multithreading Trap**:  Many engineers mistakenly believe `volatile` is used to share variables between multiple threads safely. **It is not.**

- `volatile` stops the _compiler_ from caching.
- It does **not** stop the _CPU_ from reordering instructions out of order.
- It does **not** prevent Data Races (two threads writing at the exact same millisecond).
- For multithreading, always use `std::atomic<int>` or `std::mutex`.

---
### `volatile` Member Functions


---