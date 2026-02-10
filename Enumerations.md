
An **Enum** is a user-defined data type that consists of a set of named integer constants. It makes code more readable by replacing "magic numbers" (0, 1, 2) with meaningful names (RED, GREEN, BLUE).

There are two types of Enums in C++:
1. **Un-scoped Enums** (Old C-Style).
2. **Scoped Enums** (Modern `enum class`).
---
### Un-scoped Enums

This is the original C-style enum.
**Syntax:** `enum Name { A, B, C };`


```cpp
enum Color {
    RED,    // Defaults to 0
    GREEN,  // Defaults to 1
    BLUE = 10, // Manually set to 10
    YELLOW  // Becomes 11
};

int main() {
    Color c = RED;
    
    // PROBLEM 1: Implicit conversion to int
    int x = c; // Valid! (x becomes 0)
    
    // PROBLEM 2: Scope Pollution
    // You cannot declare another enum with 'RED'
    // enum Fruit { APPLE, RED }; // ERROR: 'RED' collision!
}
```

---
### 2. Scoped Enums 

This uses the keyword **`enum class`**. It solves all the problems of old style.

```cpp
enum class Color {
    Red,
    Green,
    Blue
};

enum class Fruit {
    Apple,
    Red // OK! No collision with Color::Red
};

int main() {
    Color c = Color::Red; // Must use Scope Resolution (Color::)
    // int x = c; // ERROR: Cannot implicitly convert to int
    
    // If you REALLY need the number:
    int x = static_cast<int>(c); 
}
```

---

### 3. Specifying the Underlying Type

By default, an enum uses the size of an `int` (usually 4 bytes). If you have a huge list of enums or want to save memory (e.g., sending data over a network), you can change the underlying type.

```cpp
// This enum now only takes up 1 byte (0 to 255)
enum class Status : unsigned char {
    Ok = 0,
    Warning = 1,
    Error = 2
};

int main() {
    cout << sizeof(Status); // Prints 1
}
```
#### Why is `sizeof(Status)` only 1 byte? Shouldn't it be larger?

The confusion comes from thinking an `Enum` works like a `Struct` or an `Array`.
**Explanation:**
1. **It holds ONE value at a time:** When you create a variable `Status s = Status::Warning;`, the computer only allocates memory to store the number `1`. It does not store `0` and `2` alongside it. Since `1` fits inside an `unsigned char` (1 byte), the variable only needs 1 byte of RAM.
2. **The Names are for Humans (Compile-Time only):** You might wonder, "Where are the strings 'Ok', 'Warning', 'Error' stored?" They are **NOT** stored in the variable. The compiler sees `Status::Warning`, replaces it with the number `1`, and throws the text away. The final binary running on the CPU generally doesn't know the word "Warning" exists.