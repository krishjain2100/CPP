Because strings are commonly used in programs, most modern programming languages include a fundamental string data type. For historical reasons, strings are not a fundamental type in C++. Rather, they have a strange, complicated type that is hard to work with. Such strings are often called **C strings** or **C-style strings**, as they are inherited from the C-language.

1. All C-style string literals have an implicit null terminator. Consider a string such as `"hello"`. While this C-style string appears to only have five characters, it actually has six: `'h'`, `'e'`, `'l`‘, `'l'`, `'o'`, and `'\0'` (a character with ASCII code 0). This trailing `\0` character is a special character called a **null terminator**, and it is used to indicate the end of the string.

This is the reason the string `"Hello, world!"` has type `const char[14]` rather than `const char[13]`, the hidden null terminator counts as a character. The reason for the null-terminator is also historical: it can be used to determine where the string ends.

2. Unlike most other literals (which are values, not objects), C-style string literals are const objects that are created at the start of the program and are guaranteed to exist for the entirety of the program.

Unlike C-style string literals, `std::string` and `std::string_view` literals create temporary objects. These temporary objects must be used immediately, as they are destroyed at the end of the full expression in which they are created.

While C-style string literals are fine to use, C-style string variables behave oddly, are hard to work with (e.g. you can’t use assignment to assign a C-style string variable a new value), and are dangerous (e.g. if you copy a larger C-style string into the space allocated for a shorter C-style string, undefined behaviour will result). 

---
### `std::string`

- If `std::string` doesn’t have enough memory to store a string (maybe because of reassignment), it will request additional memory (at runtime) using dynamic memory allocation. This ability to acquire additional memory is part of what makes `std::string` so flexible, but also comparatively slow.
- To read a full line of input into a string, you’re better off using the `std::getline()` function instead. `std::getline()` requires two arguments: the first is `std::cin`, and the second is your string variable.
- The `std::ws` input manipulator tells `std::cin` to ignore any leading whitespace before extraction.

```cpp
#include <iostream>
#include <string> 
int main() {
    cout << "Enter your full name: ";
    string name{};
    std::getline(cin >> std::ws, name); // read a full line of text into name

    cout << "Enter your favorite color: ";
    string color{};
    std::getline(cin >> std::ws, color); // read a full line of text into color
}
```

If the string is small enough it can be stored inside the `std::string` object itself (This object can be either on the stack or the heap). If the string is large, then the actual string is on the heap and the `std::string` object contains a pointer to it.

---
####  `std::ws` (input manipulator)

The `std::ws` input manipulator tells `std::cin` to ignore any leading whitespace before extraction. Leading whitespace is any whitespace character (spaces, tabs, newlines) that occur at the start of the string.

1. using `std::cin >> x`: it first removes any leading whitespace from buffer and then extracts until another whitespace is found, remember it leaves the whitespace found at the end.
2. using `std::getline(std::cin, x)`: it extracts until it sees a `\n` and then stops and also removes the `\n` from the buffer but doesn't store it in the variable
3. using `std::cin >> std::ws`: It clears the buffer of any leading whitespace including `'\n'`
4. using `std::getline(std::cin >> std::ws, x)`: first 3rd process takes place then second takes place

```cpp
#include <iostream>
#include <string>

int main() {
    std::cout << "Pick 1 or 2: ";
    int choice{};
    std::cin >> choice;
    std::cout << "Now enter your name: ";
    std::string name{};
    std::getline(std::cin, name); // note: no std::ws here
    std::cout << "Hello, " << name << ", you picked " << choice << '\n';
}
```

Output:
Pick 1 or 2: 2
Now enter your name: Hello, , you picked 2

It didn't wait for you to enter your name. Instead, it prints the “Hello” string, and then exits.
When you enter a value using `operator>>`, `std::cin` captures the string `"2\n"` as input. It then extracts the value `2` to variable `choice`, leaving the newline character behind for later. Then, when `std::getline()` goes to extract text to `name`, it sees `"\n"` is already waiting in `std::cin`, and uses it. We can amend the above program to use the `std::ws` input manipulator, to tell `std::getline()` to ignore any leading whitespace characters.

