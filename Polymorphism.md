
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

---

### Virtual Functions

We may want to store the Cats and Animals in a single vector (allows only one type), so we use upcasting, but we also want to maintain the specific Cat behaviour, so we use virtual functions

Virtual functions are member functions whose behaviour can be overridden in derived classes.

As opposed to non-virtual functions, the overriding behaviour is preserved even if there is no compile-time information about the actual type of the class. 

That is to say, if a derived class is handled using pointer or reference to the base class, a call to an overridden virtual function would invoke the behaviour defined in the derived class. 

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

Override is optional when declaring a member function inside a Derived Class that overrides the same virtual function in the Base Class, but it is good practice to clearly tell the compiler of the intended use of a function. The compiler will throw an error if you try to override a function that does not exist in the Base Class. Example:

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
- 
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
- 
**Step 2: Creation (Runtime)**
- `new Derived()` creates an object.
- It sets the hidden `__vptr` inside that object to point to the **Derived V-Table**.

**Step 3: The Call (Runtime)** When you run `ptr->speak()`, the program does NOT jump to a function. It follows a treasure map:
1. **Follow `ptr`** to the object in memory.
2. **Read the `__vptr`** inside that object.
3. **Go to the V-Table** pointed to by `__vptr`.
4. **Look up the function address** at the correct index (index 0 for `speak`).
5. **Jump** to that address (`Derived::speak`).

---
### CRTP

**CRTP (Curiously Recurring Template Pattern)** is a technique to achieve **polymorphism at compile-time**.

The name comes from the fact that the class inheritance looks like a loop or a paradox:

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

---
####  How it Works 

In a normal virtual function, the program looks at a hidden table (v-table) at runtime to find the right function.

In CRTP, the **Base class knows exactly who its child is** because we told it via the template parameter `T`.

Therefore, the Base class can cast its own `this` pointer to the Child type to call the Child's functions.

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

---

#### CRTP vs. Virtual Functions 

| **Feature**      | **Virtual Functions (Dynamic)**             | **CRTP (Static)**                                     |
| ---------------- | ------------------------------------------- | ----------------------------------------------------- |
| **Binding Time** | **Runtime** (Program decides while running) | **Compile Time** (Compiler decides while building)    |
| **Mechanism**    | **V-Table Lookup** (Pointer chasing)        | **Function Overloading** (Direct Call)                |
| **Speed**        | Slower (Cannot be inlined easily)           | **Ultra Fast** (Can be fully inlined)                 |
| **Flexibility**  | High (Can store `Base*` in a vector)        | Low (Cannot store different CRTP types in one vector) |
| **Memory**       | Adds overhead (vptr + vtable)               | **Zero Overhead**                                     |

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

