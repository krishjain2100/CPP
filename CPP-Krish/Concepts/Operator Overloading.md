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

There are three different ways to overload operators: the member function way, the friend function way, and the normal function way

---
### Overloading operators using friend functions

The following example shows how to overload operator plus (+) in order to add two “Cents” objects together:

```cpp
class Cents {
private:
	int m_cents {};
public:
	Cents(int cents) : m_cents{ cents } { }
	// add Cents + Cents using a friend function
	friend Cents operator+(const Cents& c1, const Cents& c2);
	int getCents() const { return m_cents; }
};

// note: this function is not a member function!
Cents operator+(const Cents& c1, const Cents& c2) {
	return c1.m_cents + c2.m_cents;
}

int main() {
	Cents cents1{ 6 };
	Cents cents2{ 8 };
	Cents centsSum{ cents1 + cents2 };
	std::cout << "I have " << centsSum.getCents() << " cents.\n";
}
```


When C++ evaluates the expression `x + y`, x becomes the first parameter, and y becomes the second parameter. When the operands have different types, x + y does not call the same function as y + x.

```cpp
class Cents {
private:
	int m_cents {};
public:
	explicit Cents(int cents) : m_cents{ cents } { }
	friend Cents operator+(const Cents& c1, int value) {
		return Cents { c1.m_cents + value };
	}
	friend Cents operator+(int value, const Cents& c1) {
		return Cents { c1.m_cents + value };
	}
	int getCents() const { return m_cents; }
};

int main() {
	Cents c1{ Cents{ 4 } + 6 };
	Cents c2{ 6 + Cents{ 4 } };
	std::cout << "I have " << c1.getCents() << " cents.\n";
	std::cout << "I have " << c2.getCents() << " cents.\n";
}
```

It possible to define overloaded operators by calling other overloaded operators. You should do so if and when doing so produces simpler code. In cases where the implementation is trivial (e.g. a single line) it may or may not be worth doing this.

---
### Overloading operators using normal functions

Using a friend function to overload an operator is convenient because it gives you direct access to the internal members of the classes you’re operating on. However, if you don’t need that access, you can write your overloaded operators as normal functions. Note that the Cents class above contains an access function (`getCents()`) that allows us to get at `m_cents` without having to have direct access to private members. Because of this, we can write our overloaded operator+ as a non-friend


```cpp
class Cents {
private:
  int m_cents{};
public:
  Cents(int cents) : m_cents{ cents } {}
  int getCents() const { return m_cents; }
};

// note: this function is not a member function nor a friend function!
Cents operator+(const Cents& c1, const Cents& c2) {
  return Cents{ c1.getCents() + c2.getCents() };
}

int main() {
  Cents cents1{ 6 };
  Cents cents2{ 8 };
  Cents centsSum{ cents1 + cents2 };
  std::cout << "I have " << centsSum.getCents() << " cents.\n";
}
```

The one difference is that the friend function declaration inside the class serves as a prototype as well. With the normal function version, you’ll have to provide your own function prototype in the header file outside the class and then implemented in a separate file.

**Best practice:** 
Prefer overloading operators as normal functions instead of friends if it’s possible to do so without adding additional functions. You don't want more functions touching your internals but also don't want to make more access functions.

---
### Overloading the I/O operators

**Overloading `operator<<`**

The left operand is the `std::cout` object, and the right operand is your class object. `std::cout` is actually an object of type `std::ostream`. Therefore, our overloaded function will look like this:

```cpp
// std::ostream is the type for object std::cout
friend std::ostream& operator<< (std::ostream& out, const Point& point);
```

Example:

```cpp

class Point {
private:
    double m_x{}, m_y{}, m_z{};
public:
    Point(double x=0.0, double y=0.0, double z=0.0) : m_x{x}, m_y{y}, m_z{z} {}
    friend std::ostream& operator<< (std::ostream& out, const Point& point);
};

std::ostream& operator<< (std::ostream& out, const Point& p) {
    out << "Point(" << p.m_x << ", " << p.m_y << ", " << p.m_z << ')'; 
    return out; // return std::ostream so we can chain calls to operator<<
}

int main() {
    const Point point1 { 2.0, 3.0, 4.0 };
    std::cout << point1 << '\n';
}
```

If you try to return `std::ostream` by value, you’ll get a compiler error. This happens because `std::ostream` specifically disallows being copied. In this case, we return the left hand parameter as a reference. 

Returning the left-hand parameter by reference is okay in this case, since the left-hand parameter was passed in by the calling function, it must still exist when the called function returns. Therefore, we don’t have to worry about referencing something that will go out of scope and get destroyed when the operator returns.

In the above example, `operator<<` is a friend because it needs direct access to the member of `Point`. However, if the members could be accessed via getters, then `operator<<` could be implemented as a non-friend.

