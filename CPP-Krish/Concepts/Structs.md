In C++, a **member** is a variable, function, or type that belongs to a struct (or class). All members must be declared within the struct (or class) definition. 

```cpp
struct Employee{
    int id {};
    int age {};
    double wage {};
};
```

Object Initialization:
The empty braces is for value-initialization of the object.

Value initialization:
1. If default constructor given then that is called.
2. If default constructor not given:
	- Variables having initializers are initialized according to those values
	- Variables not having initializers are value initialized

If not value initialized:
1. If default constructor given then that is called
2. If default constructor not given:
	- Variables having initializers are initialized according to those values
	- Variables not having initializers are left with garbage values, except for class object which call their own default constructors

```cpp
struct Employee{
    int id {99};
    int age;
    double wage {};
    std::string name;
};

Employee emp{}; // id=99, age=0, wage=0.0, name="";
Employee emp1; //id=99, age=garbage, wage=0.0, name="";
```

---
### Aggregates

In general programming, an **aggregate data type** is any type that can contain multiple data members. Some types of aggregates allow members to have different types (e.g. structs), while others require that all members must be of a single type (e.g. arrays).

In C++, the definition of an aggregate is narrower and quite a bit more complicated. Simplified:
An aggregate in C++ is either a C-style array or a class type (struct, class, or union) that has:
- No user-declared constructors 
- No private or protected non-static data members
- No virtual functions

