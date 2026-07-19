
An **enumeration** (also called an **enumerated type** or an **enum**) is a compound data type whose values are restricted to a set of named symbolic constants (called **enumerators**).

There are two types:
1. Un-scoped enumerations
2. Scoped enumerations

Because enumerations are program-defined types, each enumeration needs to be fully defined before we can use it (a forward declaration is not sufficient).

---
### Un-scoped enumerations

Unscoped enumerations are defined via the `enum` keyword.

```cpp
enum Color {
    red,
    green,
    blue, 
};

int main() {
    Color apple { red };   // my apple is red
    Color shirt { green }; // my shirt is green
    Color cup { blue };    // my cup is blue

    Color socks { white }; // error: white is not an enumerator of Color
    Color hat { 2 };       // error: 2 is not an enumerator of Color
}
```

Inside a pair of curly braces, we define the enumerators for the `Color` type: `red`, `green`, and `blue`. These enumerators define the specific values that type `Color` is restricted to. Each enumerator must be separated by a comma (not a semicolon), a trailing comma after the last enumerator is optional but recommended for consistency. The type definition for `Color` ends with a semicolon.  The variables `socks` and `hat` cause compile errors because the initializers `white` and `2` are not enumerators of `Color`.

To quickly recap on nomenclature:
- An _enumeration_ or _enumerated type_ is the program-defined type itself (e.g. `Color`).
- An _enumerator_ is a specific named value belonging to the enumeration (e.g. `red`).

Enumerators are implicitly constexpr.

Each enumerated type you create is considered to be a **distinct type**, meaning the compiler can distinguish it from other types (unlike typedefs or type aliases, which are considered non-distinct from the types they are aliasing). Because enumerated types are distinct, enumerators defined as part of one enumerated type can’t be used with objects of another enumerated type:

---
#### The scope of unscoped enumerations

Unscoped enumerations are named such because they put their enumerator names into the same scope as the enumeration definition itself.

```cpp
// this enum is defined in the global namespace
enum Color { red,  green, blue, }; // so red is put into the global namespace
int main() {
    Color apple { red }; // my apple is red
}
```

The `Color` enumeration is defined in the global scope. Therefore, all the enumeration names (`red`, `green`, and `blue`) also go into the global scope. This pollutes the global scope and significantly raises the chance of naming collisions.

One consequence of this is that an enumerator name can’t be used in multiple enumerations within the same scope:

```cpp
enum Color { red,  green, blue, };
enum Feeling { happy, tired, blue, };
int main() {
    Color apple { red }; 
    Feeling me { happy };
}
```

In the above example, both un-scoped enumerations (`Color` and `Feeling`) put enumerators with the same name `blue` into the global scope. This leads to a naming collision and subsequent compile error.

Un-scoped enumerations also provide a named scope region for their enumerators (much like a namespace acts as a named scope region for the names declared within). This means we can access the enumerators of an un-scoped enumeration as follows:

```cpp
enum Color { red,  green, blue, };

int main() {
    Color apple { red }; // okay, accessing from global namespace
    Color berry { Color::red }; // okay, accessing from scope of Color
}
```

To prevent un-scoped enumerator naming collisions one option is to prefix each enumerator with the name of the enumeration itself. A better option is to put the enumerated type inside something that provides a separate scope region, such as a namespace:

```cpp
namespace Color {
    enum Color { red,  green, blue, };
}
namespace Feeling {
    enum Feeling { happy, tired, blue, };
}

int main() {
    Color::Color paint{ Color::blue };
    Feeling::Feeling me{ Feeling::blue };
}

// This means we now have to prefix our enumeration and enumerator names with the name of the scoped region.
```

Classes also provide a scope region, and it’s common to put enumerated types related to a class inside the scope region of the class.
Alternatively, if an enumeration is only used within the body of a single function, the enumeration should be defined inside the function. This limits the scope of the enumeration and its enumerators to just that function. The enumerators of such an enumeration will shadow identically named enumerators defined in the global scope.

---
We can use the equality operators (`operator==` and `operator!=`) to test whether an enumeration has the value of a particular enumerator or not.

```cpp
Color shirt{ blue };
if (shirt == blue) std::cout << "Your shirt is blue!";
else std::cout << "Your shirt is not blue!";
```
---
 
 Enums can also be used to define a collection of related bit flag positions for use with `std::bitset`:

```cpp
namespace Flags {
    enum State {
        isHungry,
        isSad,
        isMad,
        isHappy,
        isLaughing,
        isAsleep,
        isDead,
        isCrying,
    };
}

int main() {
    std::bitset<8> me{};
    me.set(Flags::isHappy);
    me.set(Flags::isLaughing);
    std::cout << std::boolalpha; // print bool as true/false
    std::cout << "I am happy? " << me.test(Flags::isHappy) << '\n';
    std::cout << "I am laughing? " << me.test(Flags::isLaughing) << '\n';
}
```

Un-scoped enumerators will implicitly convert to integral values. We will explore this further:
When we define an enumeration, each enumerator is automatically associated with an integer value based on its position in the enumerator list. By default, the first enumerator is given the integral value `0`, and so on.

```cpp
enum Color {
    black,   // 0
    red,     // 1
    blue,    // 2
};

int main() {
    Color shirt{ blue }; // shirt actually stores integral value 2
}
```

It is possible to explicitly define the value of enumerators. These integral values can be positive or negative, and can share the same value as other enumerators. Any non-defined enumerators are given a value one greater than the previous enumerator.

```cpp
enum Animal {
    cat = -3,    // values can be negative
    dog,         // -2
    pig,         // -1
    horse = 5,
    giraffe = 5, // shares same value as horse
    chicken,     // 6
};
```

Note in this case, `horse` and `giraffe` have been given the same value. When this happens, the enumerators become non-distinct, essentially, `horse` and `giraffe` are interchangeable. 


If an enumeration is zero-initialised (which happens when we use value-initialisation), the enumeration will be given value `0`, even if there is no corresponding enumerator with that value.

```cpp
// Animal defined above
int main() {
    Animal a {}; // value-initialization zero-initializes a to value 0
    std::cout << a; // prints 0
}
```

So if there is an enumerator with value 0, value-initialisation defaults the enumeration to the meaning of that enumerator.

The best practice is to make the enumerator representing 0 the one that is the best default meaning for your enumeration. If no good default meaning exists, consider adding an “invalid” or “unknown” enumerator that has value 0. Example

```cpp
enum Winner {
    winnerUnknown, // default value (0)
    player1,
    player2,
};

// somewhere later in your code
if (w == winnerUnknown) // handle case appropriately
```


Even though enumerations store integral values, they are not considered to be an integral type (they are a compound type). However, an unscoped enumeration will implicitly convert to an integral value. Because enumerators are compile-time constants, this is a constexpr conversion.

Consider the following program:

```cpp

// Color is defined above
int main() {
    Color shirt{ blue };
    cout << "Your shirt is " << shirt << '\n'; // what does this do?
    // prints 2
}
```

When an enumerated type is used in a function call or with an operator, the compiler will first try to find a function or operator that matches the enumerated type. For example, when the compiler tries to compile `std::cout << shirt`, the compiler will first look to see if `operator<<` knows how to print an object of type `Color` (because `shirt` is of type `Color`) to `std::cout`. Since the compiler can’t find a match, it will then then check if `operator<<` knows how to print an object of the integral type that the un-scoped enumeration converts to. Since it does, the value in `shirt` gets converted to an integral value and printed as integral value `2`.

---
#### Enumeration size and underlying type (base) 

Enumerators have values that are of an integral type. The specific integral type used to represent the value of enumerators is called the enumeration’s **underlying type** (or **base**).

For un-scoped enumerations, the C++ standard does not specify which specific integral type should be used as the underlying type, so the choice is implementation-defined. Most compilers will use `int` as the underlying type (meaning an un-scoped enum will be the same size as an `int`), unless a larger type is required to store the enumerator values. But you shouldn’t assume this will hold true for every compiler or platform.  It is possible to explicitly specify an underlying type for an enumeration. The underlying type must be an integral type. 

```cpp
#include <cstdint>  // for std::int8_t
#include <iostream>

// Use an 8-bit integer as the enum underlying type
enum Color : std::int8_t { black, red,  blue, };

int main() {
    Color c{ black };
    std::cout << sizeof(c) << '\n'; // prints 1 (byte)
}
```

Because `std::int8_t` and `std::uint8_t` are usually type aliases for char types, using either of these types as the enum base will most likely cause the enumerators to print as char values rather than int values.

While the compiler will implicitly convert an unscoped enumeration to an integer, it will _not_ implicitly convert an integer to an unscoped enumeration. 

```cpp
Color color { 2 }; 
// compile error: integer value 2 won't implicitly convert to a color a Pet
```

But you can explicitly convert an integer to an unscoped enumerator using `static_cast`:

```cpp
enum Pet {
    cat, // assigned 0
    dog, // assigned 1
    pig, // assigned 2
    whale, // assigned 3
};

int main() {
    Pet pet { static_cast<Pet>(2) }; // convert integer 2 to a Pet
    pet = static_cast<Pet>(3);  // our pig evolved into a whale!
}
```

It is  safe to static_cast any integral value that is in range of the target enumeration’s underlying type, even if there are no enumerators representing that value. Static casting a value outside the range of the underlying type will result in undefined behavior.

If the enumeration has an explicitly defined underlying type, the range of the enumeration is identical to the range of the underlying type. If the enumeration does not have an explicit underlying type, things are a bit more complicated. In this case, the compiler gets to pick the underlying type, and it can pick any signed or unsigned type so long as the value of all enumerators fit in that type. Given this, it is only safe to static_cast integral values that fit in the range of the smallest number of bits that can hold the value of all enumerators.

- With enumerators that have values 2, 9, and 12, these enumerators could minimally fit in an unsigned 4-bit integral type with range 0 to 15. Therefore, it is only safe to static_cast integral values 0 through 15 to this enumerated type.
- With enumerators that have values -28, 2, and 6, these enumerators could minimally fit in a signed 6-bit integral type with range -32 to 31. Therefore, it is only safe to static_cast integral values -32 through 31 to this enumerated type.

As of C++17, if an unscoped enumeration has an explicitly specified base, then the compiler will allow you to list initialize an unscoped enumeration using an integral value:

```cpp
enum Pet: int {
    cat, // assigned 0
    dog, // assigned 1
    pig, // assigned 2
    whale, // assigned 3
};

int main() {
    Pet pet1 { 2 }; // ok: can brace initialize unscoped enumeration with specified base with integer (C++17)
    Pet pet2 (2);   // compile error: cannot direct initialize with integer
    Pet pet3 = 2;   // compile error: cannot copy initialize with intege
    pet1 = 3;       // compile error: cannot assign with integer

}
```

---
#### Converting an enumeration to and from a string

One way is to write switch-case statements;

```cpp
#include <iostream>
#include <string_view>

enum Color { black, red,  blue, };
constexpr std::string_view getColorName(Color color) {
    switch (color) {
    case black: return "black";
    case red:   return "red";
    case blue:  return "blue";
    default:    return "???";
    }
}

int main() {
    constexpr Color shirt{ blue };
    std::cout << "Your shirt is " << getColorName(shirt) << '\n';
}
```

A reminder: Because C-style string literals exist for the entire program, it’s okay to return a `std::string_view` that is viewing a C-style string literal. When the `std::string_view` is copied back to the caller, the C-style string literal being viewed will still exist. **(did not understand)**

The function is constexpr so that we can use the color’s name in a constant expression.

