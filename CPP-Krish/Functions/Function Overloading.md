**Function overloading** allows us to create multiple functions with the same name, so long as each identically named function has different parameter types (or the functions can be otherwise differentiated). Each function sharing a name (in the same scope) is called an **overloaded function** (sometimes called an **overload** for short).

When a function call is made to a function that has been overloaded, the compiler will try to match the function call to the appropriate overload based on the arguments used in the function call. This is called **overload resolution**.

In order for a program using overloaded functions to compile, two things have to be true:
1. Each overloaded function has to be differentiated from the others.
2. Each call to an overloaded function has to resolve to an overloaded function.

### How overloaded functions are differentiated

| Function property    | Used for differentiation | Notes                                                                                        |
| -------------------- | ------------------------ | -------------------------------------------------------------------------------------------- |
| Number of parameters | Yes                      |                                                                                              |
| Type of parameters   | Yes                      | Excludes typedefs, type aliases, and const qualifier on value parameters. Includes ellipses. |
| Return type          | No                       |                                                                                              |

Because type aliases (or typedefs) are not distinct types, overloaded functions using type aliases are not distinct from overloads using the aliased type. For example, all of the following overloads are not differentiated (and will result in a compile error):

```cpp
typedef int Height; // typedef
using Age = int; // type alias

void print(int value);
void print(Age value); // not differentiated from print(int)
void print(Height value); // not differentiated from print(int)
```

For parameters passed by value, the const qualifier is also not considered. Therefore, the following functions are not considered to be differentiated:

```cpp
void print(int);
void print(const int); // not differentiated from print(int)
```

Ellipsis parameters are considered to be a unique type of parameter:

```cpp
void foo(int x, int y);
void foo(int x, ...); // differentiated from foo(int, int)
```

Thus a call to `foo(4, 5)` will match to `foo(int, int)`, not `foo(int, ...)`.


A function’s return type is not considered when differentiating overloaded functions because if you saw the function call, you cannot decide which one to call. This was an intentional choice, as it ensures the behavior of a function call can be determined independently from the rest of the expression, making understanding complex expressions much simpler. Put another way, we can always determine which version of a function will be called based solely on the arguments in the function call. If return values were used for differentiation, then we wouldn’t have an easy syntactic way to tell which overload of a function was being called, we’d also have to understand how the return value was being used, which requires a lot more analysis.

---
For member functions, additional function-level qualifiers are also considered: As an example, a const member function can be differentiated from an otherwise identical non-const member function (even if they share the same set of parameters).

| Function-level qualifier | Used for overloading |
| ------------------------ | -------------------- |
| const or volatile        | Yes                  |
| Ref-qualifiers           | Yes                  |

---

A function’s **type signature** (generally called a **signature**) is defined as the parts of the function header that are used for differentiation of the function. In C++, this includes the function name, number of parameters, parameter type, and function-level qualifiers. It notably does _not_ include the return type.

-----
### Resolving overloaded function calls

When a function call is made to an overloaded function, the compiler steps through a sequence of rules to determine which of the overloaded functions is the best match.

At each step, the compiler applies a bunch of different type conversions to the argument(s) in the function call. For each conversion applied, the compiler checks if any of the overloaded functions are now a match. After all the different type conversions have been applied and checked for matches, the step is done. The result will be one of three possible outcomes:

- No matching functions were found. The compiler moves to the next step in the sequence.
- A single matching function was found. This function is considered to be the best match. The matching process is now complete, and subsequent steps are not executed.
- More than one matching function was found. The compiler will issue an ambiguous match compile error.

If the compiler reaches the end of the entire sequence without finding a match, it will generate a compile error that no matching overloaded function could be found for the function call.

#### Step 1:
The compiler tries to find an exact match. This happens in two phases. First, the compiler will see if there is an overloaded function where the type of the arguments in the function call exactly matches the type of the parameters in the overloaded functions.

The compiler will apply a number of trivial conversions to the arguments in the function call. The **trivial conversions** are a set of specific conversion rules that will modify types (without modifying the value) for purposes of finding a match. These include:

