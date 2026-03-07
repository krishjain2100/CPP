### Related
- [[Makefile]]
- [[Shell Scripts]]
- [[CMake]]
- [[Noexcept]]

Using g++ tells the computer to compile the files and link them to generate a final executable called a.out (unless you use -o flag to give a name to this executable).

Loading is handled by the OS after you type ./a.out  and hit enter to run the executable.

### The `-c` Flag (Compile Only)

* **Action:** Compiles the `.cpp` source file into a machine-code **object file** (`.o`).
* **Crucial for:** It instructs the compiler to stop *before* the linking stage.
* If not used, then the final executable is produced after linking
  
```bash
	g++ -c main.cpp -o main.o
```

### The `-o` Flag (Output Name)

- **Action:** Specifies the exact name of the generated file.
- **Default Behaviour:** If omitted during the final linking stage, `g++` defaults to naming the executable `a.out`.
```bash
	g++ main.o utils.o math.o -o program
```

* **Usage:** You can tell the compiler exactly _where_ to put the file. For eg, 
```bash
	g++ -c src/main.cpp -o build/obj/main.o` 
```

---
### The Default Behaviour of `-c` Without `-o`

If you run `g++ -c main.cpp` exactly like that, the compiler will successfully create an object file and automatically name it **`main.o`**. It simply takes the name of your source file (`main`), strips off the `.cpp` extension, and replaces it with `.o`.
##### Compiling Default vs. Linking Default

- **During Compiling (`-c`):** Omit `-o` $\rightarrow$ You get `[filename].o` (e.g., `main.o`).
- **During Linking (No `-c`):** Omit `-o` $\rightarrow$ You get the generic `a.out` file.

---
### The Standard Flag: `-std=`

- **`-std=c++17`**: Forces the compiler to use the C++17 standard.
- If you try to use modern features (like `std::optional`) without setting this, the compiler might default to an older standard like C++11 and throw errors.
---
### Warning Flags

- **`-Wall`**: Enables all the most common warning messages. (Ironically, it doesn't actually enable _all_ warnings, just the universally agreed-upon ones).
  
- **`-Wextra`**: Enables additional warnings that `-Wall` misses, such as warning you if you pass a variable into a function but never actually use it.
  
- **`-pedantic`**: Strictly enforces the official ISO C++ standard. It will give error if you use compiler-specific extensions that might work on your machine but break on someone else's.

- **`-Werror`**: Turns every single warning into a fatal compilation error.

---
### Optimisation Flags

When you compile code, the compiler can actively rewrite your logic behind the scenes to make it run faster.

- **`-O0`** (Default): Zero optimisation. Compile times are extremely fast, but the resulting program runs slowly. Best used while actively writing and debugging code.

- **`-O2`**: Moderate optimisation. It performs nearly all supported optimisations that do not involve a space-speed tradeoff. This is the standard for release builds.

- **`-O3`**: Aggressive optimisation. It attempts things like loop unrolling and auto-vectorisation. It can make your program blazingly fast, but it makes the executable file larger and compilation takes much longer.

- **`-march=native`**: When building high-performance systems (like low-latency trading engines), this is critical. It tells the compiler to look at the exact CPU architecture you are currently compiling on and utilise every single hardware-specific instruction available to squeeze out micro-seconds of performance.

---
### Debugging Flags: `-g` and Sanitizers

- **`-g`**: Tells the compiler to generate "debug symbols" and embed them into the executable. If your program segfaults, a debugger (like `gdb`) needs these symbols to tell you exactly which line of code caused the crash. Without `-g`, the debugger just shows you raw, unreadable memory addresses.

- **`-fsanitize=address`**: Specifically instruments your code to catch memory corruption, out-of-bounds array accesses, and use-after-free bugs at runtime.

#### What Are Debug Symbols?

Debug symbols are a massive translation dictionary embedded inside your compiled program. They map the raw, unreadable machine code that the CPU executes back to the human-readable C++ source code.

1. When you compile a program normally (without the `-g` flag), the compiler strips away all human context to make the file as small and fast as possible.
	- The variable `int loop_counter` becomes a raw memory address like `[RBP-0x4]`.
	- The function `calculate_moving_average()` becomes a hex address like `0x4011a6`.
	- All line numbers and file names are completely deleted.
	
	On crash, a debugger like `gdb`, can only tell : _"Segmentation fault at address 0x4011a6."_. You have no idea what that means or where it is in your code.

2. When you compile with `-g`, the compiler generates a specialised table (usually in a format called **DWARF** on Linux) and attaches it to the `.o` or executable file. This table contains:
	
	- **Line Number Mapping:** "Machine instruction `0x4011a6` corresponds exactly to line 42 in `math.cpp`."
	- **Variable Names & Types:** "Memory address `[RBP-0x4]` is a 32-bit signed integer named `loop_counter`."
	- **Function Signatures:** "Address `0x401200` is the start of `void processData(int sz)"

	On crash, `gdb` reads the symbol table and tells:  _"Segmentation fault in math.cpp at line 42, inside calculate_moving_average(). The value of loop_counter was 10."_