**Overloading `operator>>`**

```cpp
class Point {
private:
    double m_x{}, m_y{}, m_z{};
public:
    Point(double x=0.0, double y=0.0, double z=0.0) : m_x{x}, m_y{y}, m_z{z} {}
    friend std::ostream& operator<< (std::ostream& out, const Point& point);
    friend std::istream& operator>> (std::istream& out, Point& point);
};

std::ostream& operator<< (std::ostream& out, const Point& p) {
    out << "Point(" << p.m_x << ", " << p.m_y << ", " << p.m_z << ')'; 
    return out; // return std::ostream so we can chain calls to operator<<
}

// note that point must be non-const so we can modify the object
std::istream& operator>> (std::istream& in, Point& point) {
    // This version subject to partial extraction issues (see below)
    in >> point.m_x >> point.m_y >> point.m_z;
    return in;
}

int main() {
    std::cout << "Enter a point: ";
    Point point{ 1.0, 2.0, 3.0 }; // non-zero test data
    std::cin >> point;
    std::cout << "You entered: " << point << '\n';
}
```

When the user enters `4.0 5.6 7.26` as input, ouput:
You entered: `Point(4, 5.6, 7.26)`
When the user enters `4.0b 5.6 7.26` as input, output: (Note use of b after 4.0)
You entered: Point(4, 0, 3)

Our point is now a weird hybrid consisting of one value from the user’s input (`4.0`), one value that has been zero-initialized (`0.0`), and one value that was untouched by the input function (`3.0`).

Guarding against partial extraction

When we’re extracting a single value, there are only two possible outcomes: the extraction fails, or it is successful. However, when we’re extracting more than one value as part of an input operation, things get a bit more complicated.

The above implementation of `operator>>` can result in a partial extraction. And this is exactly what we’re seeing with input `4.0b 5.6 7.26`. The extraction to `x_y` successfully extracts `4.0` from the user’s input, leaving `b 5.6 7.26` in the input stream. The extraction to `m_y` fails to extract `b`, so `m_y` is copy-assigned the value `0.0` and the input stream is set to failure mode. Since we haven’t cleared failure mode, the extraction to `m_z` aborts immediately, and the value that `m_z` had before the extraction attempt remains (`3.0`).

There is no case where this is a desirable outcome. And in some cases, it might even be actively dangerous. Imagine we were writing an `operator>>` for a `Fraction` object instead. After successfully extracting the numerator, a failed extraction to the denominator would set the denominator to `0.0`, which might later cause a divide by zero and cause the application to crash.

So how might we avoid this? One way is to make our operations transactional. A **transactional operation** must either completely succeed or completely fail -- no partial successes or failures are allowed. This is sometimes known as “all or nothing”. If a failure occurs at any point during the transaction, prior changes made by the operation must be undone.

Key insight

Transactions occur all the time in real life. Consider the case where I want to transfer money from one bank account to another. This requires two steps: First the money must first be deducted from one account, and then it must be credited to the other account. In the execution of this operation, there are three possibilities:

- The deduction step fails (e.g. not enough funds). The transaction fails, and neither account balance reflects the transfer.
- The crediting step fails (e.g. due to a technical problem). In this case, the deduction (which has already succeeded) must be undone. The transaction fails, and neither account balance reflects the transfer.
- Both steps succeed. The transaction is successful, and both account balances reflect the transfer.

The end result is that there are only two possible outcomes: the transfer fully fails and the account balances are unchanged, or the transfer succeeds and the account balances are both changed.

Let’s reimplement our overloaded `Point` `operator>>` as a transactional operation:

```cpp
// note that point must be non-const so we can modify the object
// note that this implementation is a non-friend
std::istream& operator>> (std::istream& in, Point& point)
{
    double x{};
    double y{};
    double z{};

    if (in >> x >> y >> z)      // if all extractions succeeded
        point = Point{x, y, z}; // overwrite our existing point

    return in;
}
```

In this implementation, we’re not overwriting the data members directly with the user’s input. Instead, we’re extracting the user’s input to temporary variables (`x`, `y`, and `z`). Once all extraction attempts have completed, we check whether all extractions were successful. If so, then we update all the members of `Point` together. Otherwise, we do not update any of them.

