A namespace must be defined either in the global scope, or inside another namespace. They cannot be defined in local scope. 

**Why Use Namespaces?**
- Prevent Collisions: Avoid naming conflicts between your code and libraries.
- Logical Grouping: Organise code into "modules" (e.g., Math::, UI::).
- Explicitness: std::sort tells the reader exactly where the tool came from.

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

It’s legal to declare namespace blocks in multiple locations (either across multiple files, or multiple places within the same file as they are exempt from ODR). All declarations within the namespace are considered part of the namespace.

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

Note that this capability also means you could add your own functionality to the `std` namespace. Though doing so causes undefined behavior most of the time, because the `std` namespace has a special rule prohibiting extension from user code. Therefore do not add custom functionality to the std namespace.

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

Named namespaces have external linkage, i.e, a particular named namespace is same across all the files in the program.

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

 The compiler internally renames the namespace to something unique, like `Namespace_GameCPP.` So, `Game.cpp` compiles a function that looks this: 
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

But now linker won't find `updatePlayer(Namespace_MainCPP::PlayerData)`'s definition and give error. All this is because of internal linkage

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

**When you call a function, the compiler looks in the current scope AND in the namespaces of the function's arguments.** 

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
### Qualified and unqualified names

A **qualified name** is a name that includes an associated scope. A name can be qualified by a class name using the scope resolution operator (::), or by a class object using the member selection operators (. or ->). For example:

```cpp
class C; // some class
C::s_member; // s_member is qualified by class C
obj.x; // x is qualified by class object obj
ptr->y; // y is qualified by pointer to class object ptr
```

An **unqualified name** is a name that does not include a scoping qualifier. For example, `cout` and `x` are unqualified names, as they do not include an associated scope.

---
### Using-declaration

A **using declaration** allows us to use an unqualified name (with no scope) as an alias for a qualified name.

```cpp
#include <iostream>
int main() {
   using std::cout; // this using declaration tells the compiler that cout should resolve to std::cout
   cout << "Hello world!\n"; // so no std:: prefix is needed here!
} // the using declaration expires at the end of the current scope
```

The using-declaration `using std::cout;` tells the compiler that we’re going to be using the object `cout` from the `std` namespace. So whenever it sees `cout`, it will assume that we mean `std::cout`. If there’s a naming conflict between `std::cout` and some other use of `cout` that is visible from within `main()`, `std::cout` will be preferred. 

Note that you will need a separate using-declaration for each name (e.g. one for `std::cout`, one for `std::cin`, etc…).

The using-declaration is active from the point of declaration to the end of the scope in which it is declared.

---
### Using-directives

A **using directive** allows _all_ identifiers in a given namespace to be referenced without qualification from the scope of the using-directive.

For technical reasons, using-directives do not actually introduce new meanings for names into the current scope, instead they introduce new meanings for names into an outer scope (more details about which outer scope is picked can be found [here](https://quuxplusone.github.io/blog/2020/12/21/using-directive/)).

```cpp
#include <iostream>
int main() {
   using namespace std; // all names from std namespace now accessible without qualification
   cout << "Hello world!\n"; // so no std:: prefix is needed here
} // the using-directive ends at the end of the current scope
```

The using-directive `using namespace std;` tells the compiler that all of the names in the `std` namespace should be accessible without qualification in the current scope. When we then use unqualified identifier `cout`, it will resolve to `std::cout`.

Using-directives are the solution that was provided for old pre-namespace codebases that used unqualified names for standard library functionality.  Problems with using-directives:
1. Using-directives allow unqualified access to _all_ of the names from a namespace (potentially including lots of names you’ll never use).
2. Using-directives do not prefer names from the namespace identified by the using-directive over other names (this behaviour is unlike using-declaration).


If a using-declaration or using-directive is used within a block, the names are applicable to just that block (it follows normal block scoping rules). This is a good thing, as it reduces the chances for naming collisions to occur to just within that block.

If a using-declaration or using-directive is used in a namespace (including the global namespace), the names are applicable to the entire rest of the file for global namespace and the namespace end bracket for other namespaces .


Using-statements should not be placed anywhere where they might have an impact on code in a different file. Nor should they be placed anywhere where another file’s code might be able to impact them.

For example, if you placed a using-statement in the global namespace of a header file, then every other file that `#included` that header would also get that using-statement. That’s clearly bad. This also applies to namespaces inside header files, as some other file may open the namespace again (after `#including` it) to extend it's functionality, then they will have to deal with pollution.

An example for showing issue with ordering:

FooInt.h:

```cpp
namespace Foo {
    void print(int) { cout << "print(int)\n" << endl; }
}

```

FooDouble.h:

```cpp
namespace Foo {
    void print(double) { cout << "print(double)\n" << endl; }
}
```

main.cpp (okay):

```cpp
#include <iostream>
#include "FooDouble.h"
#include "FooInt.h"

using Foo::print; // print means Foo::print

int main() {
    print(5);  // Calls Foo::print(int)
}
```

main.cpp (bad):

```cpp
#include <iostream>
#include "FooDouble.h"
using Foo::print; 
#include "FooInt.h"

int main() {
    print(5);  // Calls Foo::print(double)
}
```

In the working version, when the compiler encounters `using Foo::print`, it has already seen both `Foo::print(int)` and `Foo::print(double)`, so it makes both available to be called as just `print()`. Since `Foo::print(int)` is a better match than `Foo::print(double)`, it calls `Foo::print(int)`.

In the bad version, when the compiler encounters `using Foo::print`, it has only seen a declaration for `Foo::print(double)`, so it only makes `Foo::print(double)` available to be called unqualified. So when we call `print(5)` only `Foo::print(double)` is even eligible to be matched. Thus `Foo::print(double)` is the one that gets called!

So never put using-statements before `#include` directives and don't use them in header files.


Once a using-statement has been declared, there’s no way to cancel or replace it with a different using-statement within the scope in which it was declared.

```cpp
int main() {
    using namespace Foo;
    // there's no way to cancel the "using namespace Foo" here!
    // there's also no way to replace "using namespace Foo" with a different using statement
} // using namespace Foo ends here
```

The best you can do is this

```cpp
int main() {
    {
        using namespace Foo;
        // 
    } 
    {
        using namespace Goo;
        //
    } 
}
```


Avoid using-directives altogether (except `using namespace std::literals` to access the `s` and `sv` literal suffixes). Using-declarations are okay in .cpp files, after the `#include` directives.

----

