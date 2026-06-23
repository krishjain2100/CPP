While C-style string literals are fine to use, C-style string variables behave oddly, are hard to work with (e.g. you can’t use assignment to assign a C-style string variable a new value).

### Introducing `std::string`
- `"Alex"`, contains five characters (four explicit characters and a null-terminator).
- If `std::string` doesn’t have enough memory to store a string, it will request additional memory (at runtime) using dynamic memory allocation. This ability to acquire additional memory is part of what makes `std::string` so flexible, but also comparatively slow.
- To read a full line of input into a string, you’re better off using the `std::getline()` function instead. `std::getline()` requires two arguments: the first is `std::cin`, and the second is your string variable.
- The `std::ws` input manipulator tells `std::cin` to ignore any leading whitespace before extraction.
#### why use std::ws?
first lets see how input is removed from buffer:
1. using `std::cin >> x`: it first removes any leading whitespace from buffer and then extracts until another whitespace is found, remember it leaves the whitespace found at the end.
2. using `std::getline(std::cin, x)`: it extracts until it sees a `\n` and then stops and also removes the `\n` from the buffer but doesn't store it in the variable
3. using `std::cin >> std::ws`: It clears the buffer of any leading whitespace including '\n'
4. using `std::getline(std::cin >> std::ws, x)`: first 3rd process takes place then second takes place
So as 3 does remove leading whitespace, to remove any leading whitespace that maybe left by 1, we have to use 4 or first 3 then 2

- obv `std::ws` is not like other manipulators like `std::setprecision()`, it has to be stated again and again whenever the buffer needs to be cleared
---
- Although `std::string` is required to be null-terminated (as of C++11), the returned length of a `std::string` does not include the implicit null-terminator character.
- note that `std::string::length()` returns an unsigned integral value (most likely of type `size_t`). If you want to assign the length to an `int` variable, you should `static_cast` it to avoid compiler warnings about signed/unsigned conversions.
- In C++20, you can also use the `std::ssize()` function to get the length of a `std::string` as a large signed integral type (usually `std::ptrdiff_t`):
- Since a `ptrdiff_t` may be larger than an `int`, if you want to store the result of `std::ssize()` in an `int` variable, you should `static_cast` the result to an `int`

#### Do not pass `std::string` by value
In most cases, use a `std::string_view` parameter instead, otherwise expensive copy created
#### Returning a `std::string`
When a function returns by value to the caller, the return value is normally copied from the function back to the caller. So you might expect that you should not return `std::string` by value, as doing so would return an expensive copy of a `std::string`.

However, as a rule of thumb, it is okay to return a `std::string` by value when the expression of the return statement resolves to any of the following:
- A local variable of type `std::string`.
- A `std::string` that has been returned by value from another function call or operator.
- A `std::string` temporary that is created as part of the return statement.
In most other cases, prefer to avoid returning a `std::string` by value, as doing so will make an expensive copy.

