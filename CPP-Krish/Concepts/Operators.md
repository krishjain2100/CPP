An **operation** is a mathematical process involving zero or more input values (called **operands**) that produces a new value. The specific operation to be performed is denoted by a construct (typically a symbol or pair of symbols) called an **operator**.

### Table of operator precedence and associativity

Notes:
- Precedence level 1 is the highest precedence level, and level 17 is the lowest. Operators with a higher precedence level have their operands grouped first.
- L->R means left to right associativity.
- R->L means right to left associativity.

|Prec/Ass|Operator|Description|Pattern|
|---|---|---|---|
|1 L->R|::  <br>::|Global scope (unary)  <br>Namespace scope (binary)|::name  <br>class_name::member_name|
|2 L->R|()  <br>()  <br>type()  <br>type{}  <br>[]  <br>.  <br>->  <br>++  <br>––  <br>typeid  <br>const_cast  <br>dynamic_cast  <br>reinterpret_cast  <br>static_cast  <br>sizeof…  <br>noexcept  <br>alignof|Parentheses  <br>Function call  <br>Functional cast  <br>List init temporary object (C++11)  <br>Array subscript  <br>Member access from object  <br>Member access from object ptr  <br>Post-increment  <br>Post-decrement  <br>Run-time type information  <br>Cast away const  <br>Run-time type-checked cast  <br>Cast one type to another  <br>Compile-time type-checked cast  <br>Get parameter pack size  <br>Compile-time exception check  <br>Get type alignment|(expression)  <br>function_name(arguments)  <br>type(expression)  <br>type{expression}  <br>pointer[expression]  <br>object.member_name  <br>object_pointer->member_name  <br>lvalue++  <br>lvalue––  <br>typeid(type) or typeid(expression)  <br>const_cast<type>(expression)  <br>dynamic_cast<type>(expression)  <br>reinterpret_cast<type>(expression)  <br>static_cast<type>(expression)  <br>sizeof…(expression)  <br>noexcept(expression)  <br>alignof(type)|
|3 R->L|+  <br>-  <br>++  <br>––  <br>!  <br>not  <br>~  <br>(type)  <br>sizeof  <br>co_await  <br>&  <br>*  <br>new  <br>new[]  <br>delete  <br>delete[]|Unary plus  <br>Unary minus  <br>Pre-increment  <br>Pre-decrement  <br>Logical NOT  <br>Logical NOT  <br>Bitwise NOT  <br>C-style cast  <br>Size in bytes  <br>Await asynchronous call  <br>Address of  <br>Dereference  <br>Dynamic memory allocation  <br>Dynamic array allocation  <br>Dynamic memory deletion  <br>Dynamic array deletion|+expression  <br>-expression  <br>++lvalue  <br>––lvalue  <br>!expression  <br>not expression  <br>~expression  <br>(new_type)expression  <br>sizeof(type) or sizeof(expression)  <br>co_await expression (C++20)  <br>&lvalue  <br>*expression  <br>new type  <br>new type[expression]  <br>delete pointer  <br>delete[] pointer|
|4 L->R|->*  <br>.*|Member pointer selector  <br>Member object selector|object_pointer->*pointer_to_member  <br>object.*pointer_to_member|
|5 L->R|*  <br>/  <br>%|Multiplication  <br>Division  <br>Remainder|expression * expression  <br>expression / expression  <br>expression % expression|
|6 L->R|+  <br>-|Addition  <br>Subtraction|expression + expression  <br>expression - expression|
|7 L->R|<<  <br>>>|Bitwise shift left / Insertion  <br>Bitwise shift right / Extraction|expression << expression  <br>expression >> expression|
|8 L->R|<=>|Three-way comparison (C++20)|expression <=> expression|
|9 L->R|<  <br><=  <br>>  <br>>=|Comparison less than  <br>Comparison less than or equals  <br>Comparison greater than  <br>Comparison greater than or equals|expression < expression  <br>expression <= expression  <br>expression > expression  <br>expression >= expression|
|10 L->R|==  <br>!=|Equality  <br>Inequality|expression == expression  <br>expression != expression|
|11 L->R|&|Bitwise AND|expression & expression|
|12 L->R|^|Bitwise XOR|expression ^ expression|
|13 L->R|\||Bitwise OR|expression \| expression|
|14 L->R|&&  <br>and|Logical AND  <br>Logical AND|expression && expression  <br>expression and expression|
|15 L->R|\|  <br>or|Logical OR  <br>Logical OR|expression \| expression  <br>expression or expression|
|16 R->L|throw  <br>co_yield  <br>?:  <br>=  <br>*=  <br>/=  <br>%=  <br>+=  <br>-=  <br><<=  <br>>>=  <br>&=  <br>\|=  <br>^=|Throw expression  <br>Yield expression (C++20)  <br>Conditional  <br>Assignment  <br>Multiplication assignment  <br>Division assignment  <br>Remainder assignment  <br>Addition assignment  <br>Subtraction assignment  <br>Bitwise shift left assignment  <br>Bitwise shift right assignment  <br>Bitwise AND assignment  <br>Bitwise OR assignment  <br>Bitwise XOR assignment|throw expression  <br>co_yield expression  <br>expression ? expression : expression  <br>lvalue = expression  <br>lvalue *= expression  <br>lvalue /= expression  <br>lvalue %= expression  <br>lvalue += expression  <br>lvalue -= expression  <br>lvalue <<= expression  <br>lvalue >>= expression  <br>lvalue &= expression  <br>lvalue \|= expression  <br>lvalue ^= expression|
|17 L->R|,|Comma operator|expression, expression|