#### Why Not Leave Them On All the Time?

1.  **File Size:** Debug symbols are absolutely massive. A 5 MB executable can easily balloon to 50 MB or 100+ MB when compiled with `-g`.
2. **Security/Reverse Engineering:** If you ship a program with debug symbols to a customer, you are handing them the exact blueprint of your entire source code. Anyone can open the executable and see exactly how your proprietary algorithms work, variable by variable.    

#### Symbol Stripping

You want debugger as well as small files The industry solution is **Symbol Stripping**.

1. You compile the production program _with_ debug symbols (`-g`).
   
2. You run a tool (like `strip` or `objcopy` on Linux) that rips the DWARF symbol table out of the executable and saves it as a totally separate file (like `program.debug`).
   
3. You deploy the tiny, fast, stripped executable to your servers.
   
4. If the server crashes, it generates a **Core Dump** (a snapshot of the exact state of the RAM at the moment of the crash). You take that core dump back to your local machine, load it into `gdb` alongside your separate `program.debug` file, and perfectly reconstruct the crash line-by-line without ever slowing down the production server.

---
### The Preprocessor Flag: `-D`

When you run `g++`, the  first thing that happens (before any actual C++ code is compiled) is preprocessing (literal text-replacement). The preprocessor handles anything that starts with a `#` (like `#include` or `#define`).

Eg:

```cpp
#include <iostream>
int main() {
    int trade_volume = 5000;
#ifdef DEBUG
    std::cout << "[LOG] Processing volume: " << trade_volume << "\n";
#endif
    return 0;
}
```

If you compile this normally (`g++ main.cpp -o program`), the preprocessor looks at `#ifdef DEBUG`, sees that `DEBUG` hasn't been defined anywhere, and literally **deletes** the `std::cout` line from the code before handing it to the compiler.

```bash
g++ -DDEBUG main.cpp -o program
```

This is equivalent to `#define DEBUG` at the very top of `main.cpp`. 
Now, the preprocessor leaves the `std::cout` line in, and your program prints the log.

You could also do multiple at once: 

```bash
g++ -DDEBUG -DLINUX_ENV -DUSE_NEW_API main.cpp -o program
```

If you want to set a macro to a specific value, you use an equals sign `=`: 

```bash
g++ -DMAX_CONNECTIONS=500 main.cpp -o program
```

In high-frequency trading, evaluating an `if (is_debug_mode)` statement at runtime costs precious CPU cycles (due to branching). By using preprocessor macros and the `-D` flag, you completely eradicate the logging code from the final binary in your Release build. The CPU doesn't have to evaluate an `if` statement because the code literally doesn't exist anymore.

---
### Include Directories Flag: `-I`

Your code is organised into folders. Your `.cpp` files live in a `src/` folder , and your header files live in a completely separate `include/` folder.

If `src/main.cpp` says `#include math.h`, it only looks in the current folder by default and crashes. To fix this, you use the `-I` (Include) flag to give the compiler a map.

```bash
g++ -I./include src/main.cpp -o program
```

This tells `g++`: 
If you can't find a header file, go look inside the `./include` directory before giving up.

---
### The `-l` and `-L` Flags (Linking External Libraries)

There is a massive difference between a **Header** (`.h`) and a **Library Binary** (`.so`, `.dll`, `.a`). 
* The header file (found via `-I`) tells the compiler that these functions exist
* The library file contains the actual compiled machine code for those math functions. 

If you use `-I` but forget to link the actual library, the compiler will succeed (`-c`), but the Linker will fail with a massive `undefined reference` error because it can't find the actual implementation.

---
### Link Library Flag: `-l`

This flag tells the linker to stitch a specific pre-compiled library into your executable.

* **Syntax Rule:** If a library file is named `libmath.so`, you drop the `lib` and the `.so`, and just write `-lmath` or `-lm`.
* **Example:** The standard math library (used for `sqrt` or `sin` in C) requires linking:

```bash 
	g++ main.o utils.o -o program -lm
```

- **Example:** If you are writing multi-threaded code using `<thread>`, you must link the POSIX thread library:

```bash
	g++ main.cpp -o program -lpthread
```

---
### Library Directory Flag: `-L`

Just like `-I` tells the compiler where to search for hidden _headers_, `-L` tells the linker where to search for hidden _library binaries_ if they aren't installed in the standard system folders (like `/usr/lib`).

```bash
g++ main.cpp -o program -L./external_libs -lcustom_math
```

1. **`-L./external_libs`**: Look inside this custom folder.
2. **`-lcustom_math`**: Find the file named `libcustom_math.so` and link it.

---
