C++ supports 5 different types of casts: `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`, and C-style casts. The first four are sometimes referred to as **named casts**.

| Cast             | Description                                                                                           | Safe?                 |
| ---------------- | ----------------------------------------------------------------------------------------------------- | --------------------- |
| static_cast      | Performs compile-time type conversions between related types.                                         | Yes                   |
| dynamic_cast     | Performs runtime type conversions on pointers or references in an polymorphic (inheritance) hierarchy | Yes                   |
| const_cast       | Adds or removes const.                                                                                | Only for adding const |
| reinterpret_cast | Reinterprets the bit-level representation of one type as if it were another type                      | No                    |
| C-style casts    | Discussed below                                                                                       | No                    |

`const_cast` and `reinterpret_cast` should generally be avoided because they are only useful in rare cases and can be harmful if used incorrectly.

---
### C-style cast

In standard C programming, casting is done via `operator()`, with the name of the type to convert to placed inside the parentheses, and the value to convert to placed immediately to the right of the closing parenthesis. In C++, this type of cast is called a **C-style cast**.

```cpp
int x { 10 };
int y { 4 };
std::cout << (double)x / y << '\n'; // C-style cast of x to double
```

C++ also provides an alternative form of C-style cast known as a **function-style cast**, which resembles a function call:
```cpp
std::cout << double(x) / y << '\n'; //  // function-style cast of x to double
```

There is a significant reason that C-style casts are generally avoided in C++:
Although a C-style cast appears to be a single cast, it can actually perform a variety of different conversions depending on how it is used. A C-style cast does not make it clear which cast(s) will actually be performed.
A C-style cast tries to perform the following C++ casts, in order:
- `const_cast`
- `static_cast`
- `static_cast`, followed by `const_cast`
- `reinterpret_cast`
- `reinterpret_cast`, followed by `const_cast`


There is one thing you can do with a C-style cast that you can’t do with C++ casts: 
C-style casts can convert a derived object to a base class that is inaccessible (e.g. because it was privately inherited). It just turns a blind eye to the access specifier during `static_cast`.


---
### Casting vs initializing a temporary object

Let’s say we have some variable `x` that we need to convert to an `int`. There are two conventional ways we can do this:

1. `static_cast<int>(x)`, which returns a temporary `int` object _direct-initialized_ with `x`.
2. `int { x }`, which creates a temporary `int` object _direct-list-initialized_ with `x`.

We should avoid `int ( x )`, which is a C-style cast. This will return a temporary `int` direct-initialized with the value of `x` (like we’d expect from the syntax), but it also has the other downsides as discussed.

There are (at least) three notable differences between the `static_cast` and the direct-list-initialized temporary:

1. `int { x }` uses list initialization, which disallows narrowing conversions. This is great when initializing a variable, because we rarely intend to lose data in such cases. But when using a cast, it is presumed we know what we’re doing, and if we want to do a cast that might lose some data, we should be able to do that. The narrowing conversion restriction can be an impediment in this case.

```cpp
// We want to do fp division, so one of the operands needs to be a fp type
std::cout << double{x} / y << '\n'; 
// okay if int is 32-bit, narrowing if x is 64-bit
```

2. `static_cast` makes it clearer that we are intending to perform a conversion. Although the `static_cast` is more verbose than the direct-list-initialized alternative, in this case, that’s a good thing, as it makes the conversion easier to spot and search for. That ultimately makes your code safer and easier to understand.

3. Direct-list-initialization of a temporary only allows single-word type names. Due to a weird syntax quirk, there are several places within C++ where only single-word type names are allowed (the C++ standard calls these names “simple type specifiers”). So while `int { x }` is a valid conversion syntax, `unsigned int { x }` is not.

```cpp
	unsigned char c { 'a' };
	std::cout << unsigned int { c } << '\n'; // compile-error
```

Workaround:

```cpp
	unsigned char c { 'a' };
	using uint = unsigned int;
	std::cout << uint { c } << '\n';
```

For all these reasons, we generally prefer `static_cast` over direct-list-initialization of a temporary.

----