Expressions with a single assignment operator do not need to have the right operand of the assignment wrapped in parenthesis.

The C++ standard (mostly) uses the term **evaluation** to refer to the evaluation of operands (not the evaluation of operators or expressions). For example, given expression `a + b`, `a` will be evaluated to produce some value, and `b` will be evaluated to produce some value. These values can be then used as operands to `operator+` for value computation. Informally,  we typically use the term _evaluates_ to mean the evaluation of an entire expression (value computation), not just the operands of an expression.

---
### The order of evaluation of operands  is mostly unspecified

In most cases, the order of evaluation for operands and function arguments is unspecified, meaning they may be evaluated in any order.

```cpp
a * b + c * d // converts to the expression below first
(a * b) + (c * d)
```

 If `a` is `1`, `b` is `2`, `c` is `3`, and `d` is `4`, this expression will always compute the value `14`.
However, the precedence and associativity rules only tell us how operators and operands are grouped and the order in which value computation will occur. They do not tell us the order in which the operands or subexpressions are evaluated. The compiler is free to evaluate operands `a`, `b`, `c`, or `d` in any order. The compiler is also free to calculate `a * b` or `c * d` first.

For most expressions, this is irrelevant. But it is possible to write expressions where the order of evaluation does matter. Consider the following program"

```cpp
int getValue() {
    std::cout << "Enter an integer: ";
    int x{};
    std::cin >> x;
    return x;
}

void printCalculation(int x, int y, int z) {
    std::cout << x + (y * z);
}

int main() {
    printCalculation(getValue(), getValue(), getValue()); 
    // this line is ambiguous
}
```

The Clang compiler evaluates arguments in left-to-right order. The GCC compiler evaluates arguments in right-to-left order. So x, y, z, could get the values in different orders. Therefore avoid things like that and precalculate the functions value in a variable before putting them in the arguments.

---
### Operators

#### Arithmetic operators

- If either (or both) of the operands are floating point values, the _division operator_ performs floating point division.
- If both of the operands are integers, the _division operator_ performs integer division .
- Dividing by 0 and 0.0 discussed in [[Data Types]]
- The remainder operator with negative operands always returns results with the sign of first operand.
- C++ does not include an exponent operator.
- To do exponents in C++, `#include` the `<cmath>` header, and use the `pow()` function
- Note that the parameters (and return value) of function pow() are of type double. Due to rounding errors in floating point numbers, the results of pow() may not be precise (even if you pass it integers or whole numbers).
- If you want to do integer exponentiation, you’re best off using your own function to do so.
- In cases where code can be written to use either prefix or postfix increment/decrement, prefer the prefix versions, as they are generally more performant.

---
#### Side Effects

- A function or expression is said to have a **side effect** if it has some observable effect beyond producing a return value.
- For example, the expression `x + ++x` is unspecified behavior. When `x` is initialized to `1`, Visual Studio and GCC evaluate this as `2 + 2`, and Clang evaluates it as `1 + 2`. This is due to differences in when the compilers apply the side effect of incrementing `x`.
- These problems can generally _all_ be avoided by ensuring that any variable that has a side-effect applied is used no more than once in a given statement.

---
#### The comma operator

- The **comma operator (,)** allows you to evaluate multiple expressions wherever a single expression is allowed. The comma operator evaluates the left operand, then the right operand, and then returns the result of the right operand.
- Note that comma has the lowest precedence of all the operators, even lower than assignment. Because of this, the following two lines of code do different things:

```cpp
z = (a, b); // evaluate (a, b) first to get result of b, then assign that value to variable z.
z = a, b; // evaluates as "(z = a), b", so z gets assigned the value of a, and b is evaluated and discarded.
```

