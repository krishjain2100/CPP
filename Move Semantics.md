
We know `&&` catches temporary things. This allows us to write a **Move Constructor**.

Consider you have a `Vector` class that manages a raw pointer to a huge array.

**A. The Copy Constructor (Deep Copy - Slow)**

```cpp
Vector(const Vector& other) {
    // 1. Allocate NEW memory
    this->data = new int[other.size];
    // 2. Copy values one by one
    std::copy(other.data, other.data + other.size, this->data);
}
```

**B. The Move Constructor (Pointer Swap - Fast)** This constructor takes `Vector&& other`. It knows `other` is about to be destroyed (or is an xvalue we authoriaed to die), so we can rob it.

```cpp
// Note: NOT const, because we modify 'other'
Vector(Vector&& other) noexcept {
    // 1. STEAL the pointer directly
    this->data = other.data; 
    this->size = other.size;
    
    // 2. NULL out the victim
    // Crucial! If we don't do this, 'other's destructor 
    // will delete the memory we just stole.
    other.data = nullptr; 
    other.size = 0;
}
```

**Why `noexcept`?** Move constructors should almost always be `noexcept`. If a move fails (throws) while a `std::vector` is resizing, the vector might be left with half-moved objects, corrupting data. If you don't mark it `noexcept`, `std::vector` will often refuse to use your move constructor and Copy instead (for safety).

---

### 5. `std::move` 
As mentioned before, `std::move` does **not** move. It is literally just a cast.

```cpp
template <typename T>
typename remove_reference<T>::type&& move(T&& t) {
    return static_cast<typename remove_reference<T>::type&&>(t);
}
```

- It takes a reference `t`
- It casts it to `T&&` (r-value reference).
- It returns that reference.

```cpp
string a = "hello";
string b = a;  // Calls Copy Constructor (a is lvalue)
string c = std::move(a); // Calls Move Constructor (a becomes xvalue)

// CAREFUL: 'a' is now in a "valid but unspecified" state.
// It is likely empty, but don't count on it containing "null" or anything specific.
// You can assign to it: a = "new string"; (Safe)
// You should not read it: cout << a; (Unsafe/Undefined Logic)
```

### 6. Transferring ownership

```cpp
string s1 = "long string 1", s2 = "long string 2"
string s3 = s1 + s2; 
```

This evaluates s1 + s2, then calls the copy string constructor with this value, which is expensive.
What we would instead want is to directly want s3 to be this calcuated value. Ex:

```cpp
string myString = "hello I am krish"
string newString = move(myString);
cout << myString << endl;    // (empty) // complete transfer of ownership, but it still exists
cout << newString << endl;  // hello I am krish // this takes the string without making any copies.

//This is equivalent to, 
string newString = static_cast<string&&>(myString);   // (weird af)
```
