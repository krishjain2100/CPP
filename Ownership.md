
**Ownership = responsibility for the resource’s lifetime.**

If some object “owns” a resource (usually heap memory), it is responsible for freeing it when appropriate (e.g., calling delete, or letting a smart pointer’s destructor free it).
  
**Ownership determines:**
a) Who destroys the object
b) When it is destroyed (and thus when references to it become invalid/dangling).
c) How many owners are allowed (exclusive vs shared).

### Example 1: **Reference does NOT own lifetime**

```cpp
struct X {
    X()  { cout << "X constructed\n"; }
    ~X() { cout << "X destroyed\n"; }
};

int main() {
    X& r = *new X;   
    cout << "Inside main\n";
}
//Output
//X constructed 
//Inside main
```

**What did NOT happen**
- `X destroyed` is never printed
- No automatic cleanup
- No destructor call via reference
**Why**
- `new X` creates an object
- **Nothing deletes it**
- `r` going out of scope does **nothing**
- Reference ≠ owner

**References do not manage lifetime**

---
### Example 2: **Pointer does NOT own lifetime either**
```cpp
int main() {
    X* p = new X;
    X* q = p;  
}
// Output
// X constructed
```

Again:
- No destructor
- No delete
- Two pointers ≠ two owners

Pointers are just addresses.

---
### Example 3: **Object owns its own lifetime (stack)**
```cpp
int main() {
    X x;
    {
        X& r = x;
        X* p = &x;
    }
    cout << "Still alive\n";
}
// Output
// X constructed 
// Still alive 
// X destroyed
```

**Key observations**
- `r` and `p` disappear
- `x` does not
- Destruction happens **only** when `x`’s lifetime ends

---