- If returning a C-style string literal, use a `std::string_view` return type instead
- In certain cases, `std::string` may also be returned by (const) reference, which is another way to avoid making a copy. discuss later in [12.12 -- Return by reference and return by address](https://www.learncpp.com/cpp-tutorial/return-by-reference-and-return-by-address/) and [14.6 -- Access functions](https://www.learncpp.com/cpp-tutorial/access-functions/).

#### Literals for `std::string`
- Double-quoted string literals (like “Hello, world!”) are C-style strings by default (and thus, have a strange type).
- use a `s` suffix after the double quotes instead. it should be lower case
	The “s” suffix lives in the namespace `std::literals::string_literals`.
	The most concise way to access the literal suffixes is via using-directive `using namespace std::literals`, however this imports many unneccessary things in the scope of using directive
	We recommend `using namespace std::string_literals`, which imports only the literals for `std::string`
- You probably won’t need to use `std::string` literals very often (as it’s fine to initialize a `std::string` object with a C-style string literal), but we’ll see a few cases in future lessons (involving type deduction) where using `std::string` literals instead of C-style string literals makes things easier (see [10.8 -- Type deduction for objects using the auto keyword](https://www.learncpp.com/cpp-tutorial/type-deduction-for-objects-using-the-auto-keyword/) for an example).
- `"Hello"s` resolves to `std::string { "Hello", 5 }`

#### constexpr strings
If you try to define a `constexpr std::string`, your compiler will probably generate an error, This happens because `constexpr std::string` isn’t supported at all in C++17 or earlier, and only works in very limited cases in C++20/23. If you need constexpr strings, use `std::string_view`

#### Conclusion
`std::string` is complex, leveraging many language features that we haven’t covered yet.

#### std::string_view (C++17)

- To address the issue with `std::string` being expensive to initialize (or copy)
- lives in the <string_view> header 
- provides read-only access to an _existing_ string (a C-style string, a `std::string`, or another `std::string_view`) without making a copy.

**Initialising**
One of the neat things about a `std::string_view` is how flexible it is. A `std::string_view` object can be initialized with a C-style string, a `std::string`, or another `std::string_view`
Both a C-style string and a `std::string` will implicitly convert to a `std::string_view`. Therefore, a `std::string_view` parameter(function parameter) will accept arguments of all the three types

### What it is inside?
- It just holds a pointer and length of the string its looking at.

#### `std::string_view` will not implicitly convert to `std::string`
- Because `std::string` makes a copy of its initializer (which is expensive), to prevent accidental conversions.
- However, if this is desired, we have two options:
1. Explicitly create a `std::string` with a `std::string_view` initializer (which is allowed, since this will rarely be done unintentionally)
2. Convert an existing `std::string_view` to a `std::string` using `static_cast`

#### Assignment changes what the `std::string_view` is viewing
It changes the internal pointer and size, not the string it is viewing.

#### Literals for `std::string_view`
- We can create string literals with type `std::string_view` by using a `sv` suffix after the double-quoted string literal. The `sv` must be lower case.
- `sv` resides in namespace `std::literals::string_view::literals`, use `using namespace std::string_view_literals` just like in string
- It’s fine to initialize a `std::string_view` object with a C-style string literal
#### constexpr `std::string_view`
Unlike `std::string`, `std::string_view` has full support for constexpr

---
#### `std::string` is a (sole) owner
In programming, when we call an object an owner, we generally mean that it is the sole owner (unless otherwise specified). Sole ownership (also called single ownership) ensures it is clear who has responsibility for that data.

#### `std::string_view` is a viewer
A view is dependent on the object being viewed. If the object being viewed is modified or destroyed while the view is still being used, unexpected or undefined behavior will result.
A `std::string_view` that is viewing a string that has been destroyed is sometimes called a **dangling** view.

#### Should I prefer `std::string_view` or `const std::string&` function parameters? 
Prefer `std::string_view` in most cases. We cover this topic further in lesson [12.6 -- Pass by const lvalue reference](https://www.learncpp.com/cpp-tutorial/pass-by-const-lvalue-reference/#stringparameter).

### Improperly using `std::string_view`(Important)
```cpp
#include <iostream>
#include <string>
#include <string_view>
std::string getName()
{
    std::string s { "Alex" };
    return s;
}

int main()
{
  std::string_view name { getName() }; // name initialized with return value of function
  std::cout << name << '\n'; // undefined behavior

  return 0;
}
```
The getName() returns a temporary object which gets destroyed at then end of the full expression containing the function call. We must either use this return value immediately, or copy it to use later.(fixes are provided after the next part)

---
The following is a less-obvious variant of the above:
```cpp
#include <iostream>
#include <string>
#include <string_view>

int main()
{
    using namespace std::string_literals;
    std::string_view name { "Alex"s }; // "Alex"s creates a temporary std::string
    std::cout << name << '\n'; // undefined behavior

    return 0;
}
```
A `std::string` literal (created via the `s` literal suffix) creates a temporary `std::string` object. So in this case, `"Alex"s` creates a temporary `std::string`, which we then use as the initializer for `name`. At this point, `name` is viewing the temporary `std::string`. Then the temporary `std::string` is destroyed, leaving `name` dangling. We get undefined behavior when we then use `name`.

It is okay to initialize a `std::string_view` with a C-style string literal or a `std::string_view` literal. It’s also okay to initialize a `std::string_view` with a C-style string object, a `std::string` object, or a `std::string_view` object, as long as that string object outlives the view. 

**What happens above?**
C-style literals are permanent, the compiler bakes those letters directly into the memory of your .exe file, while sv literals are also temporary string view object pointing to the c-style literal so that also causes no problem.

---
**Fixes for the first code**
```cpp
std::string_view getName() 
{
    return "Alex"; // Perfectly safe!
}
```

```cpp
std::string_view getName() {
	std::string_view s { "Alex" };// s point to c-style literal(permanent)
	return s; // we return the view which is copied there and so this works too
} 
```
**see that the return type has changed in both of the above**

---
-  changing original string may cause errors, the string might have moved to another memory address, or if it's length is changed string_view doesn't will not know about it.
---

- We can also return string view that were passed in the function as a parameter and still point to a variable in the caller's scope
- If an argument is a temporary that is destroyed at the end of the full expression containing the function call, the returned `std::string_view` must be used immediately, as it will be left dangling after the temporary is destroyed.
- eg:
```cpp
std::string_view fn(std::string_view a){
	return a;
}
int main(){
	std::string_view a=fn("hello");//works as hello is permanent
	std::string_view a=fn("hello"s);//fails as hello is destroyed when moving to next line
	cout << fn("hello"s); // works as hello is destroyed before moving to the next line
	
}
```

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
- only way to reset the view is reassigning

### `std::string_view` may or may not be null-terminated
The ability to view just a substring of a larger string comes with one consequence of note:
	A C-style string literal and a `std::string` are always null-terminated.  
	A `std::string_view` may or may not be null-terminated.
In almost all cases, this doesn’t matter -- a `std::string_view` keeps track of the length of the string or substring it is viewing, so it doesn’t need the null-terminator. Converting a `std::string_view` to a `std::string` will work regardless of whether or not the `std::string_view` is null-terminated.

### A quick guide on when to use `std::string` vs `std::string_view` [](https://www.learncpp.com/cpp-tutorial/stdstring_view-part-2/#stringvsstringview)
This guide is not meant to be comprehensive, but is intended to highlight the most common cases:

**Variables**
Use a `std::string` variable when:
- You need a string that you can modify.
- You need to store user-inputted text.
- You need to store the return value of a function that returns a `std::string`.
Use a `std::string_view` variable when:
- You need read-only access to part or all of a string that already exists elsewhere and will not be modified or destroyed before use of the `std::string_view` is complete.
- You need a symbolic constant for a C-style string.
- You need to continue viewing the return value of a function that returns a C-style string or a non-dangling `std::string_view`.

**Function parameters**
Use a `std::string` function parameter when:
- The function needs to modify the string passed in as an argument without affecting the caller. This is rare.
- You are using language standard C++14 or older and aren’t comfortable using references yet.
Use a `std::string_view` function parameter when:
- The function needs a read-only string.
- The function needs to work with non-null-terminated strings.


Use a `const std::string&` function parameter when:
- You are using language standard C++14 or older, and the function needs a read-only string to work with (as `std::string_view` is not available until C++17).
- You are calling other functions that require a `const std::string`, `const std::string&`, or const C-style string (as `std::string_view` may not be null-terminated).
Use a `std::string&` function parameter when:
- You are using a `std::string` as an out-parameter (see [12.13 -- In and out parameters](https://www.learncpp.com/cpp-tutorial/in-and-out-parameters/)).
- You are calling other functions that require a `std::string&`, or non-const C-style string.

**Return types**
Use a `std::string` return type when:
- The return value is a `std::string` local variable or function parameter.
- The return value is a function call or operator that returns a `std::string` by value.
Use a `std::string_view` return type when:
- The function returns a C-style string literal or local `std::string_view` that has been initialized with a C-style string literal.
- The function returns a `std::string_view` parameter.

See lesson [12.12 -- Return by reference and return by address](https://www.learncpp.com/cpp-tutorial/return-by-reference-and-return-by-address/) for more information on returning reference types.
Use a `std::string_view` return type when:
- Writing an accessor for a `std::string_view` member.
Use a `std::string&` return type when:
- The function returns a `std::string&` parameter.
Use a `const std::string&` return type when:
- The function returns a `const std::string&` parameter.
- Writing an accessor for a `std::string` or `const std::string` member.
- The function returns a static (local or global) `const std::string`.

**Insights**
Things to remember about `std::string`:
- Initializing and copying `std::string` is expensive, so avoid this as much as possible.
- Avoid passing `std::string` by value, as this makes a copy.
- If possible, avoid creating short-lived `std::string` objects.
- Modifying a `std::string` will invalidate any views to that string.
- It is okay to return a local `std::string` by value.

Things to remember about `std::string_view`:
- `std::string_view` is typically used for passing string function parameters and returning string literals.
- Because C-style string literals exist for the entire program, it is always okay to set a `std::string_view` to a C-style string literal.
- When a string is destroyed, all views to that string are invalidated.
- Using an invalidated view (other than using assignment to revalidate the view) will cause undefined behavior.
- A `std::string_view` may or may not be null-terminated.