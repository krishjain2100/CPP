A namespace must be defined either in the global scope, or inside another namespace. They cannot be defined in local scope. 

**Why Use Namespaces?**
- Prevent Collisions: Avoid naming conflicts between your code and libraries.
- Logical Grouping: Organise code into "modules" (e.g., Math::, UI::).
- Explicitness: std::sort tells the reader exactly where the tool came from.

---
### Defining your own namespaces

C++ allows us to define our own namespaces via the `namespace` keyword. Namespaces that you create in your own programs are casually called **user-defined namespaces**
`namespace NamespaceIdentifier { // content }`. To access the contents of a namespace, we use [[Scope Resolution]]

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

All content declared in an unnamed namespace is treated as if it is part of the parent namespace. This might make unnamed namespaces seem useless. But unnamed namespaces gives it's content internal linkage. For functions, this is effectively the same as defining all functions as static functions. 

Unnamed namespaces will also keep **classes/structs** local to the translation unit, something for which there is no alternative equivalent mechanism to do.

Unnamed namespaces should generally not be used in header files. First obvious reason is binary bloat. Second:

```cpp
// common.h
namespace {
    struct PlayerData {
        int health;
    };
}

// Function declaration meant to be shared
void updatePlayer(PlayerData p); 
```

Now you include this header in `Game.cpp` to write the actual function.

```cpp
// Game.cpp
#include "common.h"

void updatePlayer(PlayerData p) {
    p.health = 100;
}
```

 The compiler internally renames the namespace to something unique, like `Namespace_GameCPP.` So, `Game.cpp` compiles a fn that looks this: 
 `void updatePlayer(Namespace_GameCPP::PlayerData p)`

Now you include the header in `main.cpp` because you want to use the struct and call that function.

```cpp
// main.cpp
#include "common.h"

int main() {
    PlayerData myPlayer;
    updatePlayer(myPlayer);
}
```

The compiler renames the namespace in this file to something unique, like `Namespace_MainCPP`. So, `main.cpp` creates a variable of type `Namespace_MainCPP::PlayerData`. It then tries to call `updatePlayer` passing that exact type.

But now linker won't find `updatePlayer(Namespace_MainCPP::PlayerData)`'s definition and give error

---
### Inline namespaces

```cpp
void doSomething() { std::cout << "v1\n"; }
int main() {
    doSomething();
}
```

Let's say you want to improve `doSomething()` some way that changes how it behaves. But if you do this, you risk breaking existing programs using the older version. 

One way would be to create a new version of the function with a different name. But over the course of many changes, you could end up with a whole set of almost-identically named functions (`doSomething`, `doSomething_v2`, `doSomething_v3`, etc…).

A better way is to use an inline namespace. An **inline namespace** is a namespace in which anything declared inside an inline namespace is also considered part of the parent namespace. However, unlike unnamed namespaces, inline namespaces don’t affect linkage.


```cpp
inline namespace V1 {
    void doSomething() { std::cout << "V1\n"; }
}

namespace V2 {
    void doSomething() {
        std::cout << "V2\n";
    }
}

int main() {
    V1::doSomething(); // calls the V1 version of doSomething()
    V2::doSomething(); // calls the V2 version of doSomething()
    doSomething(); // calls the inline version of doSomething() (which is V1)
}

// Output
// V1
// V2
// V1
```

In the above example, callers to `doSomething()` will get the V1 (the inline version) of `doSomething()`. Callers who want to use the newer version can explicitly call `V2::doSomething()`. This preserves the function of existing programs while allowing newer programs to take advantage of newer/better variations.

Now, if you want to push the newer version:

```cpp
namespace V1 {
    void doSomething() { std::cout << "V1\n"; }
}

inline namespace V2 {
    void doSomething() { std::cout << "V2\n"; }
}

int main() {
    V1::doSomething(); // calls the V1 version of doSomething()
    V2::doSomething(); // calls the V2 version of doSomething()
    doSomething(); // calls the inline version of doSomething() (which is V2)
}

// Output
// V1
// V2
// V2
```

If you make both inline, then any call to `doSomething()` would be flagged as ambiguous by the compiler.

---
### Mixing inline and unnamed namespaces 

A namespace can be both inline and unnamed:

```cpp
namespace V1 {
    void doSomething() { std::cout << "V1\n"; }
}

inline namespace {
    void doSomething() { std::cout << "V2\n"; }
}

int main() {
    V1::doSomething(); // calls the V1 version of doSomething()
    doSomething(); 
    // calls the inline version of doSomething() (which is the anonymous one)
}
```

However, in such cases, it’s probably better to nest an anonymous namespace inside an inline namespace. This has the same effect (all functions inside the anonymous namespace have internal linkage by default) but still gives you an explicit namespace name you can use:

```cpp
namespace V1 {
    void doSomething() { std::cout << "V1\n"; }
}

inline namespace V2  {
    namespace  {
        void doSomething() { std::cout << "V2\n"; }
    }
}

int main() {
    V1::doSomething(); // calls the V1 version of doSomething()
    V2::doSomething(); // calls the V2 version of doSomething()
    doSomething(); // calls the inline version of doSomething() (which is V2)
}
```

---
### Argument Dependent Lookup (ADL)

_**When you call a function, the compiler looks in the current scope AND in the namespaces of the function's arguments.** 

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

This is why you can write `std::cout << x;` instead of `std::operator<<(std::cout, x);`.
Sequence of what compiler sees and understands: 
- `std::cout << x;`
- `operator<<(std::cout, x);  
- `std::operator<<(std::cout, x);`

---
### `using`

`using namespace std;` is generally bad in global scope, but there are things to look out for.

- **Bad:** `using namespace std;` (Global scope). Pollution.
- **Good:** `using std::cout;` (Specific). Only imports what you need.
- **Very Bad:** Putting `using namespace ...` inside a **Header File (`.h`)**.
	If you do this in `myHeader.h`, any file which does `#include "myHeader.h"` gets their code polluted with that namespace.

---
