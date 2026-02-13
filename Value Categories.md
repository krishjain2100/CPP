
**References also exist for functions.**

---
### 1. Old View

Earlier we had only two categories:

1. **L-value (Left Value):** Has a name, has an address in memory. You can take its address (`&obj`). _Examples:_ `int x;`, `x`, `arr[0]`, `*ptr`.

2. **R-value (Right Value):** A temporary value. Does not have a name. You generally cannot take its address. _Examples:_ `10`, `"hello"`, `x + y`, `functionReturningInt()`.

```cpp
int x = 10; 
// &x;      // OK -> x is lvalue
// &10;     // ERROR -> 10 is rvalue
// &(x+1);  // ERROR -> (x+1) is rvalue
```

---
### 2. Modern View

To make Move Semantics work,  a "middle ground." had to be invented. The standard actually defines **three** main categories (plus two composite ones).
#### A. **lvalue** (Locator Value)
- **Identity:** Has a name.
- **Movable:** **NO.** (You generally want to copy it because someone else might be using it).
- _Example:_ `std::string s = "hello";` (`s` is an lvalue).

#### B. **prvalue** (Pure R-value)
- **Identity:** No name (Temporary).
- **Movable:** **YES.** (It's about to die anyway, so stealing its resources is safe).
- _Example:_ `std::string("hello")` or `10`.

#### C. **xvalue** (eXpiring Value) 
This is the special category for "an lvalue that we decided to treat as an rvalue."
- **Identity:** Has a name (usually).
- **Movable:** **YES.**
- _How to make one:_ By using `std::move()`.

### D. **glvalue** (Generalised lvalue)
	= lvalue OR xvalue. (Things with an identity).

### E. **rvalue** 
	= prvalue OR xvalue. (Things you can move from).

---
### 3. R-value References (`&&`)

To distinguish between "Copy me" and "Move me" functions, C++ introduced the double ampersand `&&`.
- `void func(int& x)` accepts **lvalues only**.
- `void func(int&& x)` accepts **rvalues only**.
- `void func(const int& x)` accepts **everything** (but is read-only).

```cpp
void process(string& s)  { cout << "L-value ref (Copy)"; }
void process(string&& s) { cout << "R-value ref (Move)"; }

int main() {
    string s = "Hello";
    process(s); // s is lvalue -> Calls "L-value ref"
    process("World"); // "World" is prvalue -> Calls "R-value ref"
    process(std::move(s)); // std::move(s) converts lvalue to xvalue -> Calls "R-value ref"
}
```

---
