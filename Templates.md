When you write a template, you are **not writing code**; you are writing a **blueprint**. The compiler doesn't generate any machine code until you actually _use_ the template.

- **Template:** The blueprint (e.g., `vector<T>`).
- **Instantiation:** The specific version the compiler builds for you (e.g., `vector<int>`).

If you use `vector<int>`, `vector<double>`, and `vector<string>`, the compiler effectively copy-pastes your class code **three times**, replacing `T` with the respective types. 
This can lead to **Code Bloat** (larger binary size), but it makes the code incredibly fast (no runtime checks).

### 1. Function Templates
Instead of writing `add(int, int)` and `add(float, float)`, you write one generic function. 

```cpp
// "T" is a placeholder for a type (int, double, string, etc.)
template <typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    cout << add(5, 10);      // Compiler generates: int add(int, int)
    cout << add(5.5, 2.1);   // Compiler generates: double add(double, double)
    // cout << add(5, 2.1);  // ERROR! T cannot be both int and double simultaneously.
}
```

---
### 2. Class Templates

Just like `std::vector<int>`, you can make your own classes generic.

```cpp
template <typename T>
class Box {
private:
    T item;
public:
    Box(T i) : item(i) { }
    T getItem() { return item; }
};

int main() {
    Box<int> intBox(123);
    Box<string> strBox("Hello");
}
```

---
### 3. Template Specialisation 

**Template Specialisation** allows you to say: _"Use the generic blueprint for everything, EXCEPT for this one specific type. For this type, use my custom code."_

```cpp
#include <iostream>
using namespace std;

// 1. The Generic Blueprint
template <typename T>
class Printer {
public:
    void print(T data) {
        cout << "Generic print: " << data << endl;
    }
};

// 2. The Specialization (Specifically for 'char*')
// We use an empty template <> to say "This is a specialized version"
template <>
class Printer<char*> {
public:
    void print(char* data) {
        cout << "STRING SPECIAL PRINT: " << data << endl;
    }
};

int main() {
    Printer<int> intPrinter;
    intPrinter.print(42);         // Outputs: Generic print: 42

    char myString[] = "Hello";
    Printer<char*> strPrinter;
    strPrinter.print(myString);   // Outputs: STRING SPECIAL PRINT: Hello
}
```

---

### 4. Multiple Template Parameters & Default Types

You can have multiple types, and you can even provide default types (just like default function arguments).

```cpp
// T and U can be different types. U defaults to 'int' if not provided.
template <typename T, typename U = int>
class Pair {
public:
    T first;
    U second;

    Pair(T a, U b) : first(a), second(b) {}
    
    void display() {
        cout << "First: " << first << ", Second: " << second << endl;
    }
};

int main() {
    // Specify both types
    Pair<string, double> p1("Price", 99.99); 
    
    // Specify only the first type, the second defaults to 'int'
    Pair<string> p2("Age", 30); 
}
```

---

### 5. Non-Type Template Parameters

A template parameter doesn't have to be a type (like `int` or `string`); it can actually be a **value** (like the number `5`). This is heavily used in standard library containers like `std::array`.

```cpp
// 'T' is a type. 'Size' is an actual integer value determined at compile time.
template <typename T, int Size>
class StaticArray {
private:
    T arr[Size]; // The size of the array is fixed at compile time!
public:
    int getSize() const { return Size; }
};

int main() {
    StaticArray<int, 10> myIntArray; // An obj with an int array of size 10
    StaticArray<string, 5> myStrArray;  // An obj with a string array of size 5
    cout << myIntArray.getSize(); // Outputs: 10
}
```

**Why do this?** Because the `Size` is known at compile-time, it allows for heavy optimisation and allows you to create arrays on the Stack rather than allocating them dynamically on the Heap with `new`.

---

### 6. The "Header File" Rule

Normally, you put the class declaration in the `.h` file and the function definitions in the `.cpp` file. **You cannot do this with templates.** 
Because the compiler needs to see the _entire blueprint_ to generate code when it encounters a template instantiation in `main.cpp`, **both the declaration and the implementation of a template must be in the header file (`.h`).** 
If you put the implementation in a `.cpp` file, you will get Linker Errors (Unresolved External Symbol).

---
### 7. Variadic Templates

Introduced in Modern C++, this allows a template to take **any number** of arguments. 
We use the ellipsis `...` operator (Parameter Pack).

```cpp
// 1. Base Case (Stopping condition)
void print() {
    cout << endl;
}

// 2. Recursive Template
// Takes one argument (first), and a "pack" of others (rest...)
template <typename T, typename... Args>
void print(T first, Args... rest) {
    cout << first << " ";
    print(rest...); // Recursive call with the remaining arguments
}

int main() {
    print(10, "Hello", 5.5, 'A'); 
    // Prints: 10 Hello 5.5 A
}
```

---

### 8. The Guardrails

For a long time, template errors were notoriously hideous. **Concepts** fix this by allowing you to put constraints on what `T` can be.

```cpp
template <typename T>
// This Constraint ensures T is an integer type (int, long, short)
requires std::integral<T> 
T add(T a, T b) {
    return a + b;
}

int main() {
    add(10, 20);      // OK
    // add("A", "B"); // ERROR: Clean message saying "string is not integral"
}
```

---
### 9. Template Meta-programming (TMP)


**The "Old School" Way (Pre-C++11)**

Before modern C++, if you wanted to calculate a Factorial at compile time, you couldn't just write a `for` loop. You had to abuse the template system to create a recursive structure.

**The Logic:** Instead of a function calling itself, you have a **Class Template instantiating a new version of itself**.

```cpp
// 1. Recursive Case
template<int N>
class Factorial {
public:
    // To calculate value, we multiply N * Factorial<N-1>::value
    // This forces the compiler to build Factorial<4>, then Factorial<3>...
    enum { value = N * Factorial<N-1>::value };
};

// 2. Base Case (Stopping Condition)
template<>
class Factorial<1> {
public:
    enum { value = 1 };
};

// Usage
int x = Factorial<5>::value; // Evaluates to 120 at compile time.
```

- **How it works:** `Factorial<5>` needs `Factorial<4>`, which needs `Factorial<3>`... all the way down to `Factorial<1>`.
    
- **The Downside:** It is incredibly verbose, hard to read, and creates huge compiler error messages.

- **Solution:**  `constexpr`

