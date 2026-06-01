
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
	- **Result:** Objects of `Child` cannot access `Parent`'s public functions anymore. Only `Child` class and its _own_ children can use them.
    
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
        // ERROR! The compiler tried to call 'Parent()' BEFORE this line.
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
### Hardware Realities

#### 1. Where is the Parent object created?

In C++, a child object is physically a **single, contiguous block of memory**. The Parent object is not a separate entity floating somewhere else; it is embedded directly _inside_ the Child object. This is called a **Base Class Sub-Object**.

If your `Parent` class has an `int id` (4 bytes) and your `Child` class has an `int age` (4 bytes), the total size of the `Child` object in RAM will be 8 bytes.

When you instantiate the `Child`, the compiler carves out an 8-byte block of memory.

1. First, it runs the `Parent` constructor on the top 4 bytes to initialise the `id`.
2. Then, it runs the `Child` constructor on the bottom 4 bytes to initialise the `age`.

The Parent is completely fused into the Child.

#### 2. How can you access the Parent object?

Because the Parent is physically part of the Child, you have a few ways to access it:

- **Direct Access (Implicit):** If the Parent's variables or functions are `public` or `protected`, the Child can just use them directly as if it owned them. `myChild.id = 10;` // Directly accessing the Parent's variable

- **Scope Resolution (Explicit):** If the Child has a variable with the _exact same name_ as the Parent (shadowing), you use the Scope Resolution Operator `::` to specifically target the Parent's version. `myChild.Parent::id = 10;`

- **Upcasting (Pointer Magic):** Because the Parent is the first thing in the Child's memory block, you can safely point a Parent pointer at a Child object. The compiler simply restricts your view to only see the top "Parent" half of the memory block.

```cpp
Parent* p = &myChild; 
cout << p->id; // Perfectly valid!
```

### 3. What actually goes into memory?

It is crucial to understand that **only variables take up space in the object's memory.** If your `Child` class has 2 variables and 50 different member functions, the object in RAM only contains the 2 variables. The 50 functions are compiled into raw machine code and stored in a completely separate, read-only section of RAM called the **Code Segment**. When you call `myChild.doSomething()`, the CPU just jumps to the Code Segment and passes the memory address of your specific object (the hidden `this` pointer) into the function.

### 4.  Memory Hacking

The `private` keyword only exists in the C++ compiler's imagination. Once the code is compiled into raw machine code (1s and 0s), there is no such thing as private RAM. Because you know exactly where the Parent's variables live in memory, you can use raw pointers to completely bypass the C++ compiler's security and read the private data anyway.

```cpp
class Parent {
private:
    int private_data = 42; // This is the very first thing in RAM
};

class Child : public Parent {
public:
    int child_data = 100;
};

int main() {
    Child myChild;
    // cout << myChild.private_data; // Compiler blocks this
    // 1. Get the raw memory address of the object
    int* memory_hacker = (int*)&myChild;
    
    // When we cast (int*), we force the compiler to take this raw memory
    // address, and pretend it points to an integer so that when we defrence it, 
    // it reads 4 bytes.
    
    // 2. Read the very first 4 bytes (which we know is the Parent's private int)
    cout << "Hacked Private Data: " << *memory_hacker << endl; // Prints 42!
}
```

**Disclaimer:** You should _never_ do this in production code. It breaks if the compiler decides to rearrange memory for optimisation (padding). But it is proof that in C++, access modifiers (`public`, `private`) are just software illusions enforced by the compiler.

---
### Multiple Inheritance

 C++ allows **Multiple Inheritance**. If `Mom` inherits from `Grandma`, and `Dad` inherits from `Grandma`, and `Child` inherits from `Mom` AND `Dad`... The `Child` ends up with **TWO copies** of `Grandma`.

**The Issues:**
1. **Ambiguity:** If you call `child.grandmaFunc()`, the compiler panics: "Which path? Mom's Grandma or Dad's Grandma?"
2. **Waste:** You have two sets of variables for Grandma (e.g., two `age` variables).

