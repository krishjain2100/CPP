**Why Use Namespaces?**
- Prevent Collisions: Avoid naming conflicts between your code and libraries.
- Logical Grouping: Organise code into "modules" (e.g., Math::, UI::).
- Explicitness: std::sort tells the reader exactly where the tool came from.

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

### Namespace Aliases

Sometimes namespaces get deeply nested or have long names (especially in libraries). You can create a nickname.

```cpp
#include <filesystem>
std::filesystem::path p = "C:/User/Docs";
// Create an alias
namespace fs = std::filesystem;
// Clean and readable
fs::path p2 = "C:/User/Docs";
```

### Argument Dependent Lookup (ADL)

_**When you call a function, the compiler looks in the current scope AND in the namespaces of the function's arguments.** 

This is why you can write `std::cout << x;` instead of `std::operator<<(std::cout, x);`.
Sequence of what compiler sees and understands: 
- `std :: cout << x;`
- `operator<<(std :: cout, x);  
- `std :: operator << (std :: cout, x);`

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

### 4. `using` - The Good, The Bad, and The Ugly

`using namespace std;` is generally bad in global scope, but there are nuances.
- **Bad:** `using namespace std;` (Global scope). Pollution.
- **Good:** `using std::cout;` (Specific). Only imports what you need.
- **Very Bad:** Putting `using namespace ...` inside a **Header File (`.h`)**.
	- If you do this in `myHeader.h`, any file which does `#include "myHeader.h"` gets their code polluted with that namespace.