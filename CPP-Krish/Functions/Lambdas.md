

Introduced in C++11, A **lambda expression** (also called a **lambda** or **closure**) allows us to define an anonymous function inside another function. This allows us both to avoid namespace naming pollution, and to define the function as close to where it is used as possible (providing additional context).

Lambdas take the form (uses the trailing return type syntax):

```
[ captureClause ] ( parameters ) -> returnType {
    statements;
}
```

- The capture clause can be empty if no captures are needed.
- The parameter list can be empty if no parameters are required. It can also be omitted entirely unless a return type is specified.
- The return type is optional, and if omitted, `auto` will be assumed (thus using type deduction used to determine the return type). While we previously noted that type deduction for function return types should be avoided, in this context, it’s fine to use (because these functions are typically so trivial).

This means a trivial lambda definition looks like this:

```cpp
int main() {
  [] {}; 
  // a lambda with an omitted return type, no captures, and omitted parameters.
}
```

When you write a lambda, the C++ compiler automatically generates an unnamed, hidden class (called a "closure type") that overloads the `operator()`.

---
### Type of a lambda

We can define a lambda right where it is needed (i.e, passing it in as an arguement). 
This use of a lambda is sometimes called a **function literal**. But this is hard to read.
We can also initialize a lambda variable with a lambda definition to use later, making a named lambda: 
```cpp
auto isEven { [](int i) {
	return (i % 2) == 0;
}};
return std::all_of(array.begin(), array.end(), isEven);
```

Lambdas don’t have a type that we can explicitly use. When we write a lambda, the compiler generates a unique type just for the lambda (it uses a new type even if two lambdas are exactly same) that is not exposed to us. In actuality, lambdas aren’t functions (which is part of how they avoid the limitation of C++ not supporting nested functions). They’re a special kind of object called a functor. Functors are objects that contain an overloaded `operator()` that make them callable like a function.


Although we don’t know the type of a lambda, there are several ways of storing a lambda for use post-definition:
- If the lambda has an empty capture clause (nothing between the hard brackets []), we can use a regular function pointer. 
- `std::function` or type deduction via the `auto` keyword will also work (even if the lambda has a non-empty capture clause).

```cpp
int main() {
  // A regular function pointer. Only works with an empty capture clause
  double (*addNumbers1)(double, double){
    [](double a, double b) {
      return a + b;
    }
  };
  addNumbers1(1, 2);

  // Using std::function. The lambda could have a non-empty capture clause
  std::function addNumbers2{ 
    [](double a, double b) {
      return a + b;
    }
  };
  // note: pre-C++17 (CTAD), use std::function<double(double, double)> instead
  addNumbers2(3, 4);

  // Using auto. Stores the lambda with its real type.
  auto addNumbers3 {
    [](double a, double b) {
      return a + b;
    }
  };
  addNumbers3(5, 6);
}
```

The only way of using the lambda’s actual type is by means of `auto`. `auto` also has the benefit of having no overhead compared to `std::function`.

To pass a lambda to a function, there are 4 options:

```cpp
// Case 1: use a `std::function` parameter
void repeat1(int repetitions, const std::function<void(int)>& fn) {
    for (int i{ 0 }; i < repetitions; ++i) fn(i);
}

// Case 2: use a function template with a type template parameter
template <typename T>
void repeat2(int repetitions, const T& fn) {
    for (int i{ 0 }; i < repetitions; ++i) fn(i);
}

// Case 3: use the abbreviated function template syntax (C++20)
void repeat3(int repetitions, const auto& fn) {
    for (int i{ 0 }; i < repetitions; ++i) fn(i);
}

// Case 4: use function pointer (only for lambda with no captures)
void repeat4(int repetitions, void (*fn)(int)) {
    for (int i{ 0 }; i < repetitions; ++i) fn(i);
}
```

In case 1, our function parameter is a `std::function`. This requires the lambda to be implicitly converted whenever the function is called, which adds some overhead. This method also has the benefit of being separable into a declaration (in a header) and a definition (in a .cpp file).

In case 2, we’re using a function template with type template parameter `T`. When the function is called, a function will be instantiated where `T` matches the actual type of the lambda. This is more efficient, but the parameters and return type of `T` are not obvious. Also, definition and declaration are not separable.