![Ezoic](https://go.ezodn.com/utilcave_com/ezoic.png "ezoic")

Tip

`if (in >> x >> y >> z)` is equivalent to `in >> x >> y >> z; if (in)`. Remember, each extraction returns `in` so that multiple extractions can be chained together. The single-statement version uses the `in` returned from the last extraction as the condition of the if-statement, whereas the multi-statement version uses `in` explicitly.

Tip

Transactional operations can be implemented using a number of different strategies. For example:

- Alter on success: Store the result of each sub-operation. If all sub-operations succeed, replace the relevant data with the stored results. This is the strategy we use in the `Point` example above.
- Restore on failure: Copy any data that can be altered. If any sub-operation fails, the changes made by prior sub-operations can be reverted using the data from copy.
- Rollback on failure: If any sub-operation fails, each prior sub-operation is reversed (using an opposite sub-operation). This strategy is often used in databases, where the data is too large to back up, and the result of sub-operations can’t be stored.

While the above `operator>>` prevents partial extractions, it is inconsistent with how `operator>>` works for fundamental types. When extraction to an object with a fundamental type fails, the object isn’t left unaltered -- it is copy assigned the value `0` (this ensures the object has some consistent value in case it wasn’t initialized before the extraction attempt). Therefore, for consistency, you may wish to have a failed extraction reset the object to its default state (at least in cases where such a thing exists).

Here’s an alternate version of `operator>>` that resets `Point` to its default state if any extraction fails:

```cpp
// note that point must be non-const so we can modify the object
// note that this implementation is a non-friend
std::istream& operator>> (std::istream& in, Point& point)
{
    double x{};
    double y{};
    double z{};

    in >> x >> y >> z;
    point = in ? Point{x, y, z} : Point{};

    return in;
}
```

Author’s note

Such an operation is technically no longer transactional (because failure doesn’t “do nothing”). There doesn’t appear to be a general term for operations that guarantee no partial results. Perhaps “indivisible operation”.

Handling semantically invalid input

Extraction can fail in different ways.

In cases where `operator>>` simply fails to extract anything to a variable, `std::cin` will automatically be placed in failure mode (which we discuss in lesson [9.5 -- std::cin and handling invalid input](https://www.learncpp.com/cpp-tutorial/stdcin-and-handling-invalid-input/)). The caller of this function can then check `std::cin` to see if it failed and handle that case as appropriate.

But what about cases where the user inputs a value that is extractable but semantically invalid (e.g. a `Fraction` with a denominator of `0`)? Because `std::cin` did extract something, it won’t go into failure mode automatically. And then the caller probably won’t realize something went wrong.


To address this, we can have our overloaded `operator>>` determine whether any of the values that were extracted are semantically invalid, and if so, manually put the input stream in failure mode. This can be done by calling `std::cin.setstate(std::ios_base::failbit);`.

Here’s an example of a transactional overloaded `operator>>` for `Point` that will cause the input stream to enter failure mode if the user inputs an extractable negative value:

```cpp
std::istream& operator>> (std::istream& in, Point& point)
{
    double x{};
    double y{};
    double z{};

    in >> x >> y >> z;
    if (x < 0.0 || y < 0.0 || z < 0.0)       // if any extractable input is negative
        in.setstate(std::ios_base::failbit); // set failure mode manually
    point = in ? Point{x, y, z} : Point{};

    return in;
}
```

Conclusion

Overloading `operator<<` and `operator>>` make it easy to output your class to screen and accept user input from the console.

---
### Overloading operators using member functions

 When overloading an operator using a member function:
- The overloaded operator must be added as a member function of the left operand.
- The left operand becomes the implicit *this object
- All other operands become function parameters.

Example:

```cpp

class Cents {
private:
    int m_cents {};
public:
    Cents(int cents) : m_cents { cents } { }
    // Overload Cents + int
    Cents operator+(int value) const;
    int getCents() const { return m_cents; }
};

Cents Cents::operator+ (int value) const {
    return Cents { m_cents + value };
}

int main() {
	const Cents cents1 { 6 };
	const Cents cents2 { cents1 + 2 };
}
```

In the member function version, the expression `cents1 + 2` becomes function call `cents1.operator+(2)`. The compiler implicitly converts an object prefix into a hidden leftmost parameter named `*this`. So in actuality, `cents1.operator+(2)` becomes `operator+(&​cents1, 2)`, which is almost identical to the friend version.

**Not everything can be overloaded as a friend function**:

The assignment `(=),` subscript (`[]`), function call (`()`), and member selection (`->`) operators must be overloaded as member functions, because the language requires them to be.

**Not everything can be overloaded as a member function**:

We are not able to overload operator<< as a member function because the left operand is an object of type `std::ostream`. `std::ostream` is fixed as part of the standard library. We can’t modify the class declaration to add the overload as a member function of `std::ostream`.
This necessitates that operator<< be overloaded as a normal function (preferred) or a friend.

Similarly, although we can overload operator+(Cents, int) as a member function, we can’t overload operator+(int, Cents) as a member function.

**When to use a normal, friend, or member function overload**:
In most cases, the language leaves it up to you to determine whether you want to use the normal/friend or member function version of the overload. However, one of the two is usually a better choice than the other.

- When dealing with binary operators that don’t modify the left operand (e.g. operator+), the normal or friend function version is typically preferred, because it works for all parameter types (even when the left operand isn’t a class object, or is a class that is not modifiable). The normal or friend function version has the added benefit of “symmetry”, as all operands become explicit parameters.

- When dealing with binary operators that do modify the left operand (e.g. operator+=), the member function version is typically preferred. In these cases, the leftmost operand will always be a class type, and having the object being modified become the one pointed to by *this is natural. Because the rightmost operand becomes an explicit parameter, there’s no confusion over who is getting modified and who is getting evaluated.

---
### Overloading unary operators

The positive (+), negative (-) and logical not (!) operators all are unary operators, which means they only operate on one operand. Because they only operate on the object they are applied to, typically unary operator overloads are implemented as member functions. 

```cpp
class Cents {
private:
    int m_cents {};
public:
    Cents(int cents): m_cents{cents} {}
    Cents operator-() const;
    int getCents() const { return m_cents; }
};

Cents Cents::operator-() const {
    return -m_cents; // since return type is a Cents, this does an implicit conversion from int to Cents using the Cents(int) constructor
}

int main() {
    const Cents nickle{ 5 };
    std::cout << (-nickle).getCents() << " cents\n";
}
```

---
### Overloading the comparison operators

Because the comparison operators are all binary operators that do not modify their left operands, we will make our overloaded comparison operators friend functions.


```cpp
class Car {
private:
    std::string m_make;
    std::string m_model;
public:
    Car(string_view make, string_view model) : m_make{make}, m_model{model} {}
    friend bool operator== (const Car& c1, const Car& c2);
    friend bool operator!= (const Car& c1, const Car& c2);
};

bool operator== (const Car& c1, const Car& c2) {
    return (c1.m_make == c2.m_make && c1.m_model == c2.m_model);
}
bool operator!= (const Car& c1, const Car& c2) {
    return (c1.m_make != c2.m_make || c1.m_model != c2.m_model);
}
₹
int main() {
    Car corolla{ "Toyota", "Corolla" };
    Car camry{ "Toyota", "Camry" };
    if (corolla == camry)
        std::cout << "a Corolla and Camry are the same.\n";
    if (corolla != camry)
        std::cout << "a Corolla and Camry are not the same.\n";
}
```


Some of the container classes in the standard library (classes that hold sets of other classes) require an overloaded operator< so they can keep the elements sorted. So if you overload it then you can use `std::sort` directly on a `std::vector<YourClass>` without writing custom comparator.

Overloaded comparison operators tend to have a high degree of redundancy, and the more complex the implementation, the more redundancy there will be.

Fortunately, many of the comparison operators can be implemented using the other comparison operators:
- operator!= can be implemented as `!(operator==)`
- operator> can be implemented as operator< with the order of the parameters flipped
- operator>= can be implemented as !(operator<)
- operator<= can be implemented as !(operator>)

This means that we only need to implement logic for operator== and operator<, and then the other four comparison operators can be defined in terms of those two.

#### The spaceship operator `<=>`

C++20 introduces the spaceship operator (`operator<=>`), which allows us to reduce the number of comparison functions we need to write down to 2 at most, and sometimes just 1. 
This is a little complex and requires an additional lesson which will be added later

---
### Overloading the increment and decrement operators

Because the increment and decrement operators are both unary operators and they modify their operands, they’re best overloaded as member functions. 

There are actually two versions of the increment and decrement operators: a prefix increment and decrement (e.g. `++x; --y;`) and a postfix increment and decrement (e.g. `x++; y--;`).

But both have the same name (eg. operator++), are unary, and take one parameter of the same type. How to distinguish between prefix and postfix overload ? (Note that different return type doesn't help in distinguishing, you will see why they will have different return types)

The C++ language specification has a special case for this:  
- If the overloaded operator has an int parameter, the operator is a postfix overload. 
- If the overloaded operator has no parameter, the operator is a prefix overload.


```cpp
class Digit {
private:
    int m_digit{};
public:
    Digit(int digit=0) : m_digit{digit} {}
    Digit& operator++(); // prefix has no parameter
    Digit& operator--(); // prefix has no parameter
    Digit operator++(int); // postfix has an int parameter
    Digit operator--(int); // postfix has an int parameter
    friend std::ostream& operator<< (std::ostream& out, const Digit& d);
};

Digit& Digit::operator++() {
    if (m_digit == 9) m_digit = 0;
    else ++m_digit;
    return *this;
}
Digit& Digit::operator--() {
    if (m_digit == 0) m_digit = 9;
    else --m_digit;
    return *this;
}
Digit Digit::operator++(int) {
    // Create a temporary variable with our current digit
    Digit temp{*this};
    ++(*this); // Use prefix operator to increment this digit
    return temp; // return saved state
}
Digit Digit::operator--(int) {
    Digit temp{*this};
    --(*this);
    return temp; 
}
std::ostream& operator<< (std::ostream& out, const Digit& d) {
	out << d.m_digit;
	return out;
}

int main() {
    Digit digit { 5 };
    std::cout << digit;
    std::cout << ++digit; // calls Digit::operator++();
    std::cout << digit++; // calls Digit::operator++(int);
    std::cout << digit;
    std::cout << --digit; // calls Digit::operator--();
    std::cout << digit--; // calls Digit::operator--(int);
    std::cout << digit;
}
```

- We return`*this`  so multiple operators can be chained together.
- The postfix operators needs to return the state of the object _before_ it is incremented or decremented.  So we use a temporary variable that holds the value of the object before it is incremented or decremented. Note that this means the return value of the overloaded operator must be a non-reference, because we can’t return a reference to a local variable that will be destroyed when the function exits. Also note that this means the postfix operators are typically less efficient because of the added overhead of instantiating a temporary variable and returning by value instead of reference.
- We’ve written the post-increment and post-decrement in such a way that it calls the pre-increment and pre-decrement to do most of the work. 

---
### Overloading the subscript operator

The subscript operator is one of the operators that must be overloaded as a member function. An overloaded `operator[]` function will always take one parameter: the subscript that the user places between the hard braces. 

```cpp
class IntList {
private:
    int m_list[10]{};
public:
    int& operator[] (int index) { return m_list[index]; }
};
int main() {
    IntList list{};
    list[2] = 3; // set a value
    std::cout << list[2] << '\n'; // get a value
}
```

Although you can provide a default value for the function parameter (compiles fine), actually using `operator[]` without a subscript inside is not considered a valid syntax (compile error), so there’s no point.

Also, C++23 adds support for overloading operator[] with multiple subscripts. This means that you can put multiple comma-separated arguments directly inside the square brackets, which is incredibly useful for things like 2D grids, 3D matrices, or multidimensional arrays.

Consider what would happen if operator[] returned an integer by value instead of by reference.  If `m_list[2]` had the value of 6, operator[] would return the value 6. `list[2] = 3` would partially evaluate to `6 = 3`and the compiler will complain: left operand must be lvalue.

We can define a non-const and a const version of operator[] separately. The non-const version will be used with non-const objects, and the const version with const-objects.

```cpp
class IntList {
private:
    int m_list[10]{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 }; 
public:
    int& operator[] (int index) { return m_list[index]; }
    // For const objects: can only be used for access
    // This function could also return by value if the type is cheap to copy since it can't be assigned to
    const int& operator[] (int index) const { return m_list[index];}
};

int main() {
    IntList list{};
    list[2] = 3; // okay: calls non-const version of operator[]
    std::cout << list[2] << '\n';
    const IntList clist{};
    // clist[2] = 3; // compile error
    std::cout << clist[2] << '\n';
}
```

In the above example, note that the implementations of `int& IntList::operator[](int)` and `const int& IntList::operator[](int) const` are identical and the only difference is the return type of the function.

Validating the index requires adding many redundant lines of code to each function. Normally we’d simply implement one function in terms of the other (e.g. have one function call the other). But that’s a bit tricky in this case. The const version of the function can’t call the non-const version of the function, because that would require discarding the const of a const object. And while the non-const version of the function can call the const version of the function, the const version of the function returns a const reference, when we need to return a non-const reference. Fortunately, there is a way to work around this.

The preferred solution is as follows:
- Implement the logic for the const version of the function.
- Have the non-const function call the const function and use `const_cast` to remove the const.

The resulting solution looks something like this:

```cpp
int& operator[] (int index) {
	// use std::as_const to get a const version of `this` (as a reference)
	// so we can call the const version of operator[]
	// then const_cast to discard the const on the returned reference
	return const_cast<int&>(std::as_const(*this)[index]);
}
const int& operator[] (int index) const { return m_list[index]; }
```

In C++23, we can do even better by making use of several features we don’t yet cover in this tutorial series: (**DID NOT UNDERSTAND**)

```cpp
// Use an explicit object parameter (self) and auto&& to differentiate const vs non-const
auto&& operator[](this auto&& self, int index) {
	// Complex code goes here
	return self.m_list[index];
}
```


One advantage is that we can implement index validation which does not happen automatically in standard cases. We can use `assert`. But if you don’t want to use an `assert` (which will be compiled out of a non-debug build) you can instead use an if-statement and your favorite error handling method (e.g. throw an exception, call `std::exit`, etc…):

**Pointers to objects and overloaded operator[] don’t mix (unlike C-style array pointers):**
If you try to call operator[] on a pointer to an object, C++ will assume you’re trying to index an array of objects of that type.  The proper syntax would be to dereference the pointer first .

**The function parameter does not need to be an integral type:**
As mentioned above, C++ passes what the user types between the hard braces as an argument to the overloaded function. In most cases, this will be an integral value. However, this is not required and can be of any type

---
### Overloading the parenthesis operator

The parenthesis operator (operator()) is interesting operator in that it allows you to vary both the type AND number of parameters it takes.

The parenthesis operator must be implemented as a member function. 
In non-object-oriented C++, the () operator is used to call functions.
In the case of classes, operator() is just a normal operator that calls a function like any other overloaded operator.

Because the () operator can take as many parameters as we want it to have, we can declare a version of operator() that takes two integer index parameters, and use it to access our two-dimensional array.

Note: As of C++23, you can use `operator[]` with multiple indices. This works just like `operator()` does

Operator() is commonly overloaded to implement **functors** (or **function object**), which are classes that operate like functions. The advantage of a functor over a normal function is that functors can store data in member variables (since they are classes). Example:

```cpp

class Accumulator {
private:
    int m_counter{ 0 };
public:
    int operator() (int i) { return (m_counter += i); }
    void reset() { m_counter = 0; } 
};

int main() {
    Accumulator acc{};
    std::cout << acc(1) << '\n'; // prints 1
    std::cout << acc(3) << '\n'; // prints 4
    Accumulator acc2{};
    std::cout << acc2(10) << '\n'; // prints 10
    std::cout << acc2(20) << '\n'; // prints 30
}
```

Functors can also have other member functions (e.g. `reset()`) that do convenient things.

---
### Overloading typecasts

We had shown how we can use a converting constructor to create a class type object from another type of object. But this only works if the destination type is a class type that can be modified to add such a constructor.
Previously, we had defined a `Cents` class. If we can convert an `int` into a `Cents` (via a constructor), then we might also want to provide a way to convert a `Cents` back into an `int`.

One not-great way is to use a conversion function. 

```cpp
void printInt(int value) { std::cout << value; }
int main() {
    Cents cents{ 7 };
    printInt(cents.getCents()); // print 7
}
```

While this function produces the result we want, it’s not truly a conversion, as the compiler won’t understand that it should use this function for casts or implicit conversion. 

This is where overloading the typecast operators comes into play. Such a typecast can be used explicitly (via a cast) or implicitly by the compiler to perform conversions as needed.

```cpp
class Cents {
private:
    int m_cents{};
public:
    Cents(int cents=0) : m_cents{ cents } {}
    // Overloaded int cast
    operator int() const { return m_cents; }
    int getCents() const { return m_cents; }
    void setCents(int cents) { m_cents = cents; }
};
```

To do so, we’ve written a new overloaded operator named `operator int()`.
There are a few things worth noting here:
- Overloaded typecasts must be non-static members, and should be `const` so they can be used with const objects.
- Overloaded typecasts do not have explicit parameters, as there is no way to pass explicit arguments to them. They do still have a hidden `*this` parameter, pointing to the implicit object (which is the object to be converted).
- Overloaded typecast do not declare a return type. The name of the conversion (e.g. int) is used as the return type, as it is the only return type allowed. This prevents redundancy in the declaration.

Now in our example, we can call `printInt()` like this:

```cpp
int main() {
    Cents cents{ 7 };
    printInt(cents); // print 7
    std::cout << '\n';
}
```

Such typecasts can also be invoked explicitly via `static_cast`:

```cpp
std::cout << static_cast<int>(cents);
```

You can provide overloaded typecasts for any data type you wish, including your own program-defined data types.


Just like we can make constructors `explicit` so that they can’t be used for implicit conversions, we can also make our overloaded typecasts `explicit` for the same reason. Explicit typecasts can only be invoked by casting (e.g. `static_cast`) or by a form of direct initialization (either parenthesis or brace). They are not considered when doing copy-initialization.

Typecasts should be generally be marked as explicit. Exceptions can be made in cases where the conversion inexpensively converts to a similar user-defined type.

----
#### Difference between converting constructors and overloaded typecasts

Overloaded typecasts and converting constructors perform similar functions:
- A converting constructor is a member function of class type B that defines how B is created from A.
- An overloaded typecast is a member function of class type A that defines how A is converted to B.

The main difference between the two is whether A or B has ownership of how the conversion happens. Since both ways require defining a member function, they can only be used for class types that can be modified. If A is not a class type that can be modified, then you can’t use an overloaded typecast. If B is not a class type that can be modified, then you can’t use a converting constructor. If neither are class types that can be modified, then you will need to use a non-member conversion function instead.

In cases where A and B are both class types that can be modified, we could use either. But since we need only one, which should we prefer? In general, a converting constructor should be preferred to an overloaded typecast. It is cleaner for a class type to own its own construction, rather than rely on another class to create and initialize it.


There are a few cases where an overloaded typecast should be used instead:
- When providing a conversion to a fundamental type (can't change their constructors).
- When the conversion returns a reference or const reference (can't return from constructors)..
- When you do not want the type being constructed to be aware of the type being converted from. This can be helpful for avoiding circular dependencies.

For an example of that last bullet, `std::string` has a constructor to create a `std::string` from a `std::string_view`. This means `<string>` must include `<string_view>`. If `std::string_view` had a constructor to create a `std::string_view` from a `std::string`, then `<string_view>` would need to include `<string>`, and this would result in a circular dependency between headers.
Instead, `std::string` has an overloaded typecast that handles conversion from `std::string` to `std::string_view` (which is fine, since it’s already including `<string_view>`). `std::string_view` does not know about `std::string` at all, and thus does not need to include `<string>`. In this way, the circular dependency is avoided.


When a converting constructor and an overloaded typecast are both defined for the same conversion, both are considered during overload resolution. Depending on whether the overloaded typecast is const, the object being converted is const, and what type of cast or initialization is used (copy vs direct), either function might be chosen (which can result a typecast being selected over a converting constructor), or the result can be ambiguous (resulting in a compile error). For this reason, you should avoid defining both an overloaded typecast and a converting constructor that can serve the same conversion.

---
### Overloading the assignment operator

The **copy assignment operator** (operator=) is used to copy values from one object to another _already existing object_. The copy constructor initializes new objects, whereas the assignment operator replaces the contents of existing objects.

- If a new object has to be created before the copying can occur, the copy constructor is used (note: this includes passing or returning objects by value).
- If a new object does not have to be created before the copying can occur, the assignment operator is used.

The copy assignment operator must be overloaded as a member function.


```cpp
#include <cassert>
#include <iostream>

class Fraction
{
private:
	int m_n { 0 };
	int m_d { 1 };

public:
	Fraction(int n = 0, int d = 1 ) : m_n { n }, m_d { d } { assert(d != 0); }
	Fraction(const Fraction& copy) : m_n { copy.m_n}, m_d { copy.m_d } {
	 // copy must already be a valid Fraction so no 0 check required
		std::cout << "Copy constructor called\n"; // just to prove it works
	}

	// Overloaded assignment
	Fraction& operator= (const Fraction& fraction);
	friend std::ostream& operator<<(std::ostream& out, const Fraction& f1);

};

// A simplistic implementation of operator= (see better implementation below)
Fraction& Fraction::operator= (const Fraction& fraction) {
    m_numerator = fraction.m_numerator;
    m_denominator = fraction.m_denominator;
    return *this;
}

int main() {
    Fraction fiveThirds { 5, 3 };
    Fraction f;
    f = fiveThirds; // calls overloaded assignment
    std::cout << f;
}
```

This should all be pretty straightforward by now. Our overloaded operator= returns *this, so that we can chain multiple assignments together:

```cpp
int main() {
    Fraction f1 { 5, 3 };
    Fraction f2 { 7, 2 };
    Fraction f3 { 9, 5 };
    f1 = f2 = f3; // chained assignment // f1 = (f2 = f3)

// Resolved from right to left because the assignment operator (`=`) has right-to-left associativity.
}
```

C++ allows self-assignment: This will call `f1.operator=(f1)`, and under the simplistic implementation above, all of the members will be assigned to themselves. This just wastes time.

```cpp
Fraction f1 { 5, 3 };
f1 = f1; // self assignment
```

However, in cases where an assignment operator needs to dynamically assign memory, self-assignment can actually be dangerous:

```cpp
class MyString {
private:
	char* m_data {};
	int m_length {};
public:
	MyString(const char* data = nullptr, int length = 0 )
		: m_length { std::max(length, 0) } {
		if (length) {
			m_data = new char[static_cast<std::size_t>(length)];
			std::copy_n(data, length, m_data); 
			// copy length elements of data into m_data
		}
	}
	~MyString() { delete[] m_data;}
	MyString(const MyString&) = default; // some compilers (gcc) warn if you have pointer members but no declared copy constructor
	// Overloaded assignment
	MyString& operator= (const MyString& str);
};

// A simplistic implementation of operator= (do not use)
MyString& MyString::operator= (const MyString& str) {
	// if data exists in the current string, delete it
	if (m_data) delete[] m_data;
	m_length = str.m_length;
	m_data = nullptr;
	// allocate a new array of the appropriate length
	if (m_length) m_data = new char[static_cast<std::size_t>(str.m_length)];
	std::copy_n(str.m_data, m_length, m_data);
	 // copies m_length elements of str.m_data into m_data
	// return the existing object so we can chain this operator
	return *this;
}

int main() {
	MyString alex("Alex", 5); // Meet Alex
	MyString employee;
	employee = alex; // Alex is our newest employee
	std::cout << employee; // Prints Alex as it should
}
```

Now run the following program:

```cpp
int main() {
    MyString alex { "Alex", 5 }; // Meet Alex
    alex = alex; // Alex is himself
    std::cout << alex; // Garbage
}
```

In this case, m_data is allocated, so the assigment function deletes m_data. But because `str` is the same as `*this`, the string that we wanted to copy has been deleted and m_data (and str.m_data) are dangling. Fortunately, we can detect when self-assignment occurs

```cpp
if (this == &str) return *this;
```

Because this is just a pointer comparison, it should be fast, and does not require operator== to be overloaded.


Typically the self-assignment check is skipped for copy constructors. Because the object being copy constructed is newly created, the only case where the newly created object can be equal to the object being copied is when you try to initialize a newly defined object with itself:
In such cases, your compiler should warn you that `c` is an uninitialized variable.

```cpp
someClass c { c };
```

The self-assignment check may be omitted in classes that can naturally handle self-assignment. Consider this Fraction class assignment operator that has a self-assignment guard:

```cpp
Fraction& Fraction::operator= (const Fraction& fraction) {
    if (this == &fraction) return *this;
    m_numerator = fraction.m_numerator; // can handle self-assignment
    m_denominator = fraction.m_denominator; // can handle self-assignment
    return *this;
}
```

If the self-assignment guard did not exist, this function would still operate correctly during a self-assignment (because all of the operations done by the function can handle self-assignment properly).

**The copy and swap idiom**
A better way to handle self-assignment issues is via what’s called the copy and swap idiom. There’s a great writeup of how this idiom works [on Stack Overflow](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom).

**The implicit copy assignment operator**
Unlike other operators, the compiler will provide an implicit public copy assignment operator for your class if you do not provide a user-defined one. This assignment operator does member-wise assignment (which is essentially the same as the member-wise initialization that default copy constructors do).

Just like other constructors and operators, you can prevent assignments from being made by making your copy assignment operator private or using the delete keyword.

Note that if your class has const members, the compiler will instead define the implicit `operator=` as deleted. This is because const members can’t be assigned, so the compiler will assume your class should not be assignable. If you want a class with const members to be assignable (for all members that aren’t const), you will need to explicitly overload `operator=` and manually assign each non-const member.

---
### Shallow vs. deep copying

The default copy constructor and default assignment operators  use a copying method known as a member-wise copy (also known as a **shallow copy**). C++ copies each member of the class individually (using the assignment operator for overloaded operator=, and direct initialization for the copy constructor). 

However, when designing classes that handle dynamically allocated memory, member-wise  copying is bad because shallow copies of a pointer just copy the address of the pointer, so now they both point to the same memory.

A **deep copy** allocates memory for the copy and then copies the actual value, so that the copy lives in distinct memory from the source. 

Let’s go ahead and show how this is done for our MyString class:

```cpp
void MyString::deepCopy(const MyString& source) {
    // first we need to deallocate any value that this string is holding.
    delete[] m_data;
    // because m_length is not a pointer, we can shallow copy it
    m_length = source.m_length;
    // m_data is a pointer, so we need to deep copy it if it is non-null
    if (source.m_data) {
        // allocate memory for our copy
        m_data = new char[m_length];
        for (int i{ 0 }; i < m_length; ++i) m_data[i] = source.m_data[i];
    }
    else m_data = nullptr;
}

MyString::MyString(const MyString& source) {
    deepCopy(source);
}
```

Now let’s do the overloaded assignment operator:

```cpp
// Assignment operator
MyString& MyString::operator=(const MyString& source) {
    // check for self-assignment
    if (this != &source) deepCopy(source);
    return *this;
}
```

---
### Overloading operators and function templates

In this lesson, we’ll look at cases where our instantiated functions won’t compile because our actual class types don’t support those operators, and show how we can define those operators so that the instantiated functions will then compile.

```cpp
class Cents {
private:
    int m_cents{};
public:
    Cents(int cents) : m_cents { cents } {}
    friend std::ostream& operator<< (std::ostream& ostr, const Cents& c) {
        ostr << c.m_cents;
        return ostr;
    }
};

template <typename T>
const T& max(const T& x, const T& y) {
    return (x < y) ? y : x;
}

int main() {
    Cents nickel{ 5 };
    Cents dime{ 10 };
    Cents bigger { max(nickel, dime) };
    std::cout << bigger << " is bigger\n";
}
```

C++ will create a template instance for max() that looks like this:

```cpp
template <>
const Cents& max(const Cents& x, const Cents& y) {
    return (x < y) ? y : x;
}
```

And then it will try to compile this function. See the problem here? C++ has no idea how to evaluate `x < y` when `x` and `y` are of type `Cents`. Consequently, this will produce a compile error. To get around this problem, simply overload `operator<` for any class we wish to use `max` with

---