**The Solution: `virtual` Inheritance**: We use the keyword `virtual` when `Mom` and `Dad` inherit from `Grandma`. This tells the compiler: _"If anyone else inherits from Grandma virtually, please **share** the same Grandma object. Do not create a new one."_

#### 1. The Normal Layout

If you do normal multiple inheritance, the compiler stacks the memory blocks on top of each other. A `Child` object in RAM looks exactly like this:

1. `Grandma` (Mom's copy)
2. `Mom`'s variables
3. `Grandma` (Dad's copy)
4. `Dad`'s variables
5. `Child`'s variables

#### 2. The Virtual Layout

When you type the word `virtual` in `class Mom : virtual public Grandma`, you are instructing the C++ compiler to physically alter the memory layout.

The compiler strips `Grandma` out of `Mom` and `Dad`. Instead, it injects a **Hidden Pointer** (specifically a Virtual Base Pointer) into both of them, and pushes the single `Grandma` object to the very bottom of the entire memory block.

Now, a `Child` object in RAM looks like this:

1. `Mom`'s variables + **Hidden Pointer to Grandma**
2. `Dad`'s variables + **Hidden Pointer to Grandma**
3. `Child`'s variables
4. `Grandma`'s variables **<-- Shared at the bottom!**

When `Mom` tries to read `age`, compiler follows the hidden pointer, jumps down to the bottom of the object, and reads `age` there." When `Dad` tries to read `age`, it follows its own pointer to the exact same physical spot.

### 3. Initialising `Grandma`

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


Once you understand that `Grandma` is pushed to the very bottom of the object, the constructor rule makes perfect sense. If `Mom` tries to initialise `Grandma`, where does she put the data?

- If you just create a standalone `Mom` object, `Grandma` is right below `Mom`.
- But if you create a `Child` object, `Grandma` is pushed all the way down below `Dad` and `Child`.
- If you create a `GrandChild` object, `Grandma` is pushed even further down.

Because `Mom` and `Dad` have no idea how deep the inheritance tree is going to go, they have no idea where `Grandma` actually lives in physical RAM. Therefore, the C++ compiler  dictates that **only the Most Derived class** (who knows the final, exact size of the entire object) is allowed to allocate and initialise `Grandma`.

If you write a function inside `Mom` that modifies `Grandma`'s age, the compiler uses that hidden pointer to find the shared `Grandma` object at the bottom of the memory block.


```cpp
#include <iostream>
using namespace std;

class Grandma {
public:
    int age;
    Grandma(int a) : age(a) { cout << "Grandma Built!\n"; } 
};

class Mom : virtual public Grandma {
public:
    // Mom's constructor tries to build Grandma(50), 
    // but the compiler IGNORES THIS when building a Child!
    Mom() : Grandma(50) {} 
    void momMakeOlder() {
        age += 5; // Mom is modifying Grandma's variable!
    }
}; 

class Dad : virtual public Grandma {
public:
    // Dad tries to build Grandma(60), compiler ignores this too!
    Dad() : Grandma(60) {} 
    void dadPrintAge() {
        cout << "Dad sees Grandma's age is: " << age << "\n";
    }
};

class Child : public Mom, public Dad {
public:
    // The Child is the ONLY one whose Grandma() call is actually executed.
    Child() : Grandma(90), Mom(), Dad() {} 
};

int main() {
    Child c; // Prints "Grandma Built!" exactly once.
    c.dadPrintAge();  // It is 90 (because Child built it).
    c.momMakeOlder(); 
    c.dadPrintAge(); // Prints 95
    return 0;
}
```

What happens if you just create a `Mom` object by herself, without a `Child`?

```cpp
Mom myMom; 
```

In this specific case, `Mom` _is_ the Most Derived Class! There is no `Child` below her. So, the compiler suddenly stops ignoring `Mom`'s constructor. It allows `Mom` to call `Grandma(50)` and build the object herself.

The compiler dynamically figures out who is at the very bottom of the chain at the exact moment you type `new` or declare the variable, and forces that specific class to be the builder. But no matter who builds it, everyone in the inheritance chain gets to share it.

---
### Example

1. **The Foundations (Level 1):** Two completely independent base classes: `Soul` and `Body`.
2. **The Hybrids (Level 2):** 
	- `Vampire` inherits **virtually** from both `Soul` and `Body`.
    - `Werewolf` inherits **virtually** from both `Soul` and `Body`. (It shares both).
    - `Cyborg` inherits **virtually** from `Soul`, but **NON-virtually** from `Body`. (It shares the Soul, but brings its own Body).
3. **The Monster (Level 3):** `Abomination` inherits from all 3 (`Vampire`, `Werewolf`, `Cyborg`).


```cpp
#include <iostream>
using namespace std;

// --- LEVEL 1: THE TWO FOUNDATIONS ---
class Soul {
public:
    Soul(int s) { cout << "   -> Soul (" << s << ") Built.\n"; }
};

class Body {
public:
    Body(int b) { cout << "   -> Body (" << b << ") Built.\n"; }
};

// --- LEVEL 2: THE PARENTS ---
class Vampire : virtual public Soul, virtual public Body {
public:
    Vampire() : Soul(-1), Body(-1) { cout << "   -> Vampire Built.\n"; }
};

class Werewolf : virtual public Soul, virtual public Body {
public:
    Werewolf() : Soul(-2), Body(-2) { cout << "   -> Werewolf Built.\n"; }
};

// The Rogue! Shares the Soul, but builds a private Body!
class Cyborg : virtual public Soul, public Body {
public:
    Cyborg() : Soul(-3), Body(404) { cout << "   -> Cyborg Built [NON-VIRTUAL BODY 404].\n"; }
};

// --- LEVEL 3: THE MONSTER ---
class Abomination : public Vampire, public Werewolf, public Cyborg {
public:
    // Abomination is the Most Derived Class.
    // It is mathematically forced to initialize BOTH virtual bases directly!
    Abomination() : Soul(999), Body(888), Vampire(), Werewolf(), Cyborg() {
        cout << "   -> Abomination Built.\n";
    }
};

int main() {
    cout << "--- SUMMONING ABOMINATION ---\n";
    Abomination myMonster;
    
    cout << "\n--- PROOF OF RAM ---\n";
    cout << "Vampire's Soul : " << myMonster.Vampire::Soul::id << "\n"; // 999
    cout << "Cyborg's Soul  : " << myMonster.Cyborg::Soul::id << "\n";  // 999 (Shared)
    cout << "Vampire's Body : " << myMonster.Vampire::Body::id << "\n"; // 888 (Shared)
    cout << "Cyborg's Body  : " << myMonster.Cyborg::Body::id << "\n";  // 404! (Private)
    return 0;
}
```

#### Output 

1. `Soul (999) Built.`
2. `Body (888) Built.` _(Virtual Base 2 is forced to the bottom alongside Soul!)_
3. `Vampire Built.`
4. `Werewolf Built.`
5. `Body (404) Built.` _(Cyborg's private, non-virtual Body is built)._
6. `Cyborg Built [NON-VIRTUAL BODY 404].`
7. `Abomination Built.`


1. **The Left-to-Right Depth-First Rule:** How does the compiler know whether to build `Soul` or `Body` first at the bottom? It reads your code left-to-right, top-to-bottom. Because `Vampire` inherited `virtual Soul, virtual Body`, the compiler builds `Soul` first, then `Body`.

2. **The Multi-Pointer Injection:** For `Vampire` and `Werewolf`, compiler injects **two hidden pointers** into their memory blocks, one pointing down to the shared `Soul`, and one pointing down to the shared `Body`

#### Memory Hierarchy

1. **`Vampire` Variables** + `[Ptr to Soul]` + `[Ptr to Body]`
2. **`Werewolf` Variables** + `[Ptr to Soul]` + `[Ptr to Body]`
3. **`Body` Variables (ID: 404)** _(Cyborg's private mechanical body!)_
4. **`Cyborg` Variables** + `[Ptr to Soul]`
5. **`Abomination` Variables**
6. **`Soul` Variables (ID: 999)** _(Shared Virtual Base 1)_
7. **`Body` Variables (ID: 888)** _(Shared Virtual Base 2)_

---
