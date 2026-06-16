### 1. Const Variables

A `const` variable cannot be changed after initialisation. It **must** be initialised when declared.

```cpp
const int maxScore = 100;
// maxScore = 200; // ERROR: Cannot assign to variable 'maxScore'
```

---
### 2. Const and Pointers
(Avoid mixing const and pointers, you'll rarely encounter it ~ Akshat Jha)

The position of `const` relative to the asterisk `*` changes the meaning entirely.

**Rule of Thumb:** Read from **Right to Left**.

| **Syntax**           | **Read as...**                        | **Meaning**                                                                                          |
| -------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `const int * p`      | "Pointer to an `int` that is `const`" | You can change **where** `p` points, but you cannot change the **data** at that address.             |
| `int * const p`      | "Const pointer to an `int`"           | You **cannot** change where `p` points (it's stuck to one address), but you can change the **data**. |
| `const int* const p` | "Const pointer to a const int"        | You can change **nothing**. The address is locked, and the data is read-only.                        |


```cpp
int x = 10;
int y = 20;

// 1. Pointer to Const (Read-only Data)
const int* p1 = &x; 
// *p1 = 15; // ERROR: Data is const
p1 = &y;     // OK: Pointer itself can move

// 2. Const Pointer (Fixed Address)
int* const p2 = &x;
*p2 = 15;    // OK: Data is mutable
// p2 = &y;  // ERROR: Pointer is const (stuck to x)

// 3. Const Pointer to Const (Locked Down)
const int* const p3 = &x;
// *p3 = 15; // ERROR
// p3 = &y;  // ERROR
```

---
### 3. Reference to Const (`const int&`)

This is a **"Read-Only View"** of an object.
- You can read the value.
- You **cannot** change the value through this reference.
- The original object _might_ be changeable by someone else, but _you_ can't touch it.

```cpp
int x = 10;
const int& ref = x; // "ref" is a read-only window into "x"

// ref = 20; // ERROR: ref is read-only
x = 20;      // OK: x is not const, so we can change it directly.
             // Now "ref" sees 20.
```

Also you can't have non-const references to const objects, as then the const objects could be modified through the reference, which would be wrong.

---
### 4. Const Reference (`int& const`) - The "Silly" One

**Question:** "What if I want a reference that cannot be reseated to point to a different object?"_
**Answer:** All references are **already** like that. Once you create a reference `int& r = x;`, it is **forever** stuck to `x`. You cannot make it point to `y`. Therefore, writing `int& const` is redundant and technically ignored by modern compilers (though sometimes treated as an error).

---
### 5. Const Member Functions 

Putting `const` at the end of a function signature promises that the function will **not modify any member variables**. It helps in compiler optimisation. If you do not write `const` at the end of the function name, the compiler assumes the function **WILL** change the object, even if the body is empty! If you have a `const Object`, you can **only** call `const` functions on it.


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
### 6. `mutable` Keyword 

Sometimes, you need to modify a variable inside a `const` function (e.g., for debugging, logging, or thread locks).  The `mutable` keyword allows a specific member variable to be changed **even if the object is const**.

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
### 7. Const Function Parameters (Pass by Const Reference)

- **Pass by Value:** `void f(BigObj x)` -> **Slow** (Makes a copy).
- **Pass by Reference:** `void f(BigObj& x)` -> **Fast** (No copy), but risky (function might change `x`).
- **Pass by Const Reference:** `void f(const BigObj& x)` -> **Fast & Safe**.

```cpp
void processImage(const Image& img) {
    // efficient (no copy)
    // safe (cannot modify img)
}
```

---
### 8. Const Return Types

Returning `const` is rare for basic types (`const int f()`), but useful for returning references to internal data safely.

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
#### 10. `volatile` 

`volatile` is a qualifier that tells the C++ compiler: _"This variable's value can change at any given microsecond from completely outside the program. Do not trust it, and do not optimise it."

**Why we use it (The Optimisation Trap):** When you compile with optimizations (`-O2` or `-O3`), the compiler tries to make your code extremely fast by caching variables inside the CPU registers.

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
