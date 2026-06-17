A namespace must be defined either in the global scope, or inside another namespace

**Why Use Namespaces?**
- Prevent Collisions: Avoid naming conflicts between your code and libraries.
- Logical Grouping: Organise code into "modules" (e.g., Math::, UI::).
- Explicitness: std::sort tells the reader exactly where the tool came from.

---
### Defining your own namespaces

C++ allows us to define our own namespaces via the `namespace` keyword. Namespaces that you create in your own programs are casually called **user-defined namespaces**

Syntax:
```
namespace NamespaceIdentifier
{
    // content of namespace here
}
```

To access the contents of a namepsace, we use [[Scope Resolution]]

---
### Identifier resolution from within a namespace

If an identifier inside a namespace is used and no scope resolution is provided, the compiler will first try to find a matching declaration in that same namespace. If no matching identifier is found, the compiler will then check each containing namespace in sequence to see if a match is found, with the global namespace being checked last.

```cpp
void print() { std::cout << " there\n"; } // global namespace
namespace Foo {
	void print() { std::cout << "Hello"; } //  Foo namespace
	void printHelloThere() {
		print();   // calls print() in Foo namespace
		::print(); // calls print() in global namespace
	}
}
int main() {
	Foo::printHelloThere();
}
// Output:
// Hello there
```

---
### Multiple namespace blocks are allowed

It’s legal to declare namespace blocks in multiple locations (either across multiple files, or multiple places within the same file). All declarations within the namespace are considered part of the namespace.

circle.h:
```cpp
#ifndef CIRCLE_H
#define CIRCLE_H
namespace BasicMath {
    constexpr double pi{ 3.14 };
}
#endif
```

growth.h:

```cpp
#ifndef GROWTH_H
#define GROWTH_H
namespace BasicMath {
    constexpr double e{ 2.7 };
}
#endif
```

main.cpp:
```cpp
#include "circle.h" // for BasicMath::pi
#include "growth.h" // for BasicMath::e
int main() {
    std::cout << BasicMath::pi << '\n';
    std::cout << BasicMath::e << '\n';
} // This works
```


The standard library makes extensive use of this feature, as each standard library header file contains its declarations inside a `namespace std` block contained within that header file. Otherwise the entire standard library would have to be defined in a single header file!

Note that this capability also means you could add your own functionality to the `std` namespace. Doing so causes undefined behavior most of the time, because the `std` namespace has a special rule prohibiting extension from user code. Therefore do not add custom functionality to the std namespace.

---
### Nested namespaces

Namespaces can be nested inside other namespaces. For example:

```cpp
namespace Foo {
    namespace Goo  {
        int add(int x, int y) {
            return x + y;
        }
    }
}
```

Note that because namespace `Goo` is inside of namespace `Foo`, we access `add` as `Foo::Goo::add`.

Since C++17, nested namespaces can also be declared this way:
```cpp
namespace Foo::Goo  {
    int add(int x, int y) {
        return x + y;
    }
}
```

This is equivalent to the prior example.

If you later need to add declarations to the `Foo` namespace (only), you can define a separate `Foo` namespace to do so:

```cpp

namespace Foo::Goo  {
    int add(int x, int y) {
        return x + y;
    }
}

namespace Foo {
     void someFcn() {} // This function is in Foo only
}
```

---
### Namespace aliases

Because typing the qualified name of a variable or function inside a nested namespace can be painful, C++ allows you to create **namespace aliases**, which allow us to temporarily shorten a long sequence of namespaces into something shorter:

```cpp
#include <iostream>

namespace Foo::Goo {
    int add(int x, int y) {
        return x + y;
    }
}

int main() {
    namespace Active = Foo::Goo; // active now refers to Foo::Goo
    std::cout << Active::add(1, 2) << '\n'; 
} // The Active alias ends here
```

---
### Anonymous Namespaces 

`static` global variables make a variable invisible outside the current file. In modern C++, **Anonymous Namespaces** are the preferred way to do this. It is effectively `static` but works for classes and structs too.

```cpp
// File: helper.cpp
// OLD WAY
static void internalHelper() { ... }

// NEW WAY
namespace {
    void internalHelper() { 
        // Only visible in helper.cpp
        // Invisible to the linker elsewhere
    }
}
```

---
### Argument Dependent Lookup (ADL)

_**When you call a function, the compiler looks in the current scope AND in the namespaces of the function's arguments.** 

This is why you can write `std::cout << x;` instead of `std::operator<<(std::cout, x);`.
Sequence of what compiler sees and understands: 
- `std::cout << x;`
- `operator<<(std::cout, x);  
- `std::operator<<(std::cout, x);`

```cpp
namespace Game {
    struct Player {};    
    void attack(Player p) { std::cout << "Attack!"; }
}

int main() {
    Game::Player p;
    // We didn't write Game::attack(p).
    // The compiler found 'attack' inside the 'Game' namespace
    // automatically because the argument 'p' belongs to 'Game'.
    attack(p); 
}
```

---
### `using`

`using namespace std;` is generally bad in global scope, but there are things to look out for.

- **Bad:** `using namespace std;` (Global scope). Pollution.
- **Good:** `using std::cout;` (Specific). Only imports what you need.
- **Very Bad:** Putting `using namespace ...` inside a **Header File (`.h`)**.
	If you do this in `myHeader.h`, any file which does `#include "myHeader.h"` gets their code polluted with that namespace.

---
