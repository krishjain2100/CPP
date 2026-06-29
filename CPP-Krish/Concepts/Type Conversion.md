
What happens when we do something like this: `float f{ 3 };` ?
Without type conversion, you'd be looking at a completely different value whose fp representation matches that with int 3 representation.

**Implicit type conversion** (also called **automatic type conversion** or **coercion**) is performed automatically by the compiler when an expression of some type is supplied in a context where some other type is expected. 

As part of the core language, the C++ standard defines a collection of conversion rules called the **standard conversions**. They specify how various fundamental types (and certain compound types, including arrays, references, pointers, and enumerations) convert to other types within that same group.

As of C++23, there are 14 different standard conversions. These can be roughly grouped into 5 general categories:

| Category                  | Meaning                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| Numeric promotions        | Conversions of small integral types to `int` or `unsigned int`, and of `float` to `double`. |
| Numeric conversions       | Other integral and floating point conversions that aren’t promotions.                       |
| Qualification conversions | Conversions that add or remove `const` or `volatile`.                                       |
| Value transformations     | Conversions that change the value category of an expression                                 |
| Pointer conversions       | Conversions from `std::nullptr` to pointer types, or pointer types to other pointer types   |

Here is the full list of standard conversions:

| Category                 | Standard Conversion           | Description                                                                                                                | Also See                                                                                                                      |
| ------------------------ | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Value transformation     | Lvalue-to-rvalue              | Converts lvalue expression to rvalue expression                                                                            | [12.2 -- Value categories (lvalues and rvalues)](https://www.learncpp.com/cpp-tutorial/value-categories-lvalues-and-rvalues/) |
| Value transformation     | Array-to-pointer              | Converts C-style array to pointer to first array element (a.k.a. array decay)                                              | [17.8 -- C-style array decay](https://www.learncpp.com/cpp-tutorial/c-style-array-decay/)                                     |
| Value transformation     | Function-to-pointer           | Converts function to function pointer                                                                                      | [20.1 -- Function Pointers](https://www.learncpp.com/cpp-tutorial/function-pointers/)                                         |
| Value transformation     | Temporary materialization     | Converts value to temporary object                                                                                         |                                                                                                                               |
| Qualification conversion | Qualification conversion      | Adds or removes `const` or `volatile` from types                                                                           |                                                                                                                               |
| Numeric promotions       | Integral promotions           | Converts smaller integral types to `int` or `unsigned int`                                                                 | [10.2 -- Floating-point and integral promotion](https://www.learncpp.com/cpp-tutorial/floating-point-and-integral-promotion/) |
| Numeric promotions       | Floating point promotions     | Converts `float` to `double`                                                                                               | [10.2 -- Floating-point and integral promotion](https://www.learncpp.com/cpp-tutorial/floating-point-and-integral-promotion/) |
| Numeric conversions      | Integral conversions          | Integral conversions that aren’t integral promotions                                                                       | [10.3 -- Numeric conversions](https://www.learncpp.com/cpp-tutorial/numeric-conversions/)                                     |
| Numeric conversions      | Floating point conversions    | Floating point conversions that aren’t floating point promotions                                                           | [10.3 -- Numeric conversions](https://www.learncpp.com/cpp-tutorial/numeric-conversions/)                                     |
| Numeric conversions      | Integral-floating conversions | Converts integral and floating point types                                                                                 | [10.3 -- Numeric conversions](https://www.learncpp.com/cpp-tutorial/numeric-conversions/)                                     |
| Numeric conversions      | Boolean conversions           | Converts integral, unscoped enumeration, pointer, or pointer-to-memver to bool                                             | [4.10 -- Introduction to if statements](https://www.learncpp.com/cpp-tutorial/introduction-to-if-statements/)                 |
| Pointer conversions      | Pointer conversions           | Converts `std::nullptr` to pointer, or pointer to void pointer or base class                                               |                                                                                                                               |
| Pointer conversions      | Pointer-to-member conversions | Converts `std::nullptr` to pointer-to-member  <br>or pointer-to-member of base class to pointer-to-member of derived class |                                                                                                                               |
| Pointer conversions      | Function pointer conversions  | Converts pointer-to-noexcept-function to pointer-to-function                                                               |                                                                                                                               |

---
#### Type conversion can fail

When a type conversion is invoked (whether implicitly or explicitly), if a valid conversion can be found by the compiler, then the compiler will produce a new value of the desired type. If the compiler can’t find an acceptable conversion, then the compilation will fail with a compile error.
In other cases, specific features may disallow some categories of conversions. For example: Brace-initialization disallows conversions that result in data loss

There are also cases where the compiler may not be able to figure out which of several possible type conversions is the best one to use. We will see examples of this in Overload Resolution

The full set of rules describing how type conversions work is both lengthy and complicated, and for the most part, type conversion “just works”.

---
### Numeric promotion

Because C++ is designed to be portable and performant across a wide range of architectures, the language designers did not want to assume a given CPU would be able to efficiently manipulate values that were narrower than the natural data size for that CPU. So C++ defines a category of type conversions informally called the `numeric promotions`.

A **numeric promotion** is the type conversion of certain narrower numeric types (such as a `char`) to certain wider numeric types (typically `int` or `double`) that can be processed efficiently.
All numeric promotions are value-preserving. Because promotions are safe, the compiler will freely use numeric promotion as needed, and will not issue a warning when doing so.

A **value-preserving conversion** (also called a **safe conversion**) is one where every possible source value can be converted into an equal value of the destination type. 

Numeric promotion solves redudancy. Consider the case where you wanted to write a function to print a value of type `int`. If type conversions did not exist, we’d have to write a different print function for `short` and another one for `char`, another version for `unsigned char`, `signed char`, `unsigned short`, `wchar_t`, `char8_t`, `char16_t`, and `char32_t`.

Numeric promotion comes to the rescue here. We can write functions that have `int` and/or `double` parameters. That same code can then be called with arguments of types that can be numerically promoted to match the types of the function parameters.

---
#### Numeric promotion categories

The numeric promotion rules are divided into two subcategories: `integral promotions` and `floating point promotions`. Only the conversions listed in these categories are considered to be numeric promotions.

1. Floating point promotions: Using the floating point promotion rules, a value of type `float` can be converted to a value of type `double`.
2. Integral promotions: Using the integral promotion rules, the following conversions can be made:
	- `signed char` or `signed short` can be converted to` int`.
	- `unsigned char`, `char8_t`, and `unsigned short` can be converted to `int` if `int` can hold the entire range of the type, or `unsigned int` otherwise.
	- If `char` is signed by default, it follows the `signed char` conversion rules above. If it is unsigned by default, it follows the `unsigned char` conversion rules above.
	- `bool` can be converted to `int`, with false becoming 0 and true becoming 1.

Note:
- Assuming an 8 bit byte and an `int` size of 4 bytes or larger (which is typical these days), the above basically means that `bool`, `char`, `signed char`, `unsigned char`, `signed short`, and `unsigned short` all get promoted to `int`. 
- On some architectures (e.g. with 2 byte `int`) it is possible for some of the unsigned integral types to be promoted to `unsigned int` rather than `int`.
- Some narrower unsigned types (such as `unsigned char`) may be promoted to larger signed types (such as `int`). So while integral promotion is value-preserving, it does not necessarily preserve the signedness (signed/unsigned) of the type.

---
#### Not all widening conversions are numeric promotions

Some widening type conversions (such as `char` to `short`, or `int` to `long`) are not considered to be numeric promotions in C++ (they are `numeric conversions`, which we’ll cover shortly). This is because such conversions do not assist in the goal of converting smaller types to larger types that can be processed more efficiently.

The distinction is mostly academic. However, in certain cases, the compiler will favour numeric promotions over numeric conversions. We’ll see examples where this makes a difference when we cover function overload resolution.

---
#### The Definition of an `int` (Gemini)

In C and C++, `int` is not just a number. It is specifically intended to represent the **natural word size** of the target computer's CPU.
- It is supposed to be the exact size of the hardware registers inside the processor.
- It is the size that the CPU can process the fastest and most efficiently without having to do extra math to chop up memory.
- Similarly double is also the natural processing type for the cpu
- The first point in the next lesson does support the above thing

You might wonder `double` is mostly 64 bits and `int` 32 bits, shouldn't they be same?
Inside your CPU, integer math and decimal math are handled by two completely different physical processors:
- **The ALU (Arithmetic Logic Unit):** Handles integers (`int`).
- **The FPU (Floating Point Unit):** Handles decimals (`float`, `double`).
Because scientific and 3D graphics calculations require massive precision, FPUs were built to handle 64-bit (and even 80-bit) math way back in the 1980s and 1990s, long before the rest of the computer caught up. So for the FPU, 64-bit `double`s have been the "natural" processing size for decades.

But now systems have become 64 bits then why is `int` still 32 bits: 
**Backward compatibility**, even though your CPU _can_ process 64-bit integers natively, keeping `int` at 32 bits turned out to be a massive performance advantage for modern computing.
Today, the bottleneck in computers is not how fast the CPU can do math; it is how fast the CPU can pull data from RAM into its ultra-fast internal memory (the Cache).
- A 32-bit `int` takes up half the physical space of a 64-bit integer.
- This means your CPU can pack **twice as many** `int` variables into its L1 Cache at the exact same time. So it is still faster to keep it 32 bits.

---
### Numeric conversions

These numeric conversions cover additional type conversions between fundamental types.

There are five basic types of numeric conversions.
1. Converting an integral type to any other integral type (excluding integral promotions):
2. Converting a floating point type to any other floating point type (excluding floating point promotions)
3. Converting a floating point type to any integral type:
4. Converting an integral type to any floating point type
5. Converting an integral type or a floating point type to a bool

Because brace initialization strictly disallows some types of numeric conversions (more on this next lesson), we use copy initialization in this lesson (which does not have any such limitations) in order to keep the examples simple.

Unlike numeric promotions (which are safe), many numeric conversions are unsafe. An **unsafe conversion** is one where at least one value of the source type cannot be converted into an equal value of the destination type.

Numeric conversions fall into three general safety categories:

1. _Value-preserving conversions_ (safe): 
	For example, `int` to `long` and `short` to `double`.
	Compilers will typically not issue warnings for implicit value-preserving conversions.
	A value converted using a value-preserving conversion can always be converted back to the source type.

2. Reinterpretive conversions are unsafe numeric conversions where the converted value may be different than the source value, but no data is lost. 
	For example: Signed/unsigned conversions fall into this category (`signed int` to an `unsigned int`)

```cpp
int n1 { 5 };
unsigned int u1 { n1 }; 
// okay: will be converted to unsigned int 5 (value preserved)
int n2 { -5 };
unsigned int u2 { n2 }; 
// bad: will result in large integer outside range of signed int
```

Even though reinterpretive conversions are unsafe, most compilers leave implicit signed/unsigned conversion warnings disabled by default  because in some areas of modern C++ (such as when working with standard library arrays), signed/unsigned conversions can be hard to avoid. Also, the majority of such conversions do not actually result in a value change. Therefore, enabling such warnings can lead to many unnecessary for signed/unsigned conversions that are actually okay.

Values converted using a reinterpretive conversion can be converted back to the source type, resulting in a value equivalent to the original value as no data was lost.

3. _Lossy conversions_ (unsafe)  data may be lost during the conversion.
	For example, `double` to `int`, `double` to `float`
	Converting a value that has lost data back to the source type will result in a value that is different than the original value
	Compilers will generally issue a warning (or in some cases, an error) when an implicit lossy conversion would be performed at runtime.

Unsafe conversions should be avoided as much as possible. However, this is not always possible and sometimes we want it delibrately. Example:`int` to `bool` to check zeroness.

Note: Some conversions may fall into different categories depending on the platform.
For example, `int` to `double` is typically a safe conversion, because `int` is usually 4 bytes and `double` is usually 8 bytes, and on such systems, all possible `int` values can be represented as a `double`. However, there are architectures where both `int` and `double` are 8 bytes. On such architectures, `int` to `double` is a lossy conversion.

---
### More on numeric conversions

The specific rules for numeric conversions are complicated and numerous, so here are the most important things to remember.

- In _all_ cases, converting a value into a type whose range doesn’t support that value will lead to results that are probably unexpected.
- Remember that overflow is well-defined for unsigned values and produces undefined behavior for signed values.
- Converting from a larger integral or floating point type to a smaller type from the same family will generally work so long as the value fits in the range of the smaller type.
- In the case of floating point values, some rounding may occur due to a loss of precision in the smaller type.
- Converting from an integer to a floating point number generally works as long as the value fits within the range of the floating point type.
- Converting from a floating point to an integer works as long as the value fits within the range of the integer, but any fractional values are lost. 
- While the numeric conversion rules might seem scary, in reality the compiler will generally warn you if you try to do something dangerous (excluding some signed/unsigned conversions).

---
### Narrowing Conversions

A **narrowing conversion** is a potentially unsafe numeric conversion where the destination type may not be able to hold all the values of the source type. 

The following conversions are defined to be narrowing (constexpr clauses are discussed below):
- From a floating point type to an integral type.
- From a floating point type to a narrower or lesser ranked floating point type, unless the value being converted is constexpr and is in range of the destination type (even if the destination type doesn’t have the precision to store all the significant digits of the number).
- From an integral to a floating point type, unless the value being converted is constexpr and whose value can be stored exactly in the destination type.
- From an integral type to another integral type that cannot represent all values of the original type, unless the value being converted is constexpr and whose value can be stored exactly in the destination type. This covers both wider to narrower integral conversions, as well as integral sign conversions (signed to unsigned, or vice-versa).

In most cases, implicit narrowing conversions will result in compiler warnings, with the exception of signed/unsigned conversions (which may or may not produce warnings, depending on how your compiler is configured).

Making narrowing conversions explicit by using `static_cast` helps document that the narrowing conversion is intentional, and will suppress any compiler warnings or errors that would otherwise result.

Narrowing conversions are disallowed when using brace initialization (which is one of the primary reasons this initialization form is preferred), and attempting to do so will produce a compile error. If you actually want to do a narrowing conversion inside a brace initialization, use `static_cast` to convert the narrowing conversion into an explicit conversion

---
#### Constexpr clauses

When the source value of a narrowing conversion isn’t known until runtime, the result of the conversion also can’t be determined until runtime. In such cases, whether the narrowing conversion preserves the value or not also can’t be determined until runtime.

When the source value of a narrowing conversion is constexpr, the specific value to be converted must be known to the compiler. In such cases, the compiler can perform the conversion itself, and then check whether the value was preserved. If the value was not preserved, the compiler can halt compilation with an error. If the value is preserved, the conversion is not considered to be narrowing (and the compiler can replace the entire conversion with the converted result, knowing that doing so is safe).

```cpp
constexpr int n1{ 5 };   // note: constexpr
unsigned int u1 { n1 };  // okay: conversion is not narrowing

constexpr int n2 { -5 }; // note: constexpr
unsigned int u2 { n2 };   // compile error: conversion is narrowing(value change)
```

**Strangely, conversions from a floating point type to an integral type do not have a constexpr exclusion clause, so these are always considered narrowing conversions even when the value to be converted is constexpr and fits in the range of the destination type.**

**Even more strangely, conversions from a constexpr floating point type to a narrower floating point type are not considered narrowing even when there is a loss of precision!**

These constexpr exception clauses are incredibly useful when list initializing non-int/non-double objects, as we can use an int or double literal (or a constexpr object) initialization value.

This allows us to avoid:
- Having to use literal suffixes in most cases
- Having to clutter our initializations with a static_cast

```cpp
// We can avoid literals with suffixes 
unsigned int u { 5 }; // okay (we don't need to use `5u`)
float f { 1.5 };      // okay (we don't need to use `1.5f`)

// We can avoid static_casts
constexpr int n{ 5 };
double d { n }; // okay (we don't need a static_cast here)
short s { 5 }; // okay (there is no suffix for short, we don't need a static_cast here)

```

This also works with copy and direct initialization.

One thing to take away: initializing a narrower or lesser ranked floating point type with a constexpr value is allowed as long as the value is in range of the destination type, even if the destination type doesn’t have enough precision to precisely store the value.  However, your compiler may still issue a warning in this case (GCC and Clang do if you use the -Wconversion compile flag).

Floating point types are ranked in this order (greater to lesser):
- Long double
- Double
- Float

---
### Arithmetic Conversions

In C++, certain operators require that their operands be of the same type. If one of these operators is invoked with operands of different types, one or both of the operands will be implicitly converted to matching types using a set of rules called the **usual arithmetic conversions**. The matching type produced as a result of the usual arithmetic conversion rules is called the **common type** of the operands.

The following operators require their operands to be of the same type:
- The binary arithmetic operators: +, -, *, /, %
- The binary relational operators: <, >, <=, >=, `==`, `!=`
- The binary bitwise arithmetic operators: &, ^, |
- The conditional operator ?: (excluding the condition, which is expected to be of type `bool`)

Overloaded operators are not subject to the usual arithmetic conversion rules.


The usual arithmetic conversion rules are somewhat complex, so we’ll simplify a bit. The compiler has a ranked list of types that looks something like this:
- long double (highest rank)
- double
- float
- long long
- long
- int (lowest rank)

Step 1:
- If one operand is an integral type and the other a floating point type, the integral operand is converted to the type of the floating point operand (no integral promotion takes place).
- Otherwise, any integral operands are numerically promoted
Step 2:
- After promotion, if one operand is signed and the other unsigned, special rules apply (see below)
- Otherwise, the operand with lower rank is converted to the type of the operand with higher rank.

The special matching rules for integral operands with different signs:
- If the rank of the unsigned operand is greater than or equal to the rank of the signed operand, the signed operand is converted to the type of the unsigned operand.
- If the type of the signed operand can represent all the values of the type of the unsigned operand, the type of the unsigned operand is converted to the type of the signed operand.
- Otherwise both operands are converted to the corresponding unsigned type of the signed operand.

You can find the full rules for the usual arithmetic conversions [here](https://en.cppreference.com/w/cpp/language/usual_arithmetic_conversions).

---
#### Some examples

In the following examples, we’ll use the `typeid` operator (included in the `<typeinfo>` header), to show the resulting type of an expression.

```cpp
int i{ 2 };
std::cout << typeid(i).name() << '\n'; // int
double d{ 3.5 };
std::cout << typeid(d).name() << '\n'; // double
std::cout << typeid(i + d).name() << ' ' << i + d << '\n'; // double
```

```cpp
short a{ 4 };
short b{ 5 };
std::cout << typeid(a + b).name() << ' ' << a + b << '\n'; // int
```

---
#### Signed and unsigned issues

```cpp
std::cout << typeid(5u-10).name() << ' ' << 5u - 10 << '\n'; 
```

You might expect the expression `5u - 10` to evaluate to `-5` since `5 - 10` = `-5`. But here’s what actually results:

unsigned int 4294967291

Due to the conversion rules, the `int` operand is converted to an `unsigned int`. And since the value `-5` is out of range of an `unsigned int`, we get a result we don’t expect.

```cpp
std::cout << std::boolalpha << (-3 < 5u) << '\n';
```

While it’s clear to us that `5` is greater than `-3`, when this expression evaluates, `-3` is converted to a large `unsigned int` that is larger than `5`. Thus, the above prints `false` rather than the expected result of `true`.
This is one of the primary reasons to avoid unsigned integers,  when you mix them with signed integers in arithmetic expressions, you’re at risk for unexpected results. And the compiler probably won’t even issue a warning.

---
#### `std::common_type` and `std::common_type_t`

In future lessons (Function templates with multiple template arguments), we’ll encounter cases where it is useful to know what the common type (after arithmetic conversion) of two type is. `std::common_type` and the useful type alias `std::common_type_t` (both defined in the `<type_traits>` header) can be used for just this purpose.

For example, `std::common_type_t<int, double>` returns the common type of `int` and `double`, and `std::common_type_t<unsigned int, long>` returns the common type of `unsigned int` and `long`.

When C++11 introduced type traits, they were implemented as `struct` templates. `std::common_type` is actually a struct that calculates the common type and stores the result inside a nested member called `type`. `std::common_type_t` extracts that `type`.

```cpp
template <class... T> 
using common_type_t = typename common_type<T...>::type;
```

---