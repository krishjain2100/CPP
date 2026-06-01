What Is Polymorphism? Same interface, different behaviour

1. **Dynamic Polymorphism (`virtual`):** Decisions made at **Runtime** (Slower, flexible).
2. **Static Polymorphism (`template`):** Decisions made at **Compile Time** (Faster, rigid).

**`templates` vs `virtuals`:**
Use `templates` for performance and generic logic. 
Use `virtuals` when the type is truly unknown until the user clicks a button or loads a file.

---
### Upcasting

Below is an example of _Static Polymorphism_, i.e., which function will be called is decided at compile time


```cpp
struct Animal { 
	void speak() { 
		cout << "Default" << endl; 
	}
};
struct Cat : Animal {
	void speak() { 
		cout << "Cat speaks" << endl; 
	}
};

int main() {
    Cat c;
    c.speak(); // Cat speaks
    
    Animal &a1 = c; // upcasing
    Animal* a2 = &c; // upcasting
    Animal a3 = c;
    
	cout << &c << endl; //  0x16f7db010
    cout << &a1 << endl; // 0x16f7db010
    cout << &a2 << endl; // 0x16f7db020
    cout << &a3 << endl; // 0x16f7db030
    
    a1.speak(); //Default
    a2->speak(); //Defualt
    a3.speak(); //Defualt
}
```

When you use a reference (`&a1 = c`) or a pointer (`*a2 = &c`), you are not creating a new object. You are simply putting on a pair of "Animal glasses" to look at the existing `Cat` object in RAM.
Because `a3` is strictly an `Animal`, it only has enough physical byte space to hold the `Animal` parts. The C++ compiler  copies the `Animal` foundation of `c` into the new memory slot for `a3`, and does not copy  the `Cat` part.

---
### Virtual Functions

We may want to store the Cats and Animals in a single vector (allows only one type), so we use upcasting, but we also want to maintain the specific Cat behaviour, so we use virtual functions

Virtual functions are member functions whose behaviour can be overridden in derived classes.

As opposed to non-virtual functions, the overriding behaviour is preserved even if there is no compile-time information about the actual type of the class. That is to say, if a derived class is handled using pointer or reference to the base class, a call to an overridden virtual function would invoke the behaviour defined in the derived class. 

Such a function call is known as _virtual function call_ or _virtual call_.

```cpp
struct Base {
    virtual void f() {
        cout << "base\n";
    }
};

struct Derived : Base {
    void f() override { // 'override' is optional
        cout << "derived\n";
    }
};
 
int main() {
    Base b;
    Derived d;
    // virtual function call through reference
    Base& br = b; // the type of br is Base&
    Base& dr = d; // the type of dr is Base& as well
    br.f(); // prints "base"
    dr.f(); // prints "derived"
 
    // virtual function call through pointer
    Base* bp = &b; // the type of bp is Base*
    Base* dp = &d; // the type of dp is Base* as well
    bp->f(); // prints "base"
    dp->f(); // prints "derived"
}
```

If a function is declared `virtual` in the Base class, any function in a Derived class with the **same signature** (name + parameters) is automatically `virtual`, whether you write the keyword or not.

---
### Override

