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

|**Syntax**|**Read as...**|**Meaning**|
|---|---|---|
|`const int* p`|"Pointer to an `int` that is `const`"|You can change **where** `p` points, but you cannot change the **data** at that address.|
|`int* const p`|"Const pointer to an `int`"|You **cannot** change where `p` points (it's stuck to one address), but you can change the **data**.|
|`const int* const p`|"Const pointer to a const int"|You can change **nothing**. The address is locked, and the data is read-only.|


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

### 4. Const Reference (`int& const`) - The "Silly" One

**Question:** "What if I want a reference that cannot be reseated to point to a different object?"_
**Answer:** All references are **already** like that. Once you create a reference `int& r = x;`, it is **forever** stuck to `x`. You cannot make it point to `y`. Therefore, writing `int& const` is redundant and technically ignored by modern compilers (though sometimes treated as an error).

### 5. Const Member Functions 

Putting `const` at the end of a function signature promises that the function will **not modify any member variables**.

If you do not write `const` at the end of the function name, the compiler assumes the function **WILL** change the object, even if the body is empty!

If you have a `const Object`, you can **only** call `const` functions on it.


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

Sometimes, you need to modify a variable inside a `const` function (e.g., for debugging, logging, or thread locks).

The `mutable` keyword allows a specific member variable to be changed **even if the object is const**.

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

### 9. `constexpr`

**Why it exists:**
- Replace template meta-programming (see - below) hacks
- Improve readability and error messages
- Enable zero-overhead abstractions

This allows the compiler to optimise code by calculating values before the program even runs.

- If you feed them **Constants** (like `5` or `const int`), they run at **Compile Time**.
- If you feed them **Variables** (like `x`), they run at **Runtime** just like a normal function.

```cpp
// 1. Compile-time function
constexpr int square(int x) {
    return x * x;
}

int main() {
    // Calculated at compile time. 
    // The compiled binary effectively just says "arr[25]".
    int arr[square(5)]; 
    // C-Style Arrays are strict:
    // The size of a raw array (`int arr[SIZE]`) must be known before the program ever runs.
}
```

#### Creating Types with `constexpr`

Writing code that writes code. Specifically, using numbers to change **Types**.

**1. Compile-Time Dimensions:** The `dim(int x)` function works like `factorial`. It returns a square `x*x`. The slide shows:

```cpp
using Matrix = std::array<int, dim(N)>; 
```

If `N=4`, `dim(N)` is 16. The compiler creates a `std::array<int, 16>`. This is much faster than `std::vector` because it lives on the Stack, not the Heap.

**2. Branching Types (`std::conditional`):** This is a very cool optimiwation technique. You can choose different variable types based on a number size.
- **Scenario:** You have a number `N`.
- If `N < 8`, it fits in a small byte (`uint8_t`).
- If `N >= 8`, you need a big integer (`uint64_t`).


```cpp
template <size_t N>
struct Storage {
    // std::conditional is a compile-time "if statement" for TYPES.
    // Format: conditional< Boolean, TypeIfTrue, TypeIfFalse >
    using type = std::conditional_t<
        (N < 8),       // Condition
        uint8_t,       // Result if True
        uint64_t       // Result if False
    >;
};
```

- **Why use this?** If you are writing a high-frequency trading platform or a game engine, you don't want to waste 64 bits of memory to store the number "5". This automates the memory optimisation for you.

---