- In almost every case, a statement written using the comma operator would be better written as separate statements.
- Most programmers do not use the comma operator at all, with the single exception of inside _for loops_, where its use is fairly common. 

---
#### The conditional operator

- The **conditional operator** (`?:`)  is a ternary operator (an operator that takes 3 operands). Because it has historically been C++’s only ternary operator, it’s also sometimes referred to as _the ternary operator_.

- Since the conditional operator is evaluated as part of an expression, it can be used anywhere an expression is accepted. In cases where the operands of the conditional operator are constant expressions, the conditional operator can be used in a constant expression. This allows the conditional operator to be used in places where statements cannot be used. eg:

```cpp
constexpr bool inBigClassroom { false };
constexpr int classSize { inBigClassroom ? 30 : 20 };
std::cout << "The class size is: " << classSize << '\n';
```

There’s no direct if-else replacement for this.

- Because C++ prioritizes the evaluation of most operators above the evaluation of the conditional operator, it’s quite easy to write expressions using the conditional operator that don’t evaluate as expected.

```cpp
int x { 2 };
std::cout << (x < 0) ? "negative" : "non-negative";
```

You might expect this to print `non-negative`, but it actually prints `0`.

Here’s what’s happening in the above example. First, `x < 0` evaluates to `false`. The partially evaluated expression is now `std::cout << false ? "negative" : "non-negative"`. Because `operator<<` has higher precedence than `operator?:`, this expression evaluates as if it were written as `(std::cout << false) ? "negative" : "non-negative"`. Thus `std::cout << false` is evaluated, which prints `0` (and returns `std::cout`).

The partially evaluated expression is now `std::cout ? "negative" : "non-negative"`. Since `std::cout` is all that is remaining in the condition, the compiler will try to convert it to a `bool` so the condition can be resolved. Perhaps surprisingly, `std::cout` has a defined conversion to `bool`, which will most likely return `false`. Assuming it returns `false`, we now have `false ? "negative" : "non-negative"`, which evaluates to `"non-negative"`. So our fully evaluated statement is `"non-negative";`. An expression statement that is just a literal (in this case, a string literal) has no effect, so we’re done

To comply with C++’s type checking rules, one of the following must be true:
- The type of the second and third operand must match.
- The compiler must be able to find a way to convert one or both of the second and third operands to matching types. The conversion rules the compiler uses are fairly complex and may yield surprising results in some cases.

Alternatively, one or both of the second and third operands is allowed to be a throw expression.

```cpp
cout << (true ? 1 : 2) << '\n'; // okay: both operands have matching type int
cout << (false ? 1 : 2.2) << '\n'; // okay: int value 1 converted to double
cout << (true ? -1 : 2u) << '\n';  // surprising result: -1 converted to unsigned int, result out of range
```

In general, it’s okay to mix operands with fundamental types (excluding mixing signed and unsigned values). If either operand is not a fundamental type, it’s generally best to explicitly convert one or both operands to a matching type yourself so you know exactly what you’ll get.

----
#### Relational Operators

**Relational operators** are operators that let you compare two values. There are 6 relational operators:

| Operator               | Symbol | Form   | Operation                                                |
| ---------------------- | ------ | ------ | -------------------------------------------------------- |
| Greater than           | >      | x > y  | true if x is greater than y, false otherwise             |
| Less than              | <      | x < y  | true if x is less than y, false otherwise                |
| Greater than or equals | >=     | x >= y | true if x is greater than or equal to y, false otherwise |
| Less than or equals    | <=     | x <= y | true if x is less than or equal to y, false otherwise    |
| Equality               | ==     | x == y | true if x equals y, false otherwise                      |
| Inequality             | !=     | x != y | true if x does not equal y, false otherwise              |
#### Comparison of calculated floating point values can be problematic

- Comparing floating point values using any of the relational operators can be dangerous. This is because floating point values are not precise, and small rounding errors in the floating point operands may cause them to be slightly smaller or slightly larger than expected. And this can throw off the relational operators.
- If the consequence of getting a wrong answer when the operands are similar is acceptable, then using these operators can be acceptable. This is an application-specific decision.
- For example, consider a game (such as Space Invaders) where you want to determine whether two moving objects (such as a missile and an alien) intersect. If the objects are still far apart, these operators will return the correct answer. If the two objects are extremely close together, you might get an answer either way. In such cases, the wrong answer probably wouldn’t even be noticed (it would just look like a near miss, or near hit) and the game would continue.
- Avoid using operator== and operator!= to compare floating point values if there is any chance those values have been calculated.
- There is one notable exception case to the above: It is safe to compare a floating point literal with a variable of the same type that has been initialized with a literal of the same type, so long as the number of significant digits in each literal does not exceed the minimum precision for that type. Float has a minimum precision of 6 significant digits, and double has a minimum precision of 15 significant digits.

