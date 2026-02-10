### 1. The Static Variable

**Behaviour:** It is created only **once** (the first time the function runs). It **never dies** until the program closes. It keeps its value between function calls.

```cpp
void counter() {
	static int count = 0; // Created ONCE. Skipped on future calls.
	count++;
	cout << count << endl;
} // 'count' is NOT destroyed. It sits in a special memory area.

int main() {
	counter(); // Prints 1
	counter(); // Prints 2
	counter(); // Prints 3
}
```

Its memory location is **static** (stationary/fixed). It sits in one spot in memory for the entire life of the program.

### 2. Static Inside a Class (The "Shared" Data)

When you put `static` on a member variable, it is no longer owned by the object. It is owned by the **Class itself**.

- **Normal Variable:** Every object gets its own copy.
- **Static Variable:** All objects share **ONE** copy.

**Common Use Case:** Counting how many objects exist.

```cpp
class Robot {
public:
	// 1. Declaration (Inside class)
	static int population;
	Robot() {
		population++; // Every time a robot is born, increase the SHARED counter
	}
};

// 2. Definition (OUTSIDE class - Required!)
int Robot::population = 0;

int main() {

	Robot r1;
	Robot r2;
	Robot r3;
	
	// They all share the same variable!
	cout << r1.population; // Prints 3
	cout << Robot::population; // Prints 3 (Better way to access it)
}
```

### 3. Static at File Scope (The "Hidden" Global)

If you create a global variable in `file1.cpp`, it is technically visible to `file2.cpp` if they try to look for it. This causes "Naming Collisions" (linker errors).

Adding `static` to a global variable makes it **invisible** to other files.

```cpp
// File1.cpp
static int secret_code = 1234; // Only File1 can see this.
```

---
### `static_cast`

It is a **compile-time** cast. It tells the compiler:

> _"I know these two types are different, but they are related (or compatible). Please convert X to Y. If they are totally unrelated, stop me with an error."_

It is different from:
- **C-Style Cast (`(int)x`):**  It tries `static_cast`, `const_cast`, and `reinterpret_cast` until something works. It is dangerous and hard to read.
- **`dynamic_cast`:** "Safe but Slow". It checks at **runtime** if the conversion is valid (using RTTI/v-table). If it fails, it returns `nullptr`.
    

#### Behaviour

- **Zero Overhead:** Since it happens at compile-time, it generates the exact same machine code as if the types were originally correct. It creates no runtime slowdown.

- **Safety Check:** It checks if a conversion is _sensible_.
    - `int` to `float`? **Yes.**
    - `Base*` to `Child*`? **Yes.**
    - `std::string` to `int`? **No** (Compile Error).
    - `UnrelatedClassA*` to `UnrelatedClassB*`? **No** (Compile Error).

#### Down-casting is dangerous

- **Upcasting (Child -> Base):** Always safe. Implicit.
- **Down-casting (Base -> Child):** Risky.

If you `static_cast` a `Base*` to a `Child*`, the compiler **trusts you**. It assumes you know for a fact that the object really is a `Child`

```cpp
class Base {};
class Derived : public Base { void bark() {} };
class Other : public Base {};

Base* ptr = new Other(); // Actually an 'Other' object

// BAD STATIC CAST
// Compiler allows this because Base and Derived are related.
// But at runtime, this pointer is NOT a Derived.
Derived* d = static_cast<Derived*>(ptr); 

d->bark(); // UNDEFINED BEHAVIOR (Crash, or worse)
// as in memory it is Other and bark() is not present there
```

**Why do we use it in CRTP then?** Because CRTP guarantees the type via the template.
`class MyClass : public Base<MyClass>` 
The syntax itself ensures that `Base<MyClass>` is _always_ the parent of `MyClass`.
Therefore, casting `Base<MyClass>*` down to `MyClass*` is 100% safe by definition.

### Further explanation:
#### 1. The Memory Layout 

Classes are just blueprints for how memory is arranged.

- **`class Other`** looks like this in RAM:
    ```[ Base Part ]  <-- Size: 1 byte (empty classes usually take 1 byte)```
    
- **`class Derived`** (if it had data) would look like this:
```
[ Base Part ]
[ Derived Variables (e.g., int speed) ] <-- Extra memory!
```
    
#### 2. The Pointer's Perspective

A pointer type (like `Derived*`) is just a pair of goggles that tells the CPU how to look at a chunk of memory.

- **`Base*` goggles:** only sees the Base part.
- **`Derived*` goggles:** expects to see the Base part and the Derived variables.

#### 3. Issues
```cpp
Base* ptr = new Other(); // You created a tiny object (Just [Base])
Derived* d = static_cast<Derived*>(ptr); // You put on "Derived Goggles"
```

You are telling the compiler: 
"This pointer points to a massive `Derived` object with all the extra `Derived`material"_.  
The compiler will trust you

Now, you execute `d->bark()`
- If `bark()` tries to print a variable (like `speed`), the CPU goes to the start of the object, skips the Base part, and reads the integer."
- But in reality there is **no integer there**. The pointer runs off the end of the `Other` object and reads random garbage memory (or crashes because that memory belongs to another program).


