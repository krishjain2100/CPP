
The **Scope Resolution Operator** is the double colon `::`.

In C++, names (variables, functions) can exist in many places: inside a class, inside a parent class, inside a namespace, or in the global scope. The `::` operator acts as a path selector to pinpoint exactly which one you mean.

### 1. Resolving Inheritance Ambiguity

Reference from `Inheritance.md` notes regarding the **Diamond Problem**.

If a `Child` inherits from both `Mom` and `Dad`, and both have a variable named `age` (inherited from `Grandma`), the compiler doesn't know which `age` to access. You use `::` to specify the path.

```cpp
int main() {
    Child c;
    // ERROR: c.age is ambiguous.
    // FIX: Use Scope Resolution to specify the path
    c.Mom::age = 80;
    cout << c.Dad::age;
}
```

### 2. Defining Class Functions Outside the Class

Usually, you declare functions inside the class (in `.h`) and define them in a separate file (in `.cpp`). You need `::` to tell the compiler that this code belongs to `MyClass`.


```cpp
class Box {
public:
    void open(); // Declaration only
};

// Definition outside
// "The 'open' function that belongs to 'Box' scope"
void Box::open() { 
    cout << "Opened";
}
```

### 3. Accessing Static Members

As discussed previously, `static` members belong to the class, not the object. You access them using the class name and `::`.


```cpp
class Robot {
public:
    static int population;
};

int main() {
    // You don't need an object (r1.population).
    // You access the class scope directly.
    Robot::population = 5; 
}
```

### 4. Namespaces & Enums

- **Namespaces:** `std::cout` means "The `cout` inside the `std` namespace."
- **Enums:** `Color::Red` means "The `Red` value inside the `Color` enum class."
    
### 5. The "Global Scope" Escape Hatch

If you have a local variable with the same name as a global variable, the local one hides the global one. You can use `::` **without a name** in front of it to say "Go to the very top (Global) scope."

```cpp
int x = 100; // Global x

int main() {
    int x = 10; // Local x
    cout << x;   // Prints 10 (Local)
    cout << ::x; // Prints 100 (Global)
}
```