Override is optional when declaring a member function inside a Derived Class that overrides the same virtual function in the Base Class, but it is good practice to clearly tell the compiler of the intended use of a function. The compiler will throw an error if you try to override a function that does not exist in the Base Class (so getting the function's signature wrong is prevented). Example:

```cpp
struct Base {
    virtual void foo(int);
};

struct Derived : Base {
    void foo(double);   // intended override, but NOT an override
	// compiles fine
};

// override saves you
struct Derived : Base {
    void foo(double) override; // ERROR: compile-time error
};
```

#### **Difference from Overloading**

**Overloading** happens when:
- Functions have the **same name**
- **Different parameter lists**
- **In the same scope**

Example:

```cpp
struct A {
    void foo(int);
    void foo(double);
};
```

**Overriding** happens when:
- Function is **virtual in base**
- Derived provides **same signature**
- Calls are resolved **at runtime**
- Uses **dynamic dispatch (vtable)**

Example:

```cpp
struct Base {
    virtual void foo(int);
};

struct Derived : Base {
    void foo(int) override;
};
```

---
### Working of Virtual Functions

It relies on two hidden things the compiler adds to your code:
1. **The V-Table (The Map):** A static array of function pointers.
2. **The V-Ptr (The Pointer):** A hidden pointer inside every object.
#### 1. The V-Table (One per Class)

When a class has _any_ virtual functions, the compiler creates a **hidden table** (array) for that class.
- This table contains the **addresses** of the virtual functions for that specific class.
- **Base Class V-Table:** Points to `Base::function`.
- **Derived Class V-Table:** Points to `Derived::function` (if overridden) or `Base::function` (if not).
#### 2. The V-Ptr (One per Object)

Every time you create an object of a class with virtual functions, the compiler secretly adds a pointer member (let's call it `__vptr`) to the very top of that object. 
This pointer holds the address of the **V-Table** belonging to that object's class.
####  3. Step-by-Step:

Let's look at the memory.

```cpp
class Base {
public:
    virtual void speak() { cout << "Base"; }
};

class Derived : public Base {
public:
    void speak() override { cout << "Derived"; }
};

int main() {
    Base* ptr = new Derived();
    ptr->speak(); 
}
```

**Step 1: Setup (Compile Time)**
- Compiler creates **V-Table for Base**: `[ &Base::speak ]`
- Compiler creates **V-Table for Derived**: `[ &Derived::speak ]`

**Step 2: Creation (Runtime)**
- `new Derived()` creates an object.
- It sets the hidden `__vptr` inside that object to point to the **Derived V-Table**.

**Step 3: The Call (Runtime)** When you run `ptr->speak()`, the program does NOT jump to a function. It follows a treasure map:
1. **Follow `ptr`** to the object in memory.
2. **Read the `__vptr`** inside that object.
3. **Go to the V-Table** pointed to by `__vptr`.
4. **Look up the function address** at the correct index (index 0 for `speak`).
5. **Jump** to that address (`Derived::speak`).

#### More Complex Example

```cpp
class Base {
public:
    int base_data = 10;
    // 1. Non-virtual (Only in Base)
    void normalBase() { std::cout << "Direct Base Jump\n"; } 
    // 2. Virtual (Will be overridden)
    virtual void sharedVirt() { std::cout << "Base Shared\n"; } 
    // 3. Virtual (Will NOT be overridden)
    virtual void keptVirt() { std::cout << "Base Kept\n"; } 
};

class Derived : public Base {
public:
    int derived_data = 20;
    // 4. Non-virtual (Only in Derived)
    void normalDerived() { std::cout << "Direct Derived Jump\n"; } 
    // 5. Overriding the Base virtual function
    void sharedVirt() override { std::cout << "Derived Shared\n"; } 
};
```

###### **Step 1: Compile-Time** 

When you compile, compiler immediately separates functions into two categories: **Static Bound** (non-virtual) and **Dynamic Bound** (virtual).

For `normalBase()` and `normalDerived()`, the compiler realises they are not virtual and hardcodes their memory addresses directly into the executable file. They are not present in any table. They take up **zero** space in the object's RAM and have zero execution overhead.

The compiler sees the `virtual` keyword and creates two hidden static arrays in the global read-only memory segment of your program.

**How Indices are Defined:** The compiler assigns an integer index to every virtual function strictly based on the order they appear in the `Base` class.
- `sharedVirt()` is seen first -> **Index 0**.
- `keptVirt()` is seen second -> **Index 1**.

**The Base V-Table:**
- `[0]` -> Points to the machine code for `Base::sharedVirt`
- `[1]` -> Points to the machine code for `Base::keptVirt`

**The Derived V-Table:** The compiler copies the Base V-Table, and then updates the pointers _only_ for the functions `Derived` chose to override. The indices are permanently locked.
- `[0]` -> Points to the machine code for `Derived::sharedVirt` _(Updated!)_
- `[1]` -> Points to the machine code for `Base::keptVirt` _(Inherited unchanged!)_

##### Step 2: Object Creation: `Base* ptr = new Derived();`

The OS allocates a block of RAM for the `Derived` object. Because the class hierarchy contains virtual functions, the compiler secretly injects a hidden pointer (`__vptr`) at **Offset 0** (the very top of the memory block). If you look at the raw physical bytes of this specific `Derived` object in RAM, it looks exactly like this: _(Note: Pointers are 8 bytes on a 64-bit system, ints are 4 bytes)._

- **Byte 0-7:** `[ __vptr ]` _(Contains the memory address of the Derived V-Table)_
- **Byte 8-11:** `[ base_data ]` _(Value: 10)_
- **Byte 12-15:** `[ derived_data ]` _(Value: 20)_

##### Step 3: Pointer Chasing

Now, let's look at what the CPU does when you call different functions using `ptr`. 
Remember, the compiler only knows `ptr` is a `Base*`.
###### Scenario A: The Non-Virtual Base Function
`ptr->normalBase();`

1. **The Compiler:** "This function is not virtual. I don't care what object is actually in RAM. `ptr` is a `Base*`, so I will just jump directly to the hardcoded address of `Base::normalBase`."
2. **The CPU:** Executes a direct `JMP` instruction.
3. **Cost:** 1 clock cycle. Instant. It completely ignores the `__vptr`.

###### Scenario B: The Non-Virtual Derived Function
`ptr->normalDerived();`

1. **The Compiler:** "Wait. `ptr` is a `Base*`. I am looking at the `Base` blueprint. There is no `normalDerived` function here."
2. **Result:** Fatal Compile Error. Even though the object in RAM _is_ a Derived, the static type of the pointer acts as a blinder. Because it's not virtual, the compiler refuses to look at the object's actual identity.

###### Scenario C: The Virtual Call
`ptr->sharedVirt();`

The compiler knows this is virtual, so does not hardcode a jump. Instead, it generates a sequence of raw Assembly instructions to chase the pointers at runtime.

1. **Dereference the Object:** The CPU looks at `ptr` to find where the object lives in RAM (e.g., `0x1000`).
2. **Fetch the V-Ptr:** The CPU reads the first 8 bytes at `0x1000`. This is the `__vptr`. It tells the CPU where the V-Table is located (e.g., `0x8000`).
3. **Apply the Index Offset:** The compiler knows `sharedVirt` is exactly **Index 0**. A memory address is 8 bytes. So the CPU calculates: `0x8000 + (0 * 8) = 0x8000`.
4. **Fetch the Function Address:** The CPU reads the 8 bytes stored at `0x8000` in the V-Table. This gives it the physical memory address of the actual function's machine code (e.g., `0xAAAA`).
5. **The Jump:** The CPU executes a jump to `0xAAAA`. Because the `__vptr` pointed to the _Derived_ V-Table, `0xAAAA` happens to be the code for `Derived::sharedVirt()`.
6. **Cost:** 3 memory lookups before the function even begins.

---
### CRTP
First read [[Templates]].

**CRTP (Curiously Recurring Template Pattern)** is a technique to achieve **polymorphism at compile-time**. The name comes from the fact that the class inheritance looks like a loop or a paradox:

```cpp
// 1. Base is a Template
template <typename T>
class Base { ... };

// 2. Derived inherits from Base... 
// BUT passes ITSELF as the template argument!
class Derived : public Base<Derived> { ... };
```

**Question**: How can `Derived` inherit from `Base<Derived>` if `Derived` isn't fully defined yet?
**The Answer:** C++ allows this because at the point of inheritance, `Derived` is an "incomplete type," but that is enough for the compiler to set up the inheritance relationship.

####  How it Works 

In a normal virtual function, the program looks at a hidden table (v-table) at runtime to find the right function. In CRTP, the **Base class knows exactly who its child is** because we told it via the template parameter `T`. Therefore, the Base class can cast its own `this` pointer to the Child type to call the Child's functions.

```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        // 1. Cast 'this' (Base*) to 'Derived*'
        // 2. We know this is safe because Derived inherits from Base<Derived>
        Derived* ptr = static_cast<Derived*>(this);
        // 3. Call the function directly (No vtable! No lookup!)
        ptr->implementation();
    }
};

class MyClass : public Base<MyClass> {
public:
    void implementation() {
        cout << "MyClass doing work!";
    }
};

int main() {
    MyClass obj;
    obj.interface(); // Calls Base::interface -> calls MyClass::implementation
}
```

#### How did down-casting become safe here ?

Base object still doesn't have the methods and variables of Derived. 
If you were to somehow create a standalone `Base<MyClass>` object and try to cast its `this` pointer to `MyClass`, the program would crash. However, **in CRTP, you never create a standalone `Base` object.** You only ever instantiate the `Derived` object (`MyClass obj;`).

Because of this, the `this` pointer inside the `Base` class is never pointing to _just_ a Base object; it is pointing to the Base"sub-object that lives entirely inside the larger `MyClass` object.

#### The Memory Layout 
When you write `MyClass obj;` in `main()`, the C++ compiler allocates a single block of memory for the entire `MyClass` object.

```plaintext
  Memory Layout of 'obj'
 --------------------------------- 
|  Base<MyClass> subobject        | <--- Inside interface(), 'this' points here
|  (Contains Base variables)      |
|---------------------------------|
|  MyClass specific parts         | 
|  (Contains MyClass variables)   |
 ---------------------------------  
 <--- After static_cast, the pointer sees this whole box
```

The `this` pointer is temporarily scoped to look at the `Base<MyClass>` part of the memory block. When you do this: `Derived* ptr = static_cast<Derived*>(this);`, you are telling the compiler to widen the view.

Because the object was created as a `MyClass` in the first place, that memory _actually exists_. The downcast is perfectly safe.

---
#### CRTP vs. Virtual Functions 

| **Feature**      | **Virtual Functions (Dynamic)**             | **CRTP (Static)**                                     |
| ---------------- | ------------------------------------------- | ----------------------------------------------------- |
| **Binding Time** | **Runtime** (Program decides while running) | **Compile Time** (Compiler decides while building)    |
| **Mechanism**    | **V-Table Lookup** (Pointer chasing)        | **Function Overloading** (Direct Call)                |
| **Speed**        | Slower (Cannot be inlined easily)           | **Ultra Fast** (Can be fully inlined)                 |
| **Flexibility**  | High (Can store `Base*` in a vector)        | Low (Cannot store different CRTP types in one vector) |
| **Memory**       | Adds overhead (vptr + vtable)               | **Zero Overhead**                                     |
|                  |                                             |                                                       |

**Visualising the Speed:**
- **Virtual:** `Base` -> (Look at vtable) -> (Find address) -> `Derived::func()`
- **CRTP:** `Base` -> `Derived::func()` (The compiler effectively deletes the middleman).

---
#### Mix-ins (Adding Functionality)

CRTP isn't just for optimisation; it's heavily used to **inject functionality** into classes.
Imagine you want to make a class that has `==` and `!=` operators.

Instead of writing them for every single class, you create a CRTP `Equality` class.

```cpp
template <typename T>
struct Equality {
    // We only implement '!='
    // It relies on 'T' having '==' implemented
    bool operator!=(const T& other) const {
        // Cast 'this' to T to call the actual '==' operator
        const T* ptr = static_cast<const T*>(this);
        return !(*ptr == other); 
        // Can't use (*ptr != other) as that would result in recursion
    }
};

class Apple : public Equality<Apple> { // Inherit the logic!
public:
    int size;
    // We only implement '=='
    bool operator==(const Apple& other) const {
        return size == other.size;
    }
    // We get '!=' for free from the Base class!
};

int main() {
    Apple a1{10}, a2{20};
    if (a1 != a2) { // Uses Equality<Apple>::operator!=
        cout << "Different!";
    }
}
```

---
#### Multiple Inheritance (Dual V-Ptr) (did not understand, to be updated)

What if a class inherits from two _completely unrelated_ base classes that both have virtual functions?

```cpp
class Printer {
public:
    virtual void print() { cout << "Printing..."; }
};

class Scanner {
public:
    virtual void scan() { cout << "Scanning..."; }
};

class Copier : public Printer, public Scanner {
public:
    void print() override { cout << "Copier Printing..."; }
    void scan() override { cout << "Copier Scanning..."; }
};
```

 **A `Copier` object has TWO hidden V-Ptrs.**

Why? Because of polymorphism and casting. If you write `Scanner* s = new Copier();`, the pointer `s` is looking strictly at the `Scanner` part of the object. When you call `s->scan()`, the compiler expects to find a V-Ptr at the very top of the memory it is looking at. It has no idea it's actually looking at a `Copier`.

To make this work, the compiler stacks the memory and injects a V-Ptr at the top of _every_ base class block.

**The Physical RAM Dump of a `Copier` object:**

1. **`[ __vptr_Printer ]`** _(Points to Copier's custom Printer V-Table)_
2. `Printer` Variables
3. **`[ __vptr_Scanner ]`** _(Points to Copier's custom Scanner V-Table)_
4. `Scanner` Variables
5. `Copier` Variables

If you do `Scanner* s = new Copier();`, the compiler actually does **Pointer Arithmetic**. It shifts the memory address of the pointer down by 8 bytes so it physically points to the middle of the `Copier` object, landing exactly on `__vptr_Scanner`.

ut what are these 2 different vptrs pointning to, they point to the smae Copier's Vtable right ?

This is the most logical assumption to make, but it is actually **incorrect**.

They do **not** point to the same V-Table.

If you inherit from two base classes with virtual functions, the C++ compiler actually generates **multiple, completely separate V-Tables** for the `Copier` class.

To understand why the compiler is forced to build multiple V-Tables, we have to look at a brutal mathematical problem: **The Index Collision.**

### The Problem: Who gets Index 0?

Remember how the compiler assigns a hardcoded integer Index to every virtual function? Let's look at the original blueprints for your base classes:

C++

```
class Printer {
    virtual void print(); // Compiler assigns Index 0
};

class Scanner {
    virtual void scan();  // Compiler assigns Index 0
};
```

Both `print()` and `scan()` were the very first virtual functions in their respective classes. Therefore, they both claim **Index 0**.

Now, imagine if the `Copier` class only had _one_ V-Table. What function does the compiler put at Index 0?

- If it puts `Copier::print` at Index 0, then a `Scanner*` pointing to this object will accidentally call `print()` when it meant to call `scan()`.
    
- If it puts `Copier::scan` at Index 0, then a `Printer*` will accidentally call `scan()` when it meant to call `print()`.
    

### The Solution: Primary and Secondary V-Tables

Because the compiler cannot change the hardcoded indices that the pointers are expecting, it solves the collision by creating two distinct lookup arrays for the exact same `Copier` class.

#### 1. The Primary V-Table (For Printer)

Because `Printer` was listed first in your code (`class Copier : public Printer, public Scanner`), it gets to be the "Primary" base.

- `__vptr_Printer` points to this table.
    
- **Index 0** in this table points to the machine code for `Copier::print()`.
    

#### 2. The Secondary V-Table (For Scanner)

Because `Scanner` was listed second, the compiler builds a completely separate "Secondary" V-Table just to satisfy the `Scanner*` pointer.

- `__vptr_Scanner` points to this table.
    
- **Index 0** in this table points to the machine code for `Copier::scan()`.
    

Now, both pointers get exactly what they want. A `Printer*` looks at Index 0 of the Primary table and gets `print`. A `Scanner*` looks at Index 0 of the Secondary table and gets `scan`.

### The Final Boss: The "Thunk" (Fixing the `this` pointer)

There is one final, mind-bending problem the compiler has to solve behind the scenes.

When you call `s->scan();`, you are executing `Copier::scan()`. Inside that function, you might want to access a `Copier` variable. That means the hidden `this` pointer inside the function needs to point to the very top of the `Copier` object (Address `0x1000`).

But remember what we just learned about Multiple Inheritance? The `Scanner* s` pointer was shifted! It is currently pointing to the _middle_ of the object (Address `0x1010`).

If the CPU just jumped straight into `Copier::scan()`, the `this` pointer would be wrong. It would be shifted by 16 bytes, and if you tried to read a variable, you would read garbage memory.

To fix this, the **Secondary V-Table doesn't actually point directly to your function.** It points to a **Thunk**.

A Thunk is a tiny, invisible, compiler-generated snippet of Assembly code that does exactly one thing: it does pointer arithmetic in reverse.

**Here is the exact physical execution path of `s->scan()`:**

1. CPU reads `s` (`0x1010`).
    
2. CPU reads `__vptr_Scanner` at that address.
    
3. CPU goes to the Secondary V-Table, Index 0.
    
4. CPU jumps to the **Thunk**.
    
5. The Thunk executes: `"Subtract 16 bytes from the 'this' pointer, moving it back to 0x1000."`
    
6. The Thunk directly jumps into the real `Copier::scan()` function.

---