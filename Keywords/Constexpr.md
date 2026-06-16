
. `constexpr`

**Why it exists:**
- Replace template meta-programming  hacks
- Improve readability and error messages
- Enable zero-overhead abstractions

This allows the compiler to optimise code by calculating values before the program even runs.

- If you feed them **Constants** (like `5` or `const int`), they run at **Compile Time**.
- If you feed them **Variables** (like `x`), they run at **Runtime** just like a normal function.

```cpp
// 1. Compile-time function
constexpr int square(int x) {
    return x * x;
}

int main() {
    // Calculated at compile time. 
    // The compiled binary effectively just says "arr[25]".
    int arr[square(5)]; 
    // C-Style Arrays are strict:
    // The size of a raw array (`int arr[SIZE]`) must be known before the program ever runs.
}
```

#### Creating Types with `constexpr`

Writing code that writes code. Specifically, using numbers to change **Types**.

**1. Compile-Time Dimensions:** 

```cpp
using Matrix = std::array<int, square(N)>;
```

If `N=4`, `square(N)` is 16. The compiler creates a `std::array<int, 16>`. This is much faster than `std::vector` because it lives on the Stack, not the Heap.

**2. Branching Types (`std::conditional`):** This is a very cool optimisation technique. You can choose different variable types based on a number size.
- **Scenario:** You have a number `N`.
- If `N < 8`, it fits in a small byte (`uint8_t`).
- If `N >= 8`, you need a big integer (`uint64_t`).

```cpp
template <size_t N>
struct Storage {
    // std::conditional is a compile-time "if statement" for TYPES.
    // Format: conditional< Boolean, TypeIfTrue, TypeIfFalse >
    using type = std::conditional_t<
        (N < 8), // Condition
        uint8_t, // Result if True
        uint64_t, // Result if False
    >;
};
```

- **Why use this?** If you are writing a high-frequency trading platform or a game engine, you don't want to waste 64 bits of memory to store the number "5". This automates the memory optimisation for you.


There is more to this (added soon)