- lvalue to rvalue conversions
- qualification conversions (e.g. non-const to const)
- non-reference to reference conversions

```cpp
void foo(const int) {}
void foo(const double&) {}

int main() {
    int x { 1 };
    foo(x);
    double d { 2.3 };
    foo(d); 
}
```

The compiler will trivially convert `x` from an `int` into a `const int`, which then matches `foo(const int)`.  The compiler will trivially convert `d` from a `double` to a `const double&`, which then matches `foo(const double&)`.


Matches made via the trivial conversions are considered exact matches. This means the following program results in an ambiguous match:

```cpp
void foo(int){}
void foo(const int&) {}
int main() {
    int x { 1 };
    foo(x); // ambiguous match with foo(int) and foo(const int&)
}
```

#### Step 2:
If no exact match is found, the compiler tries to find a match by applying numeric promotion to the argument(s). We covered how certain narrow integral and floating point types can be automatically promoted to wider types, such as `int` or `double`. If, after numeric promotion, a match is found, the function call is resolved.


```cpp
void foo(int) {}
void foo(double) {}
int main(){
    foo('a');  // promoted to match foo(int)
    foo(true); // promoted to match foo(int)
    foo(4.5f); // promoted to match foo(double)
}
```

For `foo('a')`, because an exact match for `foo(char)` could not be found in the prior step, the compiler promotes the char `'a'` to an `int`, and looks for a match. This matches `foo(int)`, so the function call resolves to `foo(int)`.

#### Step 3:
If no match is found via numeric promotion, the compiler tries to find a match by applying numeric conversion to the arguments.

```cpp
void foo(double) {}
void foo(std::string) {}

int main() {
    foo('a'); // 'a' converted to match foo(double)
}
```

In this case, because there is no `foo(char)` (exact match), and no `foo(int)` (promotion match), the `'a'` is numerically converted to a double and matched with `foo(double)`.

**Key insight : Matches made by applying numeric promotions take precedence over any matches made by applying numeric conversions.**

#### Step 4:
If no match is found via numeric conversion, the compiler tries to find a match through any user-defined conversions. Certain types (e.g. classes) can define conversions to other types that can be implicitly invoked.

```cpp
class X {
public:
    operator int() { return 0; } 
    // Here's a user-defined conversion from X to int
};

void foo(int){}
void foo(double) {}
int main() {
    X x; 
    foo(x); // x is converted to type int using the user-defined conversion from X to int
}
```

In this example, the compiler will first check whether an exact match to `foo(X)` exists. We haven’t defined one. Next the compiler will check whether `x` can be numerically promoted, which it can’t. The compiler will then check if `x` can be numerically converted, which it also can’t. Finally, the compiler will then look for any user-defined conversions. Because we’ve defined a user-defined conversion from `X` to `int`, the compiler will convert `X` to an `int` to match `foo(int)`.

After applying a user-defined conversion, the compiler may apply additional implicit promotions or conversions to find a match. So if our user-defined conversion had been to type `char` instead of `int`, the compiler would have used the user-defined conversion to `char` and then promoted the result into an `int` to match.