In case 3, it generates a function template identical to case 2.

In case 4, the function parameter is a function pointer. Since a lambda with no captures will implicitly convert to a function pointer, we can pass a lambda with no captures to this function. This is separable.

---
### Generic lambdas

In C++20, regular functions are able to use `auto` for parameters but lambda parameters can use `auto` since C++14. When a lambda has one or more `auto` parameter, the compiler will infer what parameter types are needed from the calls to the lambda.  Because lambdas with one or more `auto` parameter can work with a wide variety of types, they are called **generic lambdas**.

When used in the context of a lambda, `auto` is just a shorthand for a template parameter.
Let’s take a look at a generic lambda:

```cpp
int main() {
  constexpr std::array months{ // pre-C++17 use std::array<const char*, 12>
    "January", "February", "March", "April", "May", "June",
    "July", "August", "September", "October", "November", "December"
  };

  // Search for two consecutive months that start with the same letter.
  const auto sameLetter{ std::adjacent_find(months.begin(), months.end(),
                                      [](const auto& a, const auto& b) {
                                        return a[0] == b[0];
                                      })};
  if (sameLetter != months.end()) {
    std::cout << *sameLetter << " and " << *std::next(sameLetter);
  } // June and July
}
```

If you wrote a normal, non-generic lambda like this:

```cpp
[](const std::string& a, const std::string& b) { 
    return a[0] == b[0]; 
}
```

The compiler translates it into something like this:

```cpp
struct CompilerGeneratedName {
    bool operator()(const std::string& a, const std::string& b) const {
        return a[0] == b[0];
    }
};
```

A generic lambda:

```cpp
[](const auto& a, const auto& b) { 
    return a[0] == b[0]; 
}
```

is translated by the compiler into something like this:

```cpp
struct CompilerGeneratedName {
    template <typename T1, typename T2>
    bool operator()(const T1& a, const T2& b) const {
        return a[0] == b[0];
    }
};
```


In C++20, this was expanded to allow explicit template parameters on lambdas, giving you strict control over the types (e.g., forcing two parameters to be the exact same type).

```cpp
// C++20 Template Lambda
auto addSameType = []<typename T>(T a, T b) {
    return a + b;
};
```

---
###  Constexpr lambdas

As of C++17, lambdas are implicitly constexpr if the result satisfies the requirements of a constant expression. This generally requires two things:
- The lambda must either have no captures, or all captures must be constexpr.
- The functions called by the lambda must be constexpr. Note that many standard library algorithms and math functions weren’t made constexpr until C++20 or C++23.

---
### Generic lambdas and static variables

We discussed that when a function template contains a static local variable, each function instantiated from that template will receive its own independent static local variable. 

Generic lambdas work the same way: a unique lambda will be generated for each different type that `auto` resolves to.

The following example shows how one generic lambda turns into two distinct lambdas:

```cpp
int main() {
  // Print a value and count how many times @print has been called.
  auto print{
    [](auto value) {
      static int callCount{ 0 };
      std::cout << callCount++ << ": " << value << '\n';
    }
  };
  print("hello"); // 0: hello
  print("world"); // 1: world
  print(1); // 0: 1
  print(2); // 1: 2
  print("ding dong"); // 2: ding dong
}
```

To have a shared counter between the two generated lambdas, we’d have to define a global variable or a `static` local variable outside of the lambda. As you know from previous lessons, both global and static local variables can cause problems and make it more difficult to understand code. We’ll be able to avoid those variables after talking about lambda captures in the next lesson.

---
### Return type deduction

If return type deduction is used, a lambda’s return type is deduced from the `return` statements inside the lambda, and all return statements in the lambda must return the same type (otherwise the compiler won’t know which one to prefer).

In the case where we’re returning different types, we have two options:
1. Do explicit casts to make all the return types match, or
2. explicitly specify a return type for the lambda, and let the compiler do implicit conversions.

The second case is usually the better choice because if you ever decide to change the return type, you (usually) only need to change the lambda’s return type, and not touch the lambda body.

---
### Capture Clauses and Capture By Value

Out of the objects that have been defined outside the lambda, it can access only the objects with static storage duration (e.g. global variables and static locals) or which are constexpr or variables that can be used in constant expressions like `const int x{}`.