`std::ws` is not preserved across calls like other manipulators like `std::setprecision()`, it has to be stated again and again for use.

---

Although `std::string` is required to be null-terminated (as of C++11), the returned length of a `std::string` does not include the implicit null-terminator character.

Note that `std::string::length()` returns an unsigned integral value (most likely of type `size_t`). If you want to assign the length to an `int` variable, you should `static_cast` it to avoid compiler warnings about signed/unsigned conversions.

---
####  `std::string` as function argument
In most cases, use a `std::string_view` parameter instead, otherwise expensive copy created

---
#### Returning a `std::string`
**DID NOT UNDERSTAND FULLY**

When a function returns by value to the caller, the return value is normally copied from the function back to the caller. So you might expect that you should not return `std::string` by value, as doing so would return an expensive copy of a `std::string`.

However, as a rule of thumb, it is okay to return a `std::string` by value when the expression of the return statement resolves to any of the following:
- A local variable of type `std::string`.
- A `std::string` that has been returned by value from another function call or operator.
- A `std::string` temporary that is created as part of the return statement.

`std::string` supports a capability called move semantics, which allows an object that will be destroyed at the end of the function to instead be returned by value without making a copy.

In most other cases, prefer to avoid returning a `std::string` by value, as doing so will make an expensive copy.

