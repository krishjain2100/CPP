
There are **three ways** to declare a child class in C++. 
The mode can be **`public`**, **`protected`**, or **`private`**.

1. **Public Inheritance**
	**Syntax:** `class Child : public Parent` (Default for `struct`)
    - `public` members in Parent $\rightarrow$ remain `public` in Child.
    - `protected` members in Parent $\rightarrow$ remain `protected` in Child.
    - `private` members $\rightarrow$ remain hidden (always).

2. **Protected Inheritance**
	Syntax: `class Child : protected Parent`
	    - `public` members in Parent $\rightarrow$ become **`protected`** in Child.
	    - `protected` members in Parent $\rightarrow$ remain `protected` in Child.
	- **Result:** Objects of `Child` cannot access `Parent`'s public functions anymore. Only `Child` and its _own_ children can use them.
    
 3. **Private Inheritance** 
	Syntax: `class Child : private Parent` (Default for `class`)
	    - `public` members in Parent $\rightarrow$ become **`private`** in Child.
	    - `protected` members in Parent $\rightarrow$ become **`private`** in Child.
	- **Result:** The lineage ends here. If you derive a `GrandChild` from this `Child`, the `GrandChild` gets **nothing** from the original `Parent`. The connection is severed.

---
### Parent Constructor
When you create a Child object, the **Parent is created FIRST**. By the time the code enters the curly braces `{ ... }` of the Child's constructor, the Parent is _already_ alive.

```cpp
class Parent {
public:
    int id;
    Parent(int x) { id = x; } // Parent MUST have an int to exist
};

class Child : public Parent {
public:
    Child(int x, int y) {
        // ERROR! The compiler tried to call 'Parent()'  BEFORE this line.
        // Since Parent() doesn't exist, the code crashes here.
    }
};
```

```cpp
class Child : public Parent {
public:
    Child(int x, int y) : Parent(x) { 
        // Now Parent is built using x. We can do whatever we want.
        cout << "Child created!";
    }
};
```

---
### Order of Construction and Destruction
When you create a child object, C++ automatically builds the parent first. When you destroy it, C++ destroys the child first.

```cpp
class Parent {
public:
    int parent_data;
    Parent() {
        parent_data = 100;
        cout << "Parent Constructor" << endl;
    }
    ~Parent() {
        cout << "Parent Destructor" << endl;
    }
};

class Child : public Parent {
public:
    int child_data;
    Child() {
        // The Parent constructor has ALREADY run here.
        // That is why we can safely use 'parent_data'.
        child_data = parent_data + 50; 
        cout << "Child Constructor " << parent_data  << endl;
    }
    ~Child() {
        // The Parent is STILL alive here.
        // We can safely use parent_data one last time.
        cout << "Child Destructor " << parent_data << endl;
    }
};

int main() {
    {
        Child myHouse; // Object created here
    } // Object goes out of scope here, destructors
}
```

```
Parent Constructor
Child Constructor 100
Child Destructor 100
Parent Destructor
```

---
### The Virtual Destructor

When you `delete` a pointer, the compiler checks the type of the pointer.
- If you delete an `Animal*`, the compiler calls `~Animal()`.
- It does **not** check if the object is actually a `Dog` **UNLESS** the destructor is marked `virtual`.
    
**The Consequence (Memory Leak):** If the destructor is not virtual, the "Child" part of the object is never cleaned up.

```cpp
class Base {
public:
    // WITHOUT 'virtual', this is a "Staticaly bound" function, i.e the decisions about it are taken at compile time using the provided types
    ~Base() { cout << "Base cleaned up."; } 
};

class Derived : public Base {
    int* heavyData;
public:
    Derived() { heavyData = new int[1000]; } // 1. Allocate memory
    ~Derived() { 
        delete[] heavyData; // 2. Free memory
        cout << "Derived cleaned up."; 
    }
};

int main() {
    Base* ptr = new Derived(); // Upcasting
    delete ptr; 
    // Compiler sees: ptr is Base*.
    // Compiler calls: ~Base().
    // Compiler IGNORES: ~Derived().
    // RESULT: The 1000 ints in 'heavyData' are lost in RAM forever (Leak).
}
```

**The Fix:** Change `~Base()` to `virtual ~Base()`. Now, `delete ptr` triggers the **Dynamic** lookup. It finds the object is a `Derived`, calls `~Derived()` (cleaning the array), which then automatically calls `~Base()`.

---
### Multiple Inheritance

 C++ allows **Multiple Inheritance**. If `Mom` inherits from `Grandma`, and `Dad` inherits from `Grandma`, and `Child` inherits from `Mom` AND `Dad`... The `Child` ends up with **TWO copies** of `Grandma`.

**The Issues:**
1. **Ambiguity:** If you call `child.grandmaFunc()`, the compiler panics: "Which path? Mom's Grandma or Dad's Grandma?"
2. **Waste:** You have two sets of variables for Grandma (e.g., two `age` variables).
    
**The Solution: `virtual` Inheritance** We use the keyword `virtual` when `Mom` and `Dad` inherit from `Grandma`. This tells the compiler: _"If anyone else inherits from Grandma virtually, please **share** the same Grandma object. Do not create a new one."_


```cpp
class Grandma {
public:
    int age;
    Grandma(int a) : age(a) {} 
};

class Mom : virtual public Grandma {}; 
class Dad : virtual public Grandma {};

class Child : public Mom, public Dad {
    // Now 'Child' has only ONE 'Grandma' sub-object.
    // 'Mom' and 'Dad' both point to the SAME 'age' variable.
};

int main() {
    Child c;
    c.Mom::age = 80;
    cout << c.Dad::age; // Output: 80 (It's the same variable!)
}
```

Because `Grandma` is now a shared object, **who is responsible for initialising her?**
- Normally, `Mom` calls `Grandma()`.
- And `Dad` calls `Grandma()`.
- But now there is only **one** `Grandma`. Who calls it?

**Rule:** In Virtual Inheritance, the **Most Derived Class** (the `Child`) is responsible for calling the `Grandma` constructor directly.

```cpp
class Child : public Mom, public Dad {
public:
    Child() : Grandma(90), Mom(), Dad() { 
        // You MUST initialize Grandma here directly!
        // Mom() and Dad() ignore their Grandma calls.
    }
};
```

---