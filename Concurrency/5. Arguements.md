When you spawn a thread, it gets its own isolated memory stack. Moving data from the main thread's stack into the background thread's stack has strict rules to prevent memory corruption.

### 1. Pass by Value (Deep Copy)

By default, when you pass an argument to a `std::thread` or `std::jthread`, the standard library **copies** the data into the new thread's internal storage.


```cpp
void printNumber(int x) {
    x++; // Modifies the thread's local copy
    std::cout << x << "\n";
}

int main() {
    int myNum = 10;
    std::jthread t(printNumber, myNum); // myNum is COPIED
    
    // myNum is still 10 in main
}
```

 If `main` were to exit or the variable went out of scope while the thread was still running, a reference would point to dead memory. Copying guarantees the thread owns its data.

---

### 2. Pass by Reference (`std::ref`)

If you try to write a function that takes a reference `&`, the compiler will throw a massive error and refuse to compile.

```cpp
void modifyData(int& data) {
    data += 100;
}

int main() {
    int counter = 0;
    // std::jthread t(modifyData, counter); // COMPILE ERROR!
}
```

C++ outright blocks this because it assumes you are making a mistake. To bypass this safety check, you must use a wrapper called **`std::ref`** (or `std::cref` for const references).


```cpp
#include <functional> // required for std::ref

int main() {
    int counter = 0;
    // std::ref tells the compiler: "I know what I am doing."
    std::jthread t(modifyData, std::ref(counter)); 
}
```

**The Severe Warning:** When you use `std::ref`, you take 100% responsibility for the variable's lifetime. If `counter` is destroyed before the thread finishes, the thread will write to corrupted memory. (Because you are using `jthread`, the auto-join protects you here, but with `detach()`, this is a fatal bug).

---

### 3. Move Semantics (`std::move`)

Sometimes, you have a massive object (like a `std::vector<int>` with 1 million elements) or an object that _cannot_ be copied (like a `std::unique_ptr`). You want to transfer **ownership** of that object entirely into the new thread.

You use `std::move` to cast the variable to an xvalue, allowing the thread's internal machinery to steal the pointer.


```cpp
#include <memory>

// Takes unique_ptr by value (takes ownership)
void processFile(std::unique_ptr<int> filePtr) {
    std::cout << "Processing: " << *filePtr << "\n";
    // When this function ends, the thread destroys the memory safely.
}

int main() {
    auto myPtr = std::make_unique<int>(42);

    // std::jthread t(processFile, myPtr); // COMPILE ERROR: Cannot copy unique_ptr

    // SUCCESS: We transfer ownership. 
    std::jthread t(processFile, std::move(myPtr));

    // myPtr is now nullptr in main. The thread owns it entirely.
}
```

This is the most efficient and safe way to pass heavy data to a thread. The thread gets exclusive ownership, so there is no risk of data races or lifetime issues.

---

### 4. How many threads should I make?

Spawning 1,000 threads on a 4-core CPU is disastrous. The OS will spend more time context-switching (swapping threads in and out of the CPU) than actually executing your code.


```cpp
unsigned int cores = std::thread::hardware_concurrency();
std::cout << "This machine supports " << cores << " concurrent threads.\n";
```

- If the function returns `8`, you should generally spawn exactly 8 worker threads and divide your data among them.
- _Note:_ It can return `0` if the OS refuses to share that information, so always handle that edge case (usually by defaulting to 2 or 4).

---
