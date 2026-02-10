RTTI stands for **Run-Time Type Information**.

It is a mechanism that allows the program to determine the type of an object **while the program is running**, rather than just at compile time.

### Use-case

Normally, C++ is a **statically typed** language. The compiler knows exactly what every variable is (an `int`, a `Car*`, a `Shape*`) before you run the code.

However, when using **Polymorphism**, you often lose this specific information.
- You might have a `std::vector<Animal*>` that contains `Dog*`, `Cat*`, and `Bird*`.
- The compiler only sees `Animal*`.
- But sometimes, at runtime, you really need to know: _"Is this specific animal actually a Dog?"_

### Working

RTTI only works for classes that have **at least one `virtual` function**.
- When the compiler creates the **V-Table** (for virtual functions), it secretly adds an extra slot pointing to a `type_info` structure.
- This structure holds the actual name and details of the class.

### The Two Main Tools of RTTI

#### 1. `dynamic_cast` (Safe Down-casting)

It tries to convert a Base pointer to a Child pointer.
- If the object **IS** that Child type, it returns the pointer.
- If the object **IS NOT** that Child type, it returns `nullptr`.

```cpp
class Animal { virtual void func() {} }; // Must be polymorphic
class Dog : public Animal { public: void bark() {} };
class Cat : public Animal {};

void petTheAnimal(Animal* ptr) {
    // RTTI CHECK: "Is this pointer actually pointing to a Dog?"
    Dog* d = dynamic_cast<Dog*>(ptr);
    
    if (d != nullptr) {
        cout << "It's a Dog!";
        d->bark();
    } else {
        cout << "Not a Dog.";
    }
}
```

#### 2. `typeid` (Type Identification)

This allows you to compare types directly.

```cpp
#include <typeinfo> // Required header

void checkType(Animal* ptr) {
    if (typeid(*ptr) == typeid(Dog)) {
        cout << "This is exactly a Dog.";
    }
    // You can also get the name (implementation dependent)
    cout << typeid(*ptr).name(); // Might print "class Dog" or "3Dog"
}
```

### The Cost of RTTI

- **Memory:** It adds a small amount of data to every class (the `type_info` struct).
- **Speed:** `dynamic_cast` is slow. It has to traverse the inheritance tree at runtime to check validity.
- **Note:** You can actually turn RTTI **off** in many compilers (like `-fno-rtti` in GCC/Clang) to save memory, but then `dynamic_cast` will crash or fail to compile.