You can find the precise definition of a C++ aggregate [here](https://en.cppreference.com/w/cpp/language/aggregate_initialization).
`std::array` is also an aggregate.
The key thing to understand at this point is that structs with only data members are aggregates.

Aggregates use a form of initialization called **aggregate initialization**, which allows us to directly initialize the members of aggregates.  There are 2 primary forms of aggregate initialization:

```cpp
struct Employee {
    int id {};
    int age {};
    double wage {};
};

Employee frank = { 1, 32, 60000.0 }; 
// copy-list initialization using braced list
Employee joe { 2, 28, 45000.0 };     
// list initialization using braced list (preferred)
```

In C++20, we can also initialize (some) aggregates using a parenthesized list of values:
```cpp
Employee robert ( 3, 45, 62500.0 ); 
 // direct initialization using parenthesized list (C++20)
```
We recommend avoiding this last form, as it doesn’t currently work with aggregates that utilize brace elision (notably for multidimensional `std::array`, we can omit the inner brackets while initialization).


If an aggregate is initialized but the number of initialization values is fewer than the number of members, then each member without an explicit initializer is initialized as follows:
- If the member has a default member initializer, that is used.
- Otherwise, the member is copy-initialized from an empty initializer list. In most cases, this will perform value-initialization on those members (on class types, this will invoke the default constructor even if a list constructor exists (this is another type of constructor)).

---
#### Const Struct Objects

Variables of a struct type can be const (or constexpr), and just like all const variables, they must be initialized.

---
#### Designated Initializers

When initializing a struct from a list of values, the initializers are applied to the members in order of declaration. Now consider what would happen if you were to update the struct definition to add a new member that is not the last member.

To help avoid this, C++20 added **designated initializers**. It allow you to explicitly define which initialization values map to which members. The ***members*** can be initialized using list or copy initialization, and must be initialized in the same order in which they are declared in the struct, otherwise a warning or error will result. Members not designated an initializer will be value initialized.

```cpp
struct Foo {
    int a{};
    int b{};
    int c{};
};

Foo f1{ .a{ 1 }, .c{ 3 } };
// ok: f1.a = 1, f1.b = 0(value initialized), f1.c = 3
Foo f2{ .a = 1, .c = 3 }; 
// ok: f2.a = 1, f2.b = 0 (value initialized), f2.c = 3
Foo f3{ .b{ 2 }, .a{ 1 } };
// error: initialization order does not match order of declaration in struct
```

Similar to initializing a struct with an initializer list, you can also assign values to structs using an initializer list (which does member-wise assignment):

```cpp
struct Employee {
    int id {};
    int age {};
    double wage {};
};

Employee joe { 1, 32, 60000.0 };
joe = { joe.id, 33, 66000.0 }; // Joe had a birthday and got a raise
joe = { .id = joe.id, .age = 33, .wage = 66000.0 }; 
// Designated initializers can also be used in a list assignment
// Any members that aren’t designated in such an assignment will be assigned the value that would be used for value initialization. 
```


---
#### Initializing a struct with another struct of the same type
A struct may also be initialized using another struct of the same type
```cpp
Foo x = foo; // copy-initialization
Foo y(foo);  // direct-initialization
Foo z {foo}; // direct-list-initialization
```

---
#### Default member initialization

Explicit values in object's list initializer always take precedence over default member initialization values. (Obvious)

If an aggregate is defined with an initialization list:
- If an explicit initialization value exists, that explicit value is used.
- If an initializer is missing and a default member initializer exists, the default is used.
- If an initializer is missing and no default member initializer exists, value initialization occurs.

If an aggregate is defined with no initialization list:
- If a default member initializer exists, the default is used.
- If no default member initializer exists, the member remains uninitialized (or in a class type, default constructor is called, if default constructor does not exist compilation error is thrown).

Members are always initialized in the order of declaration.

It’s not uncommon for programmers to use default initialization instead of value initialization for class types. This is partly for historic reasons (as value initialization wasn’t introduced until C++11), and partly because there is a particular case (for non-aggregates) where default initialization can be more efficient than value initialization (we cover this case in lesson [14.11 -- Default constructors and default arguments](https://www.learncpp.com/cpp-tutorial/default-constructors-and-default-arguments/)).

----
### Passing and Returning Structs
#### Passing temporary structs

```cpp
struct Employee {
    int id {};
    int age {};
    double wage {};
};

void printEmployeeID(const Employee& employee) { // pass by reference
    std::cout << "ID:   " << employee.id << '\n';
}

// Print Joe's information
printEmployee(Employee { 14, 32, 24.15 }); // construct a temporary Employee (type explicitly specified) (preferred)

std::cout << '\n';

// Print Frank's information
printEmployee({ 15, 28, 18.27 }); // construct a temporary Employee to pass to function (type deduced from parameter)
```

In the second call, the compiler does implicit conversion to `Employee`. Note that this syntax won't work in cases where only explicit conversions are acceptable.

Structs defined inside functions are usually returned by value, so as not to return a dangling reference. We can make our functions slightly better by returning a temporary object instead:

```cpp
Point3d getZeroPoint() {
    return Point3d { 0.0, 0.0, 0.0 }; // return an unnamed Point3d
}
```

In this case, a temporary `Point3d` is constructed, copied back to the caller, and then destroyed at the end of the expression. Note this is much cleaner then first creating a temp and then returning it. We discuss anonymous objects in more detail in lesson [14.13 -- Temporary class objects](https://www.learncpp.com/cpp-tutorial/temporary-class-objects/).

In the case where the function has an explicit return type (e.g. `Point3d`), we can even omit the type in the return statement:

```cpp
Point3d getZeroPoint() {
    // We can use empty curly braces to value-initialize all members
    return {};
    // implicit conversion to Point3D
}

```

---
### Structs with program-defined members

In C++, structs (and classes) can have members that are other program-defined types. There are two ways to do this.

1. We can define one program-defined type (in the global scope) and then use it as a member of another program-defined type.
2. Types can also be nested inside other types, so if an Employee only existed as part of a Company, the Employee type could be nested inside the Company struct.

---
### Struct size and data structure alignment

Typically, the size of a struct is the sum of the size of all its members, but not always.

```cpp
struct Foo {
    short a {};
    int b {};
    double c {};
};

std::cout << "The size of short is " << sizeof(short) << " bytes\n";
std::cout << "The size of int is " << sizeof(int) << " bytes\n";
std::cout << "The size of double is " << sizeof(double) << " bytes\n";

std::cout << "The size of Foo is " << sizeof(Foo) << " bytes\n";
```

Prints:

```cpp
The size of short is 2 bytes
The size of int is 4 bytes
The size of double is 8 bytes
The size of Foo is 16 bytes
```

Note that the size of `short` + `int` + `double` is 14 bytes, but the size of `Foo` is 16 bytes!

For performance reasons, the compiler will sometimes add gaps into structures (this is called **padding**). So we can only say that the size of a struct will be _at least_ as large as the size of all the variables it contains.

In the `Foo` struct above, the compiler is invisibly adding 2 bytes of padding after member `a`, making the size of the structure 16 bytes instead of 14. 

This can actually have a pretty significant impact on the size of the struct, as the following program demonstrates:

```cpp
struct Foo1 {
    short a{}; // will have 2 bytes of padding after a
    int b{};
    short c{}; // will have 2 bytes of padding after c
};

struct Foo2 {
    int b{};
    short a{};
    short c{};
};

std::cout << sizeof(Foo1) << '\n'; // prints 12
std::cout << sizeof(Foo2) << '\n'; // prints 8
```

Note that `Foo1` and `Foo2` have the same members, the only difference being the declaration order. Yet `Foo1` is 50% larger due to the added padding.

Note:
- You can minimize padding by defining your members in decreasing order of size.
- The C++ compiler is not allowed to reorder members, so this has to be done manually.

Why is padding needed is a question that needs further study.

---
### Owning structs

In most cases, we want our structs (and classes) to be owners of the data they contain (or else we will have to deal with dangling pointers and stuff).

The easiest way to make a struct (or class) an owner is to give each data member a type that is an owner (e.g. not a viewer, pointer, or reference). If a struct or class has data members that are all owners, then the struct or class itself is automatically an owner.

This is why string data members are almost always of type `std::string` (which is an owner), and not of type `std::string_view` (which is a viewer).

---