Example:

```cpp
if (someFcn() == 0.0) // okay if someFcn() returns 0.0 as a literal only
    // do something
constexpr double gravity { 9.8 };
if (gravity == 9.8) // okay if gravity was initialized with a literal
    // we're on earth
```

It is mostly not safe to compare floating point literals of different types. For example, comparing `9.8f` to `9.8` will return false.

##### Comparing floating point numbers

- The most common method of doing floating point equality involves using a function that looks to see if two numbers are _almost_ the same. If they are “close enough”, then we call them equal. The value used to represent “close enough” is traditionally called **epsilon**.
- While this function can work, it’s not great. An epsilon of _0.00001_ is good for inputs around _1.0_, too big for inputs around _0.0000001_, and too small for inputs like _10,000_.
- Donald Knuth, a famous computer scientist, suggested the following method in his book

```cpp
#include <algorithm> // for std::max
#include <cmath>     // for std::abs

// Return true if the difference between a and b is within epsilon percent of the larger of a and b
bool approximatelyEqualRel(double a, double b, double relEpsilon) {
	return (std::abs(a - b) <= (std::max(std::abs(a), std::abs(b)) * relEpsilon));
}
```

Note that while the `approximatelyEqualRel()` function will work for most cases, it is not perfect, especially as the numbers approach zero:

```cpp
// a is really close to 1.0, but has rounding errors
constexpr double a{ 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 + 0.1 };
constexpr double relEps { 1e-8 };
constexpr double absEps { 1e-12 };
std::cout << std::boolalpha; // print true or false instead of 1 or 0
// Compare a (almost 1.0) to 1.0.
std::cout << approximatelyEqualRel(a, 1.0, relEps) << '\n'; // true
// Compare a-1.0 (almost 0.0) to 0.0
std::cout << approximatelyEqualRel(a-1.0, 0.0, relEps) << '\n'; // false
```

One way to avoid this is to use both an absolute epsilon (as we did in the first approach) and a relative epsilon (as we did in Knuth’s approach):

Comparison of floating point numbers is a difficult topic, and there’s no “one size fits all” algorithm that works for every case. However, the `approximatelyEqualAbsRel()` function with an `absEpsilon` of `1e-12` and a `relEpsilon` of `1e-8` should be good enough to handle most cases you’ll encounter.

In C++23, the two `approximatelyEqual` functions can be made constexpr by adding the `constexpr` keyword. However, prior to C++23, we run into an issue. If these constexpr function are called in a constant expression. This is because a constexpr function that is used in a constant expression can’t call a non-constexpr function, and `std::abs` wasn’t made constexpr until C++23.

----
### Logical operators

| Operator    | Symbol | Example Usage | Operation                                                 |
| ----------- | ------ | ------------- | --------------------------------------------------------- |
| Logical NOT | !      | !x            | true if x is false, or false if x is true                 |
| Logical AND | &&     | x && y        | true if x and y are both true, false otherwise            |
| Logical OR  | \|     | x \| y        | true if either (or both) x or y are true, false otherwise |
#### Short circuit evaluation

- In order for _logical AND_ to return true, both operands must evaluate to true. If the left operand evaluates to false the _logical AND_ operator will return false immediately without evaluating the right operand.
- The Logical OR and logical AND operators are an exception to the rule that the operands may evaluate in any order, as the standard explicitly states that the left operand must evaluate first.
- Only the built-in versions of these operators perform short-circuit evaluation. If you overload these operators to make them work with your own types, those overloaded operators will not perform short-circuit evaluation.

#### Logical XOR operator

- C++ doesn’t provide an explicit _logical XOR_ operator, `^` is bitwise XOR.
- However, `operator!=` produces the same result as a logical XOR when given `bool` operands
- Therefore, a logical XOR can be implemented as follows:

```cpp
if (a != b != c....) ... // a XOR b XOR c ..., assuming a and b are bool
```

If you need a form of _logical XOR_ that works with non-Boolean operands, you can `static_cast` your operands to bool:

```cpp
if (static_cast<bool>(a) != static_cast<bool>(b) != static_cast<bool>(c)) ... // a XOR b XOR c, for any type that can be converted to bool
```

However, this is a bit verbose. The following trick also works and is a bit more concise:

```cpp
if (!!a != !!b != !!c) // a XOR b XOR c, for any type that can be converted to bool
```

---