The second way to solve the program of mapping enumerators to strings is to use an array. We cover this in lesson [17.6 -- std::array and enumerations](https://www.learncpp.com/cpp-tutorial/stdarray-and-enumerations/).

---
#### Un-scoped enumerator input

```cpp
enum Pet { cat, dog, pig, whale, };

int main() {
    Pet pet { pig };
    std::cin >> pet; // compile error: std::cin doesn't know how to input a Pet
}
```

One simple way to work around this is to read in an integer, and use `static_cast` to convert the integer to an enumerator of the appropriate enumerated type:

```cpp
constexpr std::string_view getPetName(Pet pet) {
    switch (pet) {
    case cat:   return "cat";
    case dog:   return "dog";
    case pig:   return "pig";
    case whale: return "whale";
    default:    return "???";
    }
}

int main() {
    std::cout << "Enter a pet (0=cat, 1=dog, 2=pig, 3=whale): ";
    int input{};
    std::cin >> input; // input an integer
    if (input < 0 || input > 3) // This is improtant as discussed above
        std::cout << "You entered an invalid pet\n";
    else {
        Pet pet{ static_cast<Pet>(input) }; // static_cast our integer to a Pet
        std::cout << "You entered: " << getPetName(pet) << '\n';
    }
}
```

It would be nicer if the user could type in a string representing an enumerator (e.g. “pig”), and we could convert that string into the appropriate `Pet` enumerator. But firstly, we can’t switch on a string, so we need to use something else to match the string the user passed in. The simplest approach here is to use a series of if-statements. Secondly, what `Pet` enumerator should we return if the user passes in an invalid string? One option would be to add an enumerator to represent “none/invalid”, and return that. However, a better option is to use `std::optional` here.

```cpp
#include <iostream>
#include <optional> // for std::optional
#include <string>
#include <string_view>

constexpr std::optional<Pet> getPetFromString(std::string_view sv) {
    // We can only switch on an integral value (or enum), not a string
    if (sv == "cat")   return cat;
    if (sv == "dog")   return dog;
    if (sv == "pig")   return pig;
    if (sv == "whale") return whale;
    return {};
}

int main() {
    std::cout << "Enter a pet: cat, dog, pig, or whale: ";
    std::string s{};
    std::cin >> s;
    std::optional<Pet> pet { getPetFromString(s) };
    if (!pet) std::cout << "You entered an invalid pet\n";
    else std::cout << "You entered: " << getPetName(*pet) << '\n';
}
```

---
#### Overloading `operator<<` to print an enumerator 

Consider a simple expression like `std::cout << 5`. `std::cout` has type `std::ostream` (which is a user-defined type in the standard library), and `5` is a literal of type `int`.

When this expression is evaluated, the compiler will look for an overloaded `operator<<` function that can handle arguments of type `std::ostream` and `int`. It will find such a function (also defined as part of the standard I/O library) and call it. Inside that function, `std::cout` is used to output `x` to the console. Finally, the `operator<<` function returns its left-operand (which in this case is `std::cout`), so that subsequent calls to `operator<<` can be chained.


```cpp
#include <iostream>
#include <string_view>

// std::ostream is the type of std::cout, std::cerr, etc...
// The return type and parameter type are references (to prevent copies from being made)
std::ostream& operator<<(std::ostream& out, Color color)
{
    out << getColorName(color); 
    // print our color's name to whatever output stream out
    return out; // operator<< conventionally returns its left operand
    // The above can be condensed to the following single line:
    // return out << getColorName(color)
}

int main() {
	Color shirt{ blue };
	std::cout << "Your shirt is " << shirt << '\n'; // it works!
}
```

Let’s unpack our overloaded operator function a bit. First, the name of the function is `operator<<`, since that is the name of the operator we’re overloading. `operator<<` has two parameters. The left parameter (which will be matched with the left operand) is our output stream, which has type `std::ostream`. We use pass by non-const reference here because we don’t want to make a copy of a `std::ostream` object when the function is called, but the `std::ostream` object needs to be modified in order to do output. The right parameter (which will be matched with the right operand) is our `Color` object. Since `operator<<` conventionally returns its left operand, the return type matches the type of the left-operand, which is `std::ostream&`.

Now let’s look at the implementation. A `std::ostream` object already knows how to print a `std::string_view` using `operator<<` (this comes as part of the standard library). So `out << getColorName(color)` simply fetches our color’s name as a `std::string_view` and then prints it to the output stream.

---
#### Overloading `operator>>` to input an enumerator 

Similar to how we were able to teach `operator<<` to output an enumeration above, we can also teach `operator>>` how to input an enumeration:

```cpp
#include <iostream>
#include <limits>
#include <optional>
#include <string>
#include <string_view>

std::istream& operator>>(std::istream& in, Pet& pet) {
    std::string s{};
    in >> s; // get input string from user

    std::optional<Pet> match { getPetFromString(s) };
    if (match) {
        pet = *match; // dereference std::optional to get matching enumerator
        return in;
    }
    // We didn't find a match, so input must have been invalid
    // so we will set input stream to fail state
    in.setstate(std::ios_base::failbit);
    // On an extraction failure, operator>> zero-initializes fundamental types
    // Uncomment the following line to make this operator do the same thing
    // pet = {};
    return in;
}

int main() {
    std::cout << "Enter a pet: cat, dog, pig, or whale: ";
    Pet pet{};
    std::cin >> pet;
    if (std::cin) // if we found a match
        std::cout << "You chose: " << getPetName(pet) << '\n';
    else {
        std::cin.clear(); // reset the input stream to good
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
        std::cout << "Your pet was not valid\n";
    }
}
```

 The `pet` parameter is a non-const reference. This allows our `operator>>` to modify the value of the right operand that is passed in if our extraction results in a match.

If the user did not enter a valid pet, then we handle that case by putting `std::cin` into “failure mode”. This is the state that `std::cin` typically goes into when an extraction fails. The caller can then check `std::cin` to see if the extraction succeeded or failed.

We show how we can use `std::array` to make our input and output operators less redundant, and avoid having to modify them when a new enumerator is added in [[Arrays]].

---
### Issue with un-scoped enumerations

 Consider the following case:

```cpp

int main() {
    enum Color { red, blue, };
    enum Fruit { banana, apple, };
    Color color { red };
    Fruit fruit { banana };
    if (color == fruit) // The compiler will compare color and fruit as integers
        std::cout << "color and fruit are equal\n"; // and find they are equal!
    else
        std::cout << "color and fruit are not equal\n";
}
// output
// color and fruit are euqal
```

When `color` and `fruit` are compared, the compiler will look to see if it knows how to compare a `Color` and a `Fruit`. It doesn’t. Next, it will try converting `Color` and/or `Fruit` to integers to see if it can find a match. Eventually the compiler will determine that if it converts both to integers, it can do the comparison. Since `color` and `fruit` are both set to enumerators that convert to integer value `0`, `color` will equal `fruit`.

---
### Scoped enumerations

Also called enum class.
Scoped enumerations work similarly to unscoped enumerations, but have two primary differences: 
1. They won’t implicitly convert to integers.
2. The enumerators are _only_ placed into the scope region of the enumeration

To make a scoped enumeration, we use the keywords `enum class`.

```cpp
int main() {
    enum class Color { red, blue, };
    enum class Fruit { banana,  apple, }; 

    Color color { Color::red }; 
    Fruit fruit { Fruit::banana }; 
    if (color == fruit) // compile error: the compiler doesn't know how to compare different types Color and Fruit
        std::cout << "color and fruit are equal\n";
    else
        std::cout << "color and fruit are not equal\n";
}
```

Although scoped enumerations use the `class` keyword, they aren’t considered to be a “class type” (which is reserved for structs, classes, and unions). `enum struct` also works in this context, and behaves identically to `enum class`. However, use of `enum struct` is non-conventional.


Note that you can still compare enumerators from within the same scoped enumeration (since they are of the same type):

```cpp
int main() {
	enum class Color { red, blue, };
    Color shirt { Color::red };
    if (shirt == Color::red) // this Color to Color comparison is okay
        std::cout << "The shirt is red!\n";
    else if (shirt == Color::blue)
        std::cout << "The shirt is blue!\n";
}
```

You can explicitly convert a scoped enumerator to an integer by using a `static_cast`. A better choice in C++23 is to use `std::to_underlying()` (defined in the `<utility>` header), which converts an enumerator to a value of the underlying type of the enumeration.

```cpp
#include <iostream>
#include <utility> // for std::to_underlying() (C++23)

int main(){
    enum class Color { red, blue, };
    Color color { Color::blue };
    std::cout << color << '\n'; 
    // won't work, because there's no implicit conversion to int
    std::cout << static_cast<int>(color) << '\n';   
    // explicit conversion to int, will print 1
    std::cout << std::to_underlying(color) << '\n'; 
    // convert to underlying type, will print 1 (C++23)
}
```

Conversely, you can also `static_cast` an integer to a scoped enumerator, which can be useful when doing input from users:

```cpp
int input{};
std::cin >> input; // input an integer
Pet pet{ static_cast<Pet>(input) }; // static_cast our integer to a Pet
```

As of C++17, you can list initialize a scoped enumeration using an integral value without the static_cast (and unlike an unscoped enumeration, you don’t need to specify a base):

```cpp
Pet pet { 1 }; // okay
```


For cases where it would be useful to make conversion of scoped enumerators to integers easier, a useful hack is to overload the unary `operator+` to perform this conversion:

```cpp
#include <iostream>
#include <type_traits> // for std::underlying_type_t

enum class Animals {
    chicken, // 0
    dog, // 1
    cat, // 2
    elephant, // 3
    duck, // 4
    snake, // 5
    maxAnimals,
};

// Overload the unary + operator to convert an enum to the underlying type
template <typename T>
constexpr auto operator+(T a) noexcept {
    return static_cast<std::underlying_type_t<T>>(a);
    // In C++23, you can #include <utility> and return std::to_underlying(a) instead
}

int main() {
    std::cout << +Animals::elephant << '\n'; 
    // convert Animals::elephant to an integer using unary operator+
}
```

---
#### `using enum` statements C++20

Introduced in C++20, a `using enum` statement imports all of the enumerators from an enum into the current scope. When used with an enum class type, this allows us to access the enum class enumerators without having to prefix each with the name of the enum class.


```cpp
#include <iostream>
#include <string_view>

enum class Color { black, red, blue, };
constexpr std::string_view getColor(Color color) {
    using enum Color; 
    switch (color) {
    case black: return "black"; // note: black instead of Color::black
    case red:   return "red";
    case blue:  return "blue";
    default:    return "???";
    }
}

```

---