The **capture clause** is used to (indirectly) give a lambda access to variables available in the surrounding scope that it normally would not have access to. All we need to do is list the entities we want to access from within the lambda as part of the capture clause.

When the compiler encounters a lambda definition, it creates a custom object definition for the lambda. Each captured variable becomes a data member of the object. At runtime, when the lambda definition is encountered, the lambda object is instantiated, and the members of the lambda are initialized at that point.

When a lambda is called, `operator()` is invoked. By default, this `operator()` treats captures as const, meaning the lambda is not allowed to modify those captures.

In the following example, we capture the variable `ammo` and try to decrement it.

```cpp
int ammo{ 10 };
auto shoot { [ammo]() {
	--ammo; // Illegal, ammo cannot be modified.
    std::cout << "Pew! " << ammo << " shot(s) left.\n";
}};
shoot();
std::cout << ammo << " shot(s) left\n";
```

The above won’t compile, because `ammo` is treated as const within the lambda.
To allow modifications of variables that were captured, we can mark the lambda as `mutable` (it removes the const qualifier from `operator()`). Also it will apply to all the captures, we can't do it selectively

```cpp
int ammo{ 10 };
auto shoot{ [ammo]() mutable { 
	--ammo; // ok now
	std::cout << "Pew! " << ammo << " shot(s) left.\n";
}};
shoot(); // 9
shoot(); // 8
std::cout << ammo << " shot(s) left\n"; // 10
```

We can also capture variables by reference to allow the lambda to change the value of the argument. Variables that are captured by reference are non-const, unless the variable they’re capturing is `const`.  Here’s the above code with `ammo` captured by reference:

```cpp
int ammo{ 10 };

auto shoot { [ammo]() {
	--ammo; // Illegal, ammo cannot be modified.
    std::cout << "Pew! " << ammo << " shot(s) left.\n";
}};

shoot(); // 9
std::cout << ammo << " shot(s) left\n"; // 9
```

To count how many comparisons `std::sort` makes we can pass the below comparator to it.

```cpp
int comparisons{ 0 };
auto lam = [&comparisons](const auto& a, const auto& b) {
  ++comparisons;
  return a < b;
});
```

Multiple variables can be captured by separating them with a comma. This can include a mix of variables captured by value or by reference:

```cpp
int health{ 33 };
int armor{ 100 };
std::vector<CEnemy> enemies{};
[health, armor, &enemies](){};
```

---
### Default Captures

A **default capture** (also called a **capture-default**) captures all variables that are mentioned in the lambda. Variables not mentioned in the lambda are not captured.

To capture all used variables by value, use a capture value of `=`.  
To capture all used variables by reference, use a capture value of `&`.

Default captures can be mixed with normal captures. We can capture some variables by value and others by reference, but each variable can only be captured once.

```cpp
int health{ 33 };
int armor{ 100 };
std::vector<CEnemy> enemies{};

// Capture health and armor by value, and enemies by reference.
[health, armor, &enemies](){};

// Capture enemies by reference and everything else by value.
[=, &enemies](){};

// Capture armor by value and everything else by reference.
[&, armor](){};

// Illegal, we already said we want to capture everything by reference.
[&, &armor](){};

// Illegal, we already said we want to capture everything by value.
[=, armor](){};

// Illegal, armor appears twice.
[armor, &health, &armor](){};

// Illegal, the default capture has to be the first element in the capture group.
[armor, &](){};
```

---
### Declaring new variables inside capture

Sometimes we want to capture a variable with a slight modification or declare a new variable that is only visible in the scope of the lambda. We can do so by defining a variable in the lambda-capture without specifying its type (auto deduction). In-fact, you cannot give the type at all but you can use suffixes or `static_cast` so that auto deduces the desired type.

```cpp
[userArea{ width * height }](int knownArea) { return userArea == knownArea; })};
```

It is essential for moving move-only objects (like `std::unique_ptr` or `std::promise`) into a lambda, because you cannot copy them.

---
### Dangling captured references

Variables are captured at the point where the lambda is defined. If a variable captured by reference dies before the lambda, the lambda will be left holding a dangling reference.

For example:

```cpp
// returns a lambda
auto makeWalrus(const std::string& name) {
	return [&]() { // Capture name by reference and return the lambda.
		std::cout << "I am a walrus, my name is " << name << '\n'; 
	    // Undefined behavior
	};
}

int main() {
  // Create a new walrus whose name is Roofus.
  // sayName is the lambda returned by makeWalrus.
  auto sayName{ makeWalrus("Roofus") };
  // Call the lambda function that makeWalrus returned.
  sayName();
}
```

The call to `makeWalrus()` creates a temporary `std::string` from the string literal `"Roofus"`. The lambda in `makeWalrus()` captures the temporary string by reference. The temporary string dies at the end of the full expression containing the call to `makeWalrus()`, but the lambda `sayName` still references it past that point. Thus, when we call `sayName`, the dangling reference is accessed, causing undefined behavior.

Note that this also happens if `"Roofus"` is passed to `makeWalrus()` by value. Parameter `name` dies at the end of `makeWalrus()`, and the lambda is left holding a dangling reference.

---
### Unintended copies of mutable lambdas 

Because lambdas are objects, they can be copied. This can cause problems. Example:

```cpp
void myInvoke(const std::function<void()>& fn) { fn(); }

int main() {
    int i{ 0 };
    auto count{ [i]() mutable {
      std::cout << ++i << '\n';
    }};
    myInvoke(count); // 1
    myInvoke(count); // 1
    myInvoke(count); // 1
}
```

When we call `myInvoke(count)`, the compiler will see that `count` ( lambda type) doesn’t match the type of the reference parameter type (`std::function<void()>`). It will convert the lambda into a temporary `std::function` so that the reference parameter can bind to it, and this will make a copy of the lambda. Thus, our call to `fn()` is actually being executed on the copy of our lambda that exists as part of the temporary `std::function`, not the actual lambda.

If we need to pass a mutable lambda, and want to avoid the possibility of inadvertent copies being made, there are two options

One option is to use a non-capturing lambda instead and track our state using a static local variable instead : 
```cpp
void myInvoke(const std::function<void()>& fn) { fn(); }

int main() {
    auto count { []() {
        static int i { 0 }; 
        std::cout << ++i << ' ';
    } };

    myInvoke(count); // 1
    myInvoke(count); // 2
    myInvoke(count); // 3

    return 0;
}
```

The compiler places `i` in the permanent **.bss / .data segment** of your RAM.
So now the lambda object is now stateless. When `myInvoke` creates a temporary copy of the lambda as a `std::function`, it just copies an empty object. When the copy executes, it talks to the exact same permanent `static int i` in the data segment as the original lambda.

But static local variables can be difficult to keep track of and make our code less readable. 
A better option is to prevent copies of our lambda from being made in the first place.

One option is to put our lambda into a `std::function` immediately. That way, when we call `myInvoke()`, the reference parameter `fn` can bind to our `std::function`, and no temporary copy is made:

```cpp
void myInvoke(const std::function<void()>& fn) { fn(); }

int main() {
	int i{ 0 };
    std::function count{ [i]() mutable { 
    // lambda object stored in a std::function
      std::cout << ++i << '\n';
    }};
    myInvoke(count); // 1
    myInvoke(count); // 2
    myInvoke(count); // 3
}
```

An alternate solution is to use a reference wrapper. By wrapping our lambda in a `std::reference_wrapper`, whenever anybody tries to make a copy of our lambda, they’ll make a copy of the reference_wrapper instead (avoiding making a copy of the lambda):

```cpp
void myInvoke(const std::function<void()>& fn) { fn(); }

int main() {
    int i{ 0 };
    auto count{ [i]() mutable {
      std::cout << ++i << '\n';
    }};
    // std::ref(count) ensures count is treated like a reference
    // thus, anything that tries to copy count will actually copy the reference
    // ensuring that only one count exists
    myInvoke(std::ref(count)); // 1
    myInvoke(std::ref(count)); // 2
    myInvoke(std::ref(count)); // 3
}
```

This method works even if `myInvoke` takes `fn` by value (instead of by reference).

Try to avoid mutable lambdas. Non-mutable lambdas are easier to understand and don’t suffer from the above issues, as well as more dangerous issues that arise when you add parallel execution.

---








