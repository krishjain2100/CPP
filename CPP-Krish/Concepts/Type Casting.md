###  The C++ Casts

#### A. `static_cast` 

- **Primary Intent:** Well-defined, mathematically sound conversions between compatible types.
- **Timing:** Evaluated entirely at **compile-time**.
- **Under the Hood:** 
	- If converting numbers (e.g., `int` to `float`), the compiler generates assembly instructions to alter the bit pattern (e.g., converting a 2's complement integer to an IEEE-754 floating-point format). 
	- If casting pointers within a safe inheritance tree (Upcasting or Downcasting without virtual checks), it calculates the fixed byte offsets at compile time.
- **Safety:** If types are completely incompatible (e.g., `int*` to `float*`), the compiler throws a **fatal compilation error**.
- **Memory Risk:** Safe for upcasting. For downcasting (Base to Derived), it performs no runtime verification. If the Base pointer does not actually point to a Derived object, using the casted pointer results in **Undefined Behaviour (UB)**.

**Note:** int* to float* is bad because this instruction just changes the glasses for compiler but not the underlying bits arrangement as is done for int to float so representation of 5 in int could be a representation for something else under float.

It tells the compiler:
> _"I know these two types are different, but they are related (or compatible). Please convert X to Y. If they are totally unrelated, stop me with an error."_

#### Behaviour
- **Zero Overhead:** Since it happens at compile-time, it generates the exact same machine code as if the types were originally correct. It creates no runtime slowdown.
- **Safety Check:** It checks if a conversion is _sensible_.
    - `int` to `float`? **Yes.**
    - Child* to Base* ? Yes.
    - `std::string` to `int`? **No** (Compile Error).
    - `UnrelatedClassA*` to `UnrelatedClassB*`? **No** (Compile Error).
- Uses **direct-initialization**.

Since static_cast uses direct initialization, any explicit constructors of the target class type will be considered when initializing the temporary object to be returned. We discuss explicit constructors in lesson [14.16 -- Converting constructors and the explicit keyword](https://www.learncpp.com/cpp-tutorial/converting-constructors-and-the-explicit-keyword/)

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

#### B. `dynamic_cast`

- **Primary Intent:** Safe downcasting and cross-casting within polymorphic inheritance hierarchies.
- **Timing:** Evaluated at **runtime**.
- **Requirement:** The base class **must** be polymorphic (contain at least one `virtual` function).
- **Under the Hood:** 
	1. Follows the object's hidden virtual pointer (`vptr`) to its Virtual Table (`V-Table`).
    2. Queries the `std::type_info` pointer situated at a negative offset in the V-Table.
    3. Executes a runtime graph-search across the `type_info` inheritance structure.
    4. If multiple inheritance is involved, it dynamically calculates pointer arithmetic offsets to shift the pointer to the target sub-object block ("Cross-Casting").
- **Behaviour on Failure:** 
	- If casting a **pointer** and the type matches incorrectly: returns `nullptr`.
	- If casting a **reference** and the type matches incorrectly: throws a `std::bad_cast` exception (since references cannot be null).

#### C. `reinterpret_cast`

- **Primary Intent:** Treating a raw sequence of bytes as a completely different type.
- **Timing:** **Compile-time** instruction to the compiler's type checker; generates **no machine code** transformations.
- **Under the Hood:** It forces the compiler to map a variable name to an alternative type layout without changing a single bit in RAM. It tells the type system: _"Ignore what you think this memory is; treat it as Type T from now on."_
- **Common Use Cases:** Low-level hardware programming, writing raw memory buffers to disk, network packet parsing, or interacting with graphics APIs.
- **Safety:** Exceptionally dangerous. The C++ compiler's optimiser assumes you will never do this. It assumes that an `int*` and a `float*` will never point to the exact same physical location in RAM. If you use `reinterpret_cast` to make them point to the same spot, the compiler might delete or reorder your code because it thinks, _"These are two different types, so they must be entirely separate objects. I can safely optimise them independently."_ When they turn out to be the _same_ object, your program logic collapses.

**Use case: Network Programming (Parsing Packets)**

When you receive data over the internet (via a TCP/UDP socket), the network card doesn't know what a C++ `Player` class is. It just hands your program a massive array of raw bytes (`char*` or `uint8_t*`). If you know exactly how the server packed the data, you use `reinterpret_cast` to slap a C++ structure directly on top of those raw network bytes.

```cpp
// 1. A C++ struct (12 bytes total)
struct PlayerPosition {
    float x;
    float y;
    float z;
};

// 2. Data arrives from the internet as raw bytes
void onNetworkDataReceived(char* raw_buffer) {
    
    // We can't use static_cast here. A char* and a PlayerPosition* are completely unrelated.
    // We use reinterpret_cast to tell the compiler: 
    // "Trust me, I know the next 12 bytes in this buffer are actually float coordinates."
    
    PlayerPosition* pos = reinterpret_cast<PlayerPosition*>(raw_buffer);
    cout << "Player moved to X: " << pos->x;
}
```

#### D. `const_cast` 

- **Primary Intent:** Adding or removing `const` or `volatile` qualifiers from a variable.
- **Timing:** **Compile-time** directive. Generates **no machine code**.
- **Under the Hood:** It adjusts the compiler’s internal type-checking flags for that specific pointer or reference.
- **The Fatal Trap:** 
	- If the underlying object was originally declared as non-const, stripping `const` to modify it is safe.
    - If the underlying object was originally instantiated as a **true constant** (e.g., `const int x = 10;`), stripping `const` and attempting to write to that memory triggers immediate **Undefined Behaviour**, typically resulting in a seg fault or silent memory corruption because the OS may place true constants in a read-only memory page.

`const_cast` is all about stripping away read-only protections. Imagine you have a regular, normal integer.

```cpp
int my_score = 100;
```

Now, you pass it to a function that promises not to change it. The function takes a `const int*`. 

```cpp
void printScore(const int* ptr) {
    // I promise to only read it!
}
```

If, inside that function, you desperately need to change the score, you can use `const_cast` to rip off the sticky note.

```cpp
int* mutable_ptr = const_cast<int*>(ptr);
*mutable_ptr = 999; // THIS IS SAFE!
```

**Why is it safe?** Because the underlying, original object (`my_score`) was created as a normal, modifiable integer. The `const` was just a temporary promise.

Now, imagine you create a variable that is meant to be permanent from the moment it is born.

```cpp
const int max_players = 4;
```

Because you declared it `const` right at creation, the Operating System physically locks it away. On many systems, it places `max_players` inside a **Read-Only Memory Page** in your RAM (often called the `.rodata` section).

```cpp
int* cheat_ptr = const_cast<int*>(&max_players); // The compiler allows this
*cheat_ptr = 5; // FATAL CRASH
```

`const_cast` successfully tricked the C++ compiler into letting you write the code. But when the compiled program actually runs, the CPU tries to write the number `5` into the Operating System's physically locked Read-Only RAM. The Operating System detects this illegal hardware action, and immediately kills your program (Segmentation Fault).


**Use Case:**
You use this almost exclusively for one reason: **Legacy Code Integration.**
C++ is built on top of C. There are many C libraries that do not use the `const` keyword, even when they should. 
Imagine you have a `const` string, and you want to pass it into a an old C library that logs messages to a file.

```cpp
// ---------------------------------------------------------
// ANCIENT C LIBRARY (You do not own this code, you cannot edit it)
// The original author didn't use 'const', but we know for a fact 
// this function only reads the text to print it. It doesn't modify it.
void legacy_c_print(char* text) {
    printf("%s\n", text); 
}
// ---------------------------------------------------------


// YOUR MODERN C++ CODE
void myModernSystem() {
    const char* error_msg = "Critical Failure!";
    
    // legacy_c_print(error_msg); 
    // ^ COMPILER ERROR: You cannot pass a 'const char*' into a function asking for 'char*'.
    
    // THE FIX:
    // We rip off the 'const' badge just to make the compiler happy, 
    // because we know the legacy function is actually safe and won't mutate our string.
    legacy_c_print( const_cast<char*>(error_msg) ); 
}
```

If `const_cast` did not exist, you would be forced to create a completely new, mutable copy of the string in RAM just to pass it to the old function, which is a massive waste of memory and CPU time.

---
### The C-Style Algorithm

When you write a C-style cast `(Type)variable`resolves it by trying a strict sequence of C++ explicit casts until one passes compilation:

1. `const_cast`
2. `static_cast`
3. `static_cast` followed by `const_cast`
4. `reinterpret_cast`
5. `reinterpret_cast` followed by `const_cast`
#### Step 1: `const_cast`
- Are they the exact same type, just with a `const` or `volatile` badge in the way?
- **Example:** `const int*` to `int*`.
- **Result if it stops here:** It quietly strips the read-only protection.
#### Step 2: `static_cast`
- Is this a mathematically sound conversion (like `int` to `float`), or a safe navigation of a known inheritance tree (Upcasting) (Not the RTTI Inheritance Tree, just the code bluprint)?
- **Example:** `int` to `float`, or `Dog*` to `Animal*`.
- **Result if it stops here:** Safe. It generates the proper math or pointer offset assembly instructions.
#### Step 3: `static_cast` followed by `const_cast`
- Do I need to do a safe math conversion AND strip a read-only badge at the same time?
- **Example:** `const int` to `float`.
- **Result if it stops here:** Safe, though slightly complex.
#### Step 4: `reinterpret_cast`
- Okay, math conversions completely failed. These two types are fundamentally incompatible. Can I just force the CPU to look at the raw memory bits through a fraudulent lens?"
- **Example:** `int*` to `float*`.
- **Result if it stops here:** **Massive Danger.** The compiler gives up on math and safety. It does not alter the bits; it just lies to the type system.
#### Step 5: `reinterpret_cast` followed by `const_cast`
- The types are incompatible AND there is a read-only lock? Break the lock and lie to the type system simultaneously
- **Example:** `const int*` to `float*`.
- **Result if it stops here:** Absolute maximum danger. You have bypassed every single safeguard in the language.

There is one thing you can do with a C-style cast that you can’t do with C++ casts: 
C-style casts can convert a derived object to a base class that is inaccessible (e.g. because it was privately inherited).

---
### Casting vs initializing a temporary object

Let’s say we have some variable `x` that we need to convert to an `int`. There are two conventional ways we can do this:

1. `static_cast<int>(x)`, which returns a temporary `int` object _direct-initialized_ with `x`.
2. `int { x }`, which creates a temporary `int` object _direct-list-initialized_ with `x`.

We should avoid `int ( x )`, which is a C-style cast. This will return a temporary `int` direct-initialized with the value of `x` (like we’d expect from the syntax), but it also has the other downsides as discussed.

There are (at least) three notable differences between the `static_cast` and the direct-list-initialized temporary:

1. `int { x }` uses list initialization, which disallows narrowing conversions. This is great when initializing a variable, because we rarely intend to lose data in such cases. But when using a cast, it is presumed we know what we’re doing, and if we want to do a cast that might lose some data, we should be able to do that. The narrowing conversion restriction can be an impediment in this case.

```cpp
// We want to do fp division, so one of the operands needs to be a fp type
std::cout << double{x} / y << '\n'; 
// okay if int is 32-bit, narrowing if x is 64-bit
```

2. `static_cast` makes it clearer that we are intending to perform a conversion. Although the `static_cast` is more verbose than the direct-list-initialized alternative, in this case, that’s a good thing, as it makes the conversion easier to spot and search for. That ultimately makes your code safer and easier to understand.

3. Direct-list-initialization of a temporary only allows single-word type names. Due to a weird syntax quirk, there are several places within C++ where only single-word type names are allowed (the C++ standard calls these names “simple type specifiers”). So while `int { x }` is a valid conversion syntax, `unsigned int { x }` is not.

```cpp
	unsigned char c { 'a' };
	std::cout << unsigned int { c } << '\n'; // compile-error
```

Workaround:

```cpp
	unsigned char c { 'a' };
	using uint = unsigned int;
	std::cout << uint { c } << '\n';
```

For all these reasons, we generally prefer `static_cast` over direct-list-initialization of a temporary.

----