If returning a C-style string literal, use a `std::string_view` return type instead.
In certain cases, `std::string` may also be returned by (const) reference, which is another way to avoid making a copy. discuss later in [12.12 -- Return by reference and return by address](https://www.learncpp.com/cpp-tutorial/return-by-reference-and-return-by-address/) and [14.6 -- Access functions](https://www.learncpp.com/cpp-tutorial/access-functions/).

---
#### Literals for `std::string`

- Double-quoted string literals (like “Hello, world!”) are C-style strings by default (and thus, have a strange type).

- Use a `s` suffix after the double quotes instead. it should be lower case
	The “s” suffix lives in the namespace `std::literals::string_literals`.
	The most concise way to access the literal suffixes is via using-directive `using namespace std::literals`, however this imports many unnecessary things as well. So use `using namespace std::string_literals`, which imports only the literals for `std::string`

- You probably won’t need to use `std::string` literals very often (as it’s fine to initialse a `std::string` object with a C-style string literal), but we’ll see a few cases in future lessons (involving type deduction) where using `std::string` literals instead of C-style string literals makes things easier (see [10.8 -- Type deduction for objects using the auto keyword](https://www.learncpp.com/cpp-tutorial/type-deduction-for-objects-using-the-auto-keyword/) for an example).

- `"Hello"s` resolves to `std::string { "Hello", 5 }` which creates a temporary `std::string` initialised with C-style string literal “Hello”

```cpp
#include <string> 
int main() {
    using namespace std::string_literals; // easy access to the s suffix
    std::cout << "foo\n";   // no suffix is a C-style string literal
    std::cout << "goo\n"s;  // s suffix is a std::string literal
}
```

---
### Constexpr Strings
If you try to define a `constexpr std::string`, your compiler will probably generate an error, This happens because `constexpr std::string` isn’t supported at all in C++17 or earlier, and only works in very limited cases in C++20/23. If you need constexpr strings, use `std::string_view`

---
### std::string_view (C++17)

To address the issue with `std::string` being expensive to initialize (or copy), C++17 introduced `std::string_view` (which lives in the `<string_view>` header). `std::string_view` provides read-only access to an _existing_ string (a C-style string, a `std::string`, or another `std::string_view`) without making a copy.  It just holds a pointer and length of the string its looking at.

A `std::string_view` object can be initialised with a C-style string, a `std::string`, or another `std::string_view`.
Both a C-style string and a `std::string` will implicitly convert to a `std::string_view` when passed as an argument to a `std::string_view. 

---
#### `std::string_view` will not implicitly convert to `std::string`

This is because `std::string` makes a copy of its initializer (which is expensive and we don't want it to happen accidentally). However, if this is desired, we have two options:
1. Explicitly create a `std::string` with a `std::string_view` initializer
2. Convert an existing `std::string_view` to a `std::string` using `static_cast`

```cpp

void printString(std::string str) { std::cout << str << '\n'; }
int main() {
	std::string_view sv{ "Hello, world!" };
	// printString(sv); 
	 // compile error: won't implicitly convert std::string_view to a std::string
	std::string s{ sv }; 
	// okay: we can create std::string using std::string_view initializer
	printString(s); // and call the function with the std::string
	printString(static_cast<std::string>(sv)); 
	// okay: we can explicitly cast a std::string_view to a std::string
}
```

---
#### Assignment changes what the `std::string_view` is viewing
It changes the internal pointer and size, not the string it is viewing.

---
#### Literals for `std::string_view`
- We can create string literals with type `std::string_view` by using a `sv` suffix after the double-quoted string literal. The `sv` must be lower case.
- `sv` resides in namespace `std::literals::string_view::literals`, use `using namespace std::string_view_literals` just like in string

```cpp
int main() {
    using namespace std::string_literals;      // access the s suffix
    using namespace std::string_view_literals; // access the sv suffix
    std::cout << "foo\n";   // no suffix is a C-style string literal
    std::cout << "goo\n"s;  // s suffix is a std::string literal
    std::cout << "moo\n"sv; // sv suffix is a std::string_view literal
}
```

Ordinary string literal `"foo\n"` has type `const char[5]` because the characters are:
`'f' 'o' 'o' '\n' '\0'.` The compiler places these characters in the program's read-only static storage. This memory exists for the entire lifetime of the program.

`"goo\n"s; is equivalent to std::string("moo\n"). The literal characters still live in static storage: Then, at runtime, a `std::string` object is constructed.
Conceptually: `std::string temp{"goo\n"};` So this actually creates a `std::string` object.

`"moo\n"sv` is approximately `std::string_view{"moo\n", 4}`. The characters still live in static storage. 

---
#### Constexpr
Unlike `std::string`, `std::string_view` has full support for constexpr

```cpp
constexpr std::string_view s{ "Hello, world!" };
// s is a string symbolic constant
cout << s << '\n'; 
// s will be replaced with "Hello, world!" at compile-time
```

---
#### `std::string` is a (sole) owner
In programming, when we call an object an owner, we generally mean that it is the sole owner (unless otherwise specified). Sole ownership (also called single ownership) ensures it is clear who has responsibility for that data.

#### `std::string_view` is a viewer
A view is dependent on the object being viewed. If the object being viewed is modified or destroyed while the view is still being used, unexpected or undefined behavior will result.
A `std::string_view` that is viewing a string that has been destroyed is sometimes called a **dangling** view.

#### Should I prefer `std::string_view` or `const std::string&` function parameters? 
Prefer `std::string_view` in most cases. We cover this topic further in lesson [12.6 -- Pass by const lvalue reference](https://www.learncpp.com/cpp-tutorial/pass-by-const-lvalue-reference/#stringparameter).

---
### Improperly using `std::string_view`(Important)

```cpp
std::string getName() {
    std::string s { "Alex" };
    return s;
}

int main() {
  std::string_view name { getName() }; 
  std::cout << name << '\n'; // undefined behavior
}
```
The getName() returns a temporary object which gets destroyed at then end of the full expression containing the function call. We must either use this return value immediately, or copy it to use later.(fixes are provided after the next part)

The following is a less-obvious variant of the above:
```cpp
int main(){
    using namespace std::string_literals;
    std::string_view name { "Alex"s }; 
    std::cout << name << '\n'; // undefined behavior
}
```

A `std::string` literal (created via the `s` literal suffix) creates a temporary `std::string` object. So in this case, `"Alex"s` creates a temporary `std::string`, which we then use as the initialiser for `name`. At this point, `name` is viewing the temporary `std::string`. Then the temporary `std::string` is destroyed, leaving `name` dangling. We get undefined behaviour when we then use `name`.

It is okay to initialise a `std::string_view` with a C-style string literal or a `std::string_view` literal. C-style literals are permanent, the compiler stores them in the data segment, while `string_view` literals are temporary string view object pointing to the C-style literal so that also causes no problem.

Also it’s also okay to initialise a `std::string_view` with a C-style string object, a `std::string` object, or a `std::string_view` object, as long as that string object outlives the view.  

**Fixes for the first code**
Note that the return type has changed in both cases.
```cpp
std::string_view getName()  {
    return "Alex"; // Perfectly safe!
}
```

```cpp
std::string_view getName() {
	std::string_view s { "Alex" };// s points to c-style literal(permanent)
	return s; // we return the view which is copied there and so this works too
} 
```

---
### Invalidation of `string_view`

If the `std::string` reallocates memory in order to accommodate the new string data, it will return the memory used for the old string data back to the operating system. Since the `std::string_view` is still viewing the old string data, it is now dangling (pointing to a now-invalid object). 
If the `std::string` does not reallocate memory, it will copy the new string data over the old string data (starting at the same memory address). The `std::string_view` will now be viewing the new string data (since it was placed at the same memory address as it was viewing), but it will not realize that the length of the `std::string` has probably changed. If the new string is longer than the old string, the `std::string_view` will now be viewing a substring of the new string (of the same length as the old string). If the new string is shorter than the old string, the `std::string_view` will now be viewing a superstring of the new string (consisting of the entire new string, plus whatever garbage characters are still in the memory beyond the end of the string).

Therefore modifying a `std::string` is likely to invalidate all views into that `std::string`.

---

 It is generally okay to return a function parameter of type `std::string_view` since they will be valid when the function returns to the scope where they were passed as arguements.

There is one important subtlety here. If the argument is a temporary object (that will be destroyed at the end of the full expression containing the function call), the `std::string_view` return value must be used in the same expression. After that point, the temporary is destroyed and the std::string_view is left dangling. Example:

```cpp
std::string_view fn(std::string_view a){
	return a;
}
int main(){
	std::string_view a = fn("hello"); // works as hello is permanent
	std::string_view a = fn("hello"s); 
	// fails as hello is destroyed when moving to next line
	cout << fn("hello"s); 
	// works as hello is destroyed before moving to the next line
}
```

---
### View modification functions

- The `remove_prefix()` member function removes characters from the left side of the view.
- The `remove_suffix()` member function removes characters from the right side of the view.

```cpp
std::string_view str{ "Peach" };
// Remove 1 character from the left side of the view
str.remove_prefix(1);//each
// Remove 2 characters from the right side of the view
str.remove_suffix(2);//ea
```

Only way to reset the view is reassigning.

Other ways to view a substring:

- Constructor with pointer + length
```cpp
std::string s = "snowball";
std::string_view sv(s.data() + 2, 3); // "owb"
```
This is a view starting at offset 2, length 3.

- Using `substr()`
```cpp
std::string_view sv = "snowball";
auto sub = sv.substr(2, 3); // "owb"
```
It just returns a new view with adjusted `(ptr, len)`.

---
### `std::string_view` may or may not be null-terminated

The ability to view just a substring of a larger string comes with one consequence of note:
A `std::string_view` may or may not be null-terminated.
In almost all cases, this doesn’t matter as a `std::string_view` keeps track of the length of the string or substring it is viewing, so it doesn’t need the null-terminator.
Converting a `std::string_view` to a `std::string` will work regardless of whether or not the `std::string_view` is null-terminated.

---

