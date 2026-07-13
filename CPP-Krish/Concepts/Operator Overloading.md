
You can redefine how standard symbols (`+`, `-`, `==`, `<<`) work with your custom objects.
We can implement this as a **Member Function** or a **Friend Function**. (Member is usually preferred for `+`, `-`, etc.)


```cpp
class Vector2D {
public:
    int x, y;
    Vector2D(int x, int y) : x(x), y(y) {}

    // Syntax: ReturnType operatorOP(Argument)
    // "When someone writes 'this + other', run this code."
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(this->x + other.x, this->y + other.y);
    }
    
    // Comparison Operator
    bool operator==(const Vector2D& other) const {
        return (this->x == other.x) && (this->y == other.y);
    }
};

int main() {
    Vector2D v1(1, 2);
    Vector2D v2(3, 4);
    Vector2D v3 = v1 + v2; // Calls v1.operator+(v2)
    if (v1 == v2) cout << "Same!";
}
```

#### The Special Case:  `<<` (Printing)

You cannot make this a member function because the syntax `cout << obj` puts `cout` on the left. To fix this, we implement it as a **Global Friend Function**.

```cpp
class Person {
    string name;
public:
    Person(string n) : name(n) {}
    // Friend declaration
    friend ostream& operator<<(ostream& os, const Person& p) {
		os << "Person(" << p.name << ")";
	    return os;
	    // We return 'ostream&' to allow chaining: cout << p1 << p2;
    }
};
```

The C++ compiler treats every single operator as a standard function call.

When the compiler sees a binary operator (like `+` or `<<`), it first looks at the object on the **left**. If you write `v1 + v2`, `v1` is on the left. The compiler translates this into a member function call: `v1.operator+(v2)`. The left object owns the function.

Now look at printing: `cout << myPerson`. Because `cout` is on the left, the compiler tries to translate it to: `cout.operator<<(myPerson)`.

For this to work, you would have to open the official C++ Standard Library source code, find the `std::ostream` class (which is the blueprint for `cout`), and type your custom method into it. Because you cannot edit the C++ Standard Library, a member function is impossible.

Because the compiler cannot find a method inside `cout`, its fallback plan is to look for a **standalone, global function** that takes _both_ objects as arguments. `cout << myPerson`
is converted to `operator<<(cout, myPerson)`

- **Argument 1 (`ostream& os`):** We must pass it by reference (`&`) because `cout` is tied to the physical hardware of the console screen, you cannot create a "copy" of a hardware stream in RAM.

- **Argument 2 (`const Person& p`):** The compiler passes `myPerson` into this slot.

If you write `cout << p1 << p2;`, the compiler evaluates it left-to-right:

1. It executes the first half: `operator<<(cout, p1)`.
2. The function prints `p1`'s name, and then **returns `cout`**.
3. Because it returned `cout`, the line of code physically resolves to `cout << p2;`.
4. It then executes the second half: `operator<<(cout, p2)`.

---
### Introduction

When evaluating an expression containing an operator, the compiler uses the following rules:
- If _all_ of the operands are fundamental data types, the compiler will call a built-in routine if one exists. If one does not exist, the compiler will produce a compiler error.
- If _any_ of the operands are program-defined types, the compiler will use the function overload resolution algorithm to find an unambiguous best match. This may involve implicitly converting one or more operands to match the parameter types of an overloaded operator. It may also involve implicitly converting program-defined types into fundamental types (via an overloaded typecast, which we’ll cover later in this chapter) so that it can match a built-in operator. If no match can be found (or an ambiguous match is found), the compiler will error.

Limitations: 
- Following cannot be overloaded: conditional `(?:)`, `sizeof`, scope `(::)`, member selector `(.)`, pointer member selector `(.*)`, `typeid`, and the casting operators.
- You can not create new operators or rename existing operators. You can only overload the operators that exist. For example, you can not create an `operator**` to do exponents.
- At least one of the operands in an overloaded operator must be a user-defined type. This means you could overload `operator+(int, Mystring)`, but not `operator+(int, double)`. Because standard library classes are considered to be user-defined, this means you could define `operator+(double, std::string)`.  However, this is not a good idea because a future language standard could define this overload, which could break  programs.  Thus the overloaded operators should should have at least one program-defined type. 
- It is not possible to change the number of operands an operator supports.
- All operators keep their default precedence and associativity and this can not be changed.

Best practice
- Operators that do not modify their operands (e.g. arithmetic operators) should generally return results by value.
- Operators that modify their leftmost operand (e.g. pre-increment, any of the assignment 

---