**Run-time type information (RTTI)** is a feature of C++ that exposes information about an object’s data type at runtime. This capability is leveraged by dynamic_cast. 
### Use-case

Normally, C++ is a **statically typed** language. The compiler knows exactly what every variable is (an `int`, a `Car*`, a `Shape*`) before you run the code.

However, when using **Polymorphism**, you often lose this specific information.
- You might have a `std::vector<Animal*>` that contains `Dog*`, `Cat*`, and `Bird*`.
- The compiler only sees `Animal*`.
- But sometimes, at runtime, you really need to know: _"Is this specific animal actually a Dog?"_

### Working

RTTI only works for classes that have **at least one `virtual` function**.
- When the compiler creates the **V-Table** for a class (which has atleast one virtual function), it adds an extra slot pointing to a `type_info` structure right above the function pointers in the V-Table (usually at offset `-1`).
- This `type_info` struct contains the mangled name of the class (e.g., `"3Dog"`)([[Name Mangling]]) and pointers to its Base classes' `type_info` structs. This creates a graph in your program's memory.

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

`dynamic_cast` fetches the `type_info` via vptr. If it's a match with that of `Dog` then it returns the pointer.  If `ptr` actually points to a `Poodle` (which inherits from `Dog`), the exact match fails. The RTTI system then looks at the `Poodle`'s `type_info` struct, finds the pointer to its base class (`Dog`), and checks that. 

So it walks up the inheritance tree. If it finds `Dog` along the way, it knows the cast is safe and returns the pointer. If it reaches the top of the tree without finding `Dog`, it returns `nullptr`.

You can upcast using `dynamic_cast()` but it is wasteful since `static_cast()` is safe, and some optimisers may even convert your `dynamic_cast()` call  to `static_cast()` if you try to upcast.

##### Cross-Casting

Because `dynamic_cast` relies on a graph search rather than a straight line, it can do something `static_cast` is incapable of doing: **Cross-Casting**. If you use Multiple Inheritance, you can cast sideways _across_ the memory block. Here is exactly how it is handled under the hood:
Consider `WirelessMouse` inherits form both `USB` and `Bluetooth`

**The Memory Layout of a `WirelessMouse` object :** 
- **Byte 0:** `USB`'s vptr (Virtual Pointer)
- **Byte 8:** `USB`'s Member Data
- **Byte 16:** `Bluetooth`'s vptr _(Notice this is physically shifted down!)_
- **Byte 24:** `Bluetooth`'s Member Data
- **Byte 32:** `WirelessMouse`'s Member Data

If you have a `Bluetooth*`, it **must** point to Byte 16. If it points to Byte 0, and you try to call a Bluetooth function, the CPU will read the `USB` vptr instead, resulting in a total program crash. Therefore, converting a `USB*` to a `Bluetooth*` requires doing **pointer arithmetic**, physically adding `+16` to the memory address.

 **How `dynamic_cast` achieves this**

1. It looks at the `USB` vptr and goes to the V-Table.
2. The V-Table's RTTI says: _"This isn't just a USB; the complete object is actually a WirelessMouse."_
3. The RTTI graph holds a map of offsets: _"In a WirelessMouse, the Bluetooth sub-object is located exactly 16 bytes away from the USB sub-object."_
4. `dynamic_cast` takes your pointer's memory address (e.g., `0x1000`), adds `16` to it, and returns `0x1010` as a `Bluetooth*`.

```cpp
class USB { virtual void dummy() {} };
class Bluetooth { virtual void dummy() {} };

// Multiple Inheritance
class WirelessMouse : public USB, public Bluetooth {};

int main() {
    // 1. Allocate object in RAM (Assume it starts at Memory Address 0x1000)
    WirelessMouse* mouse = new WirelessMouse(); 
    
    // 2. Upcast to USB
    // The USB sub-object is at the very top. 
    // usb_ptr holds exactly 0x1000.
    USB* usb_ptr = mouse;

    // 3. CROSS-CAST: From USB directly sideways to Bluetooth!
    // static_cast<Bluetooth*>(usb_ptr) would fail to compile
    // dynamic_cast using RTTI search, realizes this USB is part of a Mouse, 
    // calculates that Bluetooth lives +16 bytes down, and shifts the pointer.
    Bluetooth* bt_ptr = dynamic_cast<Bluetooth*>(usb_ptr); 
    // bt_ptr now safely holds 0x1010!
}
```

Note that there are several cases where downcasting using dynamic_cast will not work:
1. With protected or private inheritance.
2. For classes that do not declare or inherit any virtual functions (and thus don’t have a virtual table).
3. In certain cases involving virtual base classes (see [this page](https://msdn.microsoft.com/en-us/library/cby9kycs.aspx) for an example of some of these cases, and how to resolve them).

#### 2. `typeid` (Type Identification)

This allows you to compare types directly.
It compares the memory address of the two `type_info` structs.

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

---
### The Cost of RTTI

- **Memory:** It adds a small amount of data to every class (the `type_info` struct).
- **Speed:** `dynamic_cast` is slow. It has to traverse the inheritance tree at runtime to check validity.
- **Note:** Because RTTI has a pretty significant space performance cost, some compilers allow you to turn RTTI off as an optimisation ((like `-fno-rtti` in GCC/Clang)). Needless to say, if you do this, dynamic_cast won’t function correctly.

---