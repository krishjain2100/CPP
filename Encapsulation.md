### **Constructor** 
must be public, along with destructor

1. Assignment inside the constructor body (worse)
	- `x` is **default-initialised** : default constructor is called   
	- Then `x = v;` **assigns** to an already-existing object
```cpp
class A {
private:
    int x;
public:
    A(int v) {
        x = v;
    }
};
```

- Constructor initialisation list (better)
	- `x` is **directly constructed** with `v`
```cpp
class A {
    int x;
public:
    A(int v) : x(v) {}
};
```


 2. **Some members _must_ use initialisation lists**
	a) `const` members
	b) reference members: References **must** be initialized when the object is created.
```cpp
class A {
    const int x;
public:
    A(int v) : x(v) {}   // OK
};

A(int v) { 
	x = v;  // ERROR: assignment to const 
} 

class A {
    int& r;
public:
    A(int& x) : r(x) {}
};

```

---

3. **Order of initialisation**
	Members are initialised **in the order they are declared**, NOT the order in the init list.

```cpp
class A {
    int x;
    int y;
public:
    A() : y(10), x(y) {}  // BUG
    // x is initialised with y which is uninitalised
};
```

---

### The Three Keywords:

- **public**: Any part of the program can access public members.

- **private:** Only the class itself can access these. Not even the derived classes can see it. Only functions inside that specific class definition can use these.

- **protected**: The class itself and classes that inherit from it (entire lineage). It acts like private to the outside world, but acts like public to any class that inherits from it.

```cpp
class SecretBase {
	public: int code_public = 1;      // Everyone can see this
	protected: int code_protected = 2;   // Only me and my children can see this
	private: int code_private = 3;     // ONLY me. My children cannot see this.
};

class Spy : public SecretBase {
public:
    void reveal() {
        cout << code_public;     // OK
        cout << code_protected;  // OK: 
        // cout << code_private; // ERROR! 
    }
};

int main() {
    SecretBase base;
    cout << base.code_public;    // OK:
    // cout << base.code_protected; // ERROR! 
    // cout << base.code_private; // ERROR! 
}
```
---
### Explicit Keyword

By default, if a class constructor takes exactly **one argument**, the C++ compiler can use that constructor to automatically convert the argument type into the class type. The compiler can only do one implicit conversion at a time (remember that).

**The Bug (Implicit Conversion):**

```cpp
class BankAccount {
public:
    int balance;
    // Without 'explicit', int can magically become a BankAccount
    BankAccount(int b) : balance(b) {} 
};

void printBal(BankAccount acc) {
    cout << acc.balance;
}

int main() {
    BankAccount myAcc = 100; // VALID?! C++ converts 100 -> BankAccount(100)
    printBal(50); // VALID?!// C++ created a temporary account with balance 50. 
}
```

**The Fix:** Always put `explicit` in front of single-argument constructors.
```cpp
explicit BankAccount(int b) : balance(b) {}
```

---
### The Copy Constructor 

Pre-req: **A shallow copy** shares the underlying dynamic memory, while **a deep copy** creates a completely independent copy of that memory.

When you do `Account a = b;`,  it every variable from `b` to `a` byte-by-byte
- **Primitive variables (`int`, `double`):** This is fine.
- **Pointers:** This is a **DISASTER**, as it results in a shallow copy
	If `b` has a pointer to a heap array, `a` copies the _address_. Now both `a` and `b` point to the same memory.
	- `b` dies $\rightarrow$ deletes the memory.
	- `a` dies $\rightarrow$ deletes the _same_ memory again.
	- **Result:** Double Free Error (Crash).

You must write a custom **Copy Constructor** (make a deep copy) to allocate _new_ memory for the copy.

```cpp
class ArrayWrapper {
    int* data;
    int size;
public:

    ArrayWrapper(int s) : size(s) { data = new int[s]; }
    ~ArrayWrapper() {  delete[] data; }
    
    ArrayWrapper(const ArrayWrapper& other) {
        size = other.size;
        data = new int[other.size];
        for(int i=0; i<size; i++) {
            data[i] = other.data[i];
        }
    }
};
```

---
### Copy Assignment Operator

Handles **Assignment** (`a = b;`). This is different from the Copy Constructor because `a` **already exists** and might already hold memory. Steps

- Check for **Self-Assignment** (`a = a`). If we don't check, we might delete our own data before copying it.
- **Delete** the old memory (to prevent leaks).
- **Allocate** new memory.
- **Copy** the data.
- **Return `*this`** (to allow chaining `a = b = c`).

```cpp
ArrayWrapper& operator=(const ArrayWrapper& other) {
    // Step 1: Self-assignment check
    if (this == &other) {
        return *this; 
    }
    // Step 2: Clean up existing resource
    delete[] data; 

    // Step 3: Deep Copy (same logic as Copy Constructor)
    size = other.size;
    data = new int[other.size];
    std::copy(other.data, other.data + other.size, data);

    // Step 4: Return reference
    return *this;
}
```

---
### **Rule of Three**
If your class manages a resource (like a raw pointer) and needs a custom Destructor, you almost certainly need a Copy Constructor and a Copy Assignment Operator too, just to avoid double free crashes.

---
### Abstract Classes

Sometimes, a Base class is just a concept (like "Shape"). It doesn't make sense to have a "Generic Shape." You can only have a Circle, Square, etc.

**Syntax:** `virtual void functionName() = 0;`

**The Rules:**
1. **No Implementation:** The Base class  has _no code_ for this function.
2. **Mandatory Override:** Any class inheriting from this **MUST** provide the code for this function. If it doesn't, it becomes an abstract class too.
3. **Cannot Instantiate:** You cannot create an object of a class with atleast one pure virtual function. `Shape s;` is a compiler error.

```cpp
class Shape {
public:
    // Pure Virtual Function
    virtual void draw() = 0; 
    virtual ~Shape() {}
};

class Circle : public Shape {
public:
    void draw() override { cout << "Drawing Circle"; }
};

int main() {
    // Shape s; // ERROR: Cannot instantiate abstract class
    Shape* s1 = new Circle(); // OK: Pointer to base
    s1->draw(); // Calls Circle::draw()
}
```

The compiler-generated destructor (in case you forget to write one) is **NOT `virtual`** by default. This is dangerous for Abstract Classes because they are almost always used via base pointers (`Shape* s = new Circle();`). So remember to write one yourself

---

### `friend` Keyword

Sometimes you want to give **one specific outsider** access to your private data without opening it up to the whole world.

**Use Cases:**
1. **Operator Overloading:** Teaching `cout` how to print your object.
2. **Tightly Coupled Classes:** A `Node` class giving access to a `LinkedList` class.

```cpp
class Box {
private:
    int width;
public:
    Box(int w) : width(w) {}
    friend void printWidth(Box b);
};
// Note: This function is NOT a member of Box.
// It is a regular global function.
void printWidth(Box b) {
    // It can access 'b.width' because it is a friend.
    cout << "Width: " << b.width << endl;
}
```

---

### Operator Overloading

You can redefine how standard symbols (`+`, `-`, `==`, `<<`) work with your custom objects.
We can implement this as a **Member Function** or a **Friend Function**. (Member is usually preferred for `+`, `-`, etc.)


```cpp
class Vector2D {
public:
    int x, y;
    Vector2D(int x, int y) : x(x), y(y) {}

    // Syntax: ReturnType operatorOP(Argument)
    // "When someone writes 'this + other', run this code."
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(this->x + other.x, this->y + other.y);
    }
    
    // Comparison Operator
    bool operator==(const Vector2D& other) const {
        return (this->x == other.x) && (this->y == other.y);
    }
};

int main() {
    Vector2D v1(1, 2);
    Vector2D v2(3, 4);
    Vector2D v3 = v1 + v2; // Calls v1.operator+(v2)
    if (v1 == v2) cout << "Same!";
}
```

#### The Special Case:  `<<` (Printing)

You cannot make this a member function because the syntax `cout << obj` puts `cout` on the left. To fix this, we implement it as a **Global Friend Function**.

```cpp
class Person {
    string name;
public:
    Person(string n) : name(n) {}
    // Friend declaration
    friend ostream& operator<<(ostream& os, const Person& p);
};
// We return 'ostream&' to allow chaining: cout << p1 << p2;
ostream& operator<<(ostream& os, const Person& p) {
    os << "Person(" << p.name << ")";
    return os;
}
```