We discuss how to create user-defined conversions for class types (by overloading the typecast operators) in lesson [21.11 -- Overloading typecasts](https://www.learncpp.com/cpp-tutorial/overloading-typecasts/).

The constructor of a class also acts as a user-defined conversion from other types to that class type, and can be used during this step to find matching functions.

#### Step 5:
If no match is found via user-defined conversion, the compiler will look for a matching function that uses ellipsis.

### Step 6:
If no matches have been found by this point, the compiler gives up and will issue a compile error about not being able to find a matching function.

---
### Ambiguous Match

An **ambiguous match** occurs when the compiler finds two or more functions that can be made to match in the same step. When this occurs, the compiler will stop matching and issue a compile error stating that it has found an ambiguous function call.

Let’s take a look at an example that illustrates this:

```cpp
void foo(int) {}
void foo(double) {}
int main(){
    foo(5L); // 5L is type long
    return 0;
}
```

Since literal `5L` is of type `long`, the compiler will first look to see if it can find an exact match for `foo(long)`, but it will not find one. Next, the compiler will try numeric promotion, but values of type `long` can’t be promoted, so there is no match here either.

Following that, the compiler will try to find a match by applying numeric conversions to the `long` argument. In the process of checking all the numeric conversion rules, the compiler will find two potential matches. If the `long` argument is numerically converted into an `int`, then the function call will match `foo(int)`. If the `long` argument is instead converted into a `double`, then it will match `foo(double)` instead. Since two possible matches via numeric conversion have been found, the function call is considered ambiguous.


Here’s another example that yields ambiguous matches:

```cpp
void foo(unsigned int){}
void foo(float){}

foo(0);   // int can be numerically converted to unsigned int or to float
foo(3.14159);  // double can be numerically converted to unsigned int or to float
```

Although you might expect `0` to resolve to `foo(unsigned int)` and `3.14159` to resolve to `foo(float)`, both of these calls result in an ambiguous match. The `int` value `0` can be numerically converted to either an `unsigned int` or a `float`, so either overload matches equally well, and the result is an ambiguous function call.

The same applies for the conversion of a `double` to either a `float` or `unsigned int`. Both are numeric conversions, so either overload matches equally well, and the result is again ambiguous.

Default arguments can also cause ambiguous matches. We cover such cases in lesson [11.5 -- Default arguments](https://www.learncpp.com/cpp-tutorial/default-arguments/).

----
### Resolving ambiguous matches

Because ambiguous matches are a compile-time error, an ambiguous match needs to be disambiguated before your program will compile. There are a few ways to resolve ambiguous matches:

1. Define a new overloaded function that takes parameters of exactly the type you are trying to call the function with. Then C++ will be able to find an exact match for the function call.
2. Explicitly cast the ambiguous argument(s) to match the type of the function you want to call. For example, to have `foo(0)` match `foo(unsigned int)` in the above example, you would do this:
3. If your argument is a literal, you can use the literal suffix to ensure your literal is interpreted as the correct type:

---
### Matching for functions with multiple arguments

If there are multiple arguments, the compiler applies the matching rules to each argument in turn. The function chosen is the one for which each argument matches at least as well as all the other functions, with at least one argument matching better than all the other functions

In the case that such a function is found, it is clearly and unambiguously the best choice. If no such function can be found, the call will be considered ambiguous (or a non-match).

```cpp
void print(char, int) { std::cout << 'a' << '\n'; }
void print(char, double) { std::cout << 'b' << '\n'; }
void print(char, float) { std::cout << 'c' << '\n'; }
print('x', 'a');
```

In the above program, all functions match the first argument exactly. However, the top function matches the second parameter via promotion, whereas the other functions require a conversion. Therefore, `print(char, int)` is unambiguously the best match.

---
### Deleting a function using the `= delete` specifier

In cases where we have a function that we explicitly do not want to be callable, we can define that function as deleted by using the **= delete** specifier. If the compiler matches a function call to a deleted function, compilation will be halted with a compile error.


```cpp
void printInt(int x) { std::cout << x << '\n'; }
void printInt(char) = delete; // calls to this function will halt compilation
void printInt(bool) = delete; // calls to this function will halt compilation

int main() {
    printInt(97);   // okay
    printInt('a');  // compile error: function deleted
    printInt(true); // compile error: function deleted
    printInt(5.0);  // compile error: ambiguous match
    return 0;
}
```

`printInt(5.0)` is an interesting case, with perhaps unexpected results. First, the compiler checks to see if exact match `printInt(double)` exists. It does not. Next, the compiler tries to find a best match. Although `printInt(int)` is the only non-deleted function, the deleted functions are still considered as candidates in function overload resolution. Because none of these functions are unambiguously the best match, the compiler will issue an ambiguous match compilation error.

Key insight: `= delete` means “I forbid this”, not “this doesn’t exist”.
Deleted function participate in all stages of function overload resolution (not just in the exact match stage). If a deleted function is selected, then a compilation error results.


Other types of functions can be similarly deleted. We discuss deleting member functions in lesson [14.14 -- Introduction to the copy constructor](https://www.learncpp.com/cpp-tutorial/introduction-to-the-copy-constructor/), and deleting function template specializations in lesson [11.7 -- Function template instantiation](https://www.learncpp.com/cpp-tutorial/function-template-instantiation/).

#### Deleting all non-matching overloads Advanced

There may be times when we want a certain function to be called only with arguments whose types exactly match the function parameters. We can do this by using a function template (introduced in upcoming lesson [11.6 -- Function templates](https://www.learncpp.com/cpp-tutorial/function-templates/)) as follows:


```cpp
// This function will take precedence for arguments of type int
void printInt(int x) { std::cout << x << '\n'; }

// This function template will take precedence for arguments of other types
// Since this function template is deleted, calls to it will halt compilation
template <typename T>
void printInt(T x) = delete;

int main() {
    printInt(97);   // okay
    printInt('a');  // compile error
    printInt(true); // compile error
}
```

---

### Default Arguments

Note that you must use the equals sign to specify a default argument. Using parenthesis or brace initialization won’t work:

```cpp
void foo(int x = 5);   // ok
void goo(int x ( 5 )); // compile error
void boo(int x { 5 }); // compile error
```

Perhaps surprisingly, default arguments are handled by the compiler at the call site. Default arguments are inserted by the compiler at site of the function call.

Default arguments are useful in cases where we need to add a new parameter to an existing function. 

A function can have multiple parameters with default arguments. 
1. In a function call, any explicitly provided arguments must be the leftmost arguments (arguments with defaults cannot be skipped).
2. If a parameter is given a default argument, all subsequent parameters (to the right) must also be given default arguments.
3. If more than one parameter has a default argument, the leftmost parameter should be the one most likely to be explicitly set by the user.


Once declared, a default argument can not be redeclared in the same translation unit. That means for a function with a forward declaration and a function definition, the default argument can be declared in either the forward declaration or the function definition, but not both.

```cpp
void print(int x, int y=4); // forward declaration
void print(int x, int y=4)  {
	// compile error: redefinition of default argument
    std::cout << x << y << '\n';
}
```

The default argument must also be declared in the translation unit before it can be used:

```cpp
void print(int x, int y); // forward declaration, no default argument
int main() {
    print(3); // compile error: default argument for y hasn't been defined yet
}
void print(int x, int y=4)  { std::cout << x << y << '\n'; }
```

The best practice is to declare the default argument in the forward declaration and not in the function definition, as the forward declaration is more likely to be seen by other files and included before use (particularly if it’s in a header file).

Functions with default arguments may be overloaded. For example, the following is allowed:

```cpp
void print(std::string_view s) { std::cout << s << '\n';}
void print(char c = ' ') { std::cout << c << '\n'; }

int main() {
    print("Hello, world"); // resolves to print(std::string_view)
    print('a');            // resolves to print(char)
    print();               // resolves to print(char)
}
```

Now consider this case:

```cpp
void print(int x);                  // signature print(int)
void print(int x, int y = 10);      // signature print(int, int)
void print(int x, double y = 20.5); // signature print(int, double)
```

Default values (not arguments) are not part of a function’s signature, so these function declarations are differentiated overloads.


Default arguments can easily lead to ambiguous function calls:

```cpp
void print(int x);                  // signature print(int)
void print(int x, int y = 10);      // signature print(int, int)
void print(int x, double y = 20.5); // signature print(int, double)

int main() {
    print(1, 2);   // will resolve to print(int, int)
    print(1, 2.5); // will resolve to print(int, double)
    print(1);      // ambiguous function call
}
```

In the case where we mean to call `print(int, int)` or `print(int, double)` we can always explicitly specify the second parameter. But what if we want to call `print(int)`? It’s not obvious how we can do so. There is a way though

Default arguments don’t work for functions called through function pointers
We cover this topic in lesson [20.1 -- Function Pointers](https://www.learncpp.com/cpp-tutorial/function-pointers/). Because default arguments are not considered using this method, this also provides a workaround to call a function that would otherwise be ambiguous due to default arguments.

---