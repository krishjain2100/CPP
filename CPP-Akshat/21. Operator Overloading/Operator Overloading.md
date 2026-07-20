In C++, operators are implemented as functions. By using function overloading on the operator functions, you can define your own versions of the operators that work with different data types (including classes that you’ve written). Using function overloading to overload operators is called **operator overloading**.

### Resolving overloaded operators
When evaluating an expression containing an operator, the compiler uses the following rules:
- If _all_ of the operands are fundamental data types, the compiler will call a built-in routine if one exists. If one does not exist, the compiler will produce a compiler error.
- If _any_ of the operands are program-defined types (e.g. one of your classes, or an enum type), the compiler will use the function overload resolution algorithm to see if it can find an overloaded operator that is an unambiguous best match. This may involve implicitly converting one or more operands to match the parameter types of an overloaded operator. It may also involve implicitly converting program-defined types into fundamental types (via an overloaded typecast, which we’ll cover later in this chapter) so that it can match a built-in operator. If no match can be found (or an ambiguous match is found), the compiler will error.

- First, almost any existing operator in C++ can be overloaded. The exceptions are: conditional (?:), sizeof, scope (::), member selector (.), pointer member selector (.*), typeid, and the casting operators.
- Second, you can only overload the operators that exist. You can not create new operators or rename existing operators.
- Third, at least one of the operands in an overloaded operator must be a user-defined type. This means you could overload `operator+(int, Mystring)`, but not `operator+(int, double)`.
- Because standard library classes are considered to be user-defined, this means you could define `operator+(double, std::string)`. However, this is not a good idea because a future language standard could define this overload, which could break any programs that used your overload.
- Fourth, it is not possible to change the number of operands an operator supports.
- Finally, all operators keep their default precedence and associativity (regardless of what they’re used for) and this can not be changed.

Best Pracitces:
When overloading operators, it’s best to keep the function of the operators as close to the original intent of the operators as possible.
If the meaning of an overloaded operator is not clear and intuitive, use a named function instead.

Operators that do not modify their operands (e.g. arithmetic operators) should generally return results by value.

Operators that modify their leftmost operand (e.g. pre-increment, any of the assignment operators) should generally return the leftmost operand by reference.

---
# 2. Arithmetic operator using friend functions

It turns out that there are three different ways to overload operators: the member function way, the friend function way, and the normal function way.

Remember that if you make the arguments non const references, then you wouldn't be able to use them for temporary objects which is a feature we mostly want so make them const references.

After that nothing noteworthy in the whole lecture
Just that you can define arithmetic operators using friend functions

---
# 3. Overloading operators using normal functions

nothing much here too
Just that you have to forward declare you operator functions everywhere or include them in the .h files.
Prefer overloading operators as normal functions instead of friends if it’s possible to do so without adding additional functions(the less functions touching your classes’s internals, the better).

---
# 4. Overloading the I/O operators
A print() is a good choice for outputting classes but it can't be called while printing normally using cout. You'll have to do something like this:
```cpp
const Point point { 5.0, 6.0, 7.0 };

std::cout << "My point is: ";
point.print();
std::cout << " in Cartesian space.\n";
```

### Overloading `operator<<`
The left operand is the `std::cout` object, and the right operand is your `Point` class object. `std::cout` is actually an object of type `std::ostream`. Therefore, our overloaded function will look like this:
```cpp
// std::ostream is the type for object std::cout
friend std::ostream& operator<< (std::ostream& out, const Point& point);
```

Make sure to return by referenece. 
If you try to return `std::ostream` by value, you’ll get a compiler error. This happens because `std::ostream` specifically disallows being copied.
Chaining is the reason for returning the left operand.

### Overloading `operator>>`
This is done in a manner analogous to overloading the output operator. The key thing you need to know is that `std::cin` is an object of type `std::istream`.
Partial extraction issues might come, like in the Point class(which has 3 double variables internally).
```cpp
std::istream& operator>> (std::istream& in, Point& point)
{
    // This version subject to partial extraction issues (see below)
    in >> point.m_x >> point.m_y >> point.m_z;

    return in;
}

Point point{ 1.0, 2.0, 3.0 }; // non-zero test data
std::cin >> point;

std::cout << "You entered: " << point << '\n';
```

Now let’s see what happens when the user enters `4.0b 5.6 7.26` as input (notice the `b` after the `4.0`):  You entered: Point(4, 0, 3)
I think its pretty clear what happened here
The extraction to `x_y` successfully extracts `4.0` from the user’s input, leaving `b 5.6 7.26` in the input stream. The extraction to `m_y` fails to extract `b`, so `m_y` is copy-assigned the value `0.0` and the input stream is set to failure mode. Since we haven’t cleared failure mode, the extraction to `m_z` aborts immediately,

So how might we avoid this? One way is to make our operations transactional. A **transactional operation** must either completely succeed or completely fail -- no partial successes or failures are allowed. This is sometimes known as “all or nothing”. If a failure occurs at any point during the transaction, prior changes made by the operation must be undone.
Let’s reimplement our overloaded `Point` `operator>>` as a transactional operation:
```cpp
// note that point must be non-const so we can modify the object
// note that this implementation is a non-friend
std::istream& operator>> (std::istream& in, Point& point){
    double x{};
    double y{};
    double z{};

    if (in >> x >> y >> z)      // if all extractions succeeded
        point = Point{x, y, z}; // overwrite our existing point

    return in;
}
```

While the above `operator>>` prevents partial extractions, it is inconsistent with how `operator>>` works for fundamental types. When extraction to an object with a fundamental type fails, the object isn’t left unaltered -- it is copy assigned the value `0` (this ensures the object has some consistent value in case it wasn’t initialized before the extraction attempt). Therefore, for consistency, you may wish to have a failed extraction reset the object to its default state (at least in cases where such a thing exists).
Such an operation is technically no longer transactional (because failure doesn’t “do nothing”).

### Handling semantically invalid input
To address this, we can have our overloaded `operator>>` determine whether any of the values that were extracted are semantically invalid, and if so, manually put the input stream in failure mode. This can be done by calling `std::cin.setstate(std::ios_base::failbit);`.
```cpp
in.setstate(std::ios_base::failbit); // set failure mode manually
```

---
# 5. Overloading operators using member functions
Overloading operators using a member function is very similar to overloading operators using a friend function. When overloading an operator using a member function:
- The overloaded operator must be added as a member function of the left operand.
- The left operand becomes the implicit *this object
- All other operands become function parameters.
```cpp
// Cents is a class with a member variable m_cents
Cents Cents::operator+ (int value) const
{
    return Cents { m_cents + value };
}
```

In the friend function version, the expression `cents1 + 2` becomes function call operator+(cents1, 2).
In the member function version, the expression `cents1 + 2` becomes function call `cents1.operator+(2)`. We mentioned that the compiler implicitly converts an object prefix into a hidden leftmost parameter named *this. So in actuality, `cents1.operator+(2)` becomes `operator+(&​cents1, 2)`, which is almost identical to the friend version.

###### So if we can overload an operator as a friend or a member, which should we use? In order to answer that question, there’s a few more things you’ll need to know.

**Not everything can be overloaded as a friend function**
The assignment (=), subscript ([]), function call (()), and member selection (->) operators must be overloaded as member functions, because the language requires them to be.

**Not everything can be overloaded as a member function**
Like the << operator where the left operand are std::iostream. std::iostream is fixed as part of the standard library. We can’t modify the class declaration to add the overload as a member function of std::iostream. 
Similarly, although we can overload operator+(Cents, int) as a member function (as we did above), we can’t overload operator+(int, Cents) as a member function

Typically, we won’t be able to use a member overload if the left operand is either not a class (e.g. int), or it is a class that we can’t modify (e.g. std::ostream).

#### When to use a normal, friend, or member function overload
In most cases, the language leaves it up to you to determine whether you want to use the normal/friend or member function version of the overload. However, one of the two is usually a better choice than the other.

When dealing with binary operators that don’t modify the left operand (e.g. operator+), the normal or friend function version is typically preferred, because it works for all parameter types (even when the left operand isn’t a class object, or is a class that is not modifiable). The normal or friend function version has the added benefit of “symmetry”, as all operands become explicit parameters (instead of the left operand becoming *this and the right operand becoming an explicit parameter).

When dealing with binary operators that do modify the left operand (e.g. operator+=), the member function version is typically preferred. In these cases, the leftmost operand will always be a class type, and having the object being modified become the one pointed to by *this is natural. Because the rightmost operand becomes an explicit parameter, there’s no confusion over who is getting modified and who is getting evaluated.

Unary operators are usually overloaded as member functions as well, since the member version has no parameters.

The following rules of thumb can help you determine which form is best for a given situation:
- If you’re overloading assignment (=), subscript ([]), function call (()), or member selection (->), do so as a member function.
- If you’re overloading a unary operator, do so as a member function.
- If you’re overloading a binary operator that does not modify its left operand (e.g. operator+), do so as a normal function (preferred) or friend function.
- If you’re overloading a binary operator that modifies its left operand, but you can’t add members to the class definition of the left operand (e.g. operator<<, which has a left operand of type ostream), do so as a normal function (preferred) or friend function.
- If you’re overloading a binary operator that modifies its left operand (e.g. operator+=), and you can modify the definition of the left operand, do so as a member function.

---
# 6. Overloading unary operators +, -, and !
The positive (+), negative (-) and logical not (!) operators all are unary operators. Because they only operate on the object they are applied to, typically unary operator overloads are implemented as member functions.

For integers, 0 evaluates to false, and anything else to true, so operator! as applied to integers will return true for an integer value of 0 and false otherwise.
Extending the concept, we can say that operator! should evaluate to true if the state of the object is “false”, “zero”, or whatever the default initialization state is.

unary + returns by value usually.
Nothing usefule after this.

---
# 7. Overloading the comparison operators
Because the comparison operators are all binary operators that do not modify their left operands, we will make our overloaded comparison operators friend functions.
For == and !=, the code should be straightforward.
< and > might not make sense for many classes. for eg for Cars
However, there is one common exception to the above recommendation. What if we wanted to sort a list of Cars? In such a case, we might want to overload the comparison operators to return the member (or members) you’re most likely to want to sort on. For example, an overloaded operator< for Cars might sort based on make and model alphabetically.
Some of the container classes in the standard library (classes that hold sets of other classes) require an overloaded operator< so they can keep the elements sorted.

Overloaded comparison operators tend to have a high degree of redundancy, and the more complex the implementation, the more redundancy there will be.
Fortunately, many of the comparison operators can be implemented using the other comparison operators:
- operator!= can be implemented as !(operator== )
- operator> can be implemented as operator< with the order of the parameters flipped
- operator>= can be implemented as !(operator<)
- operator<= can be implemented as !(operator>)
This means that we only need to implement logic for operator== and operator<, and then the other four comparison operators can be defined in terms of those two!

### The spaceship operator <=> C++20
C++20 introduces the spaceship operator (`operator<=>`), which allows us to reduce the number of comparison functions we need to write down to 2 at most, and sometimes just 1!
Check the sytax online.

Remember std::sort needs < operator

---
# 8. Overloading the increment and decrement operators
Overloading the increment (`++`) and decrement (`--`) operators is pretty straightforward, with one small exception. There are actually two versions of the increment and decrement operators: a prefix and a postfix one.

Because the increment and decrement operators are both unary operators and they modify their operands, they’re best overloaded as member functions.

### Overloading prefix increment and decrement
Prefix increment and decrement are overloaded exactly the same as any normal unary operator.

### Overloading postfix increment and decrement
Normally, functions can be overloaded when they have the same name but a different number and/or different type of parameters. However, consider the case of the prefix and postfix increment and decrement operators. Both have the same name (eg. operator++), are unary, and take one parameter of the same type. So how it is possible to differentiate the two when overloading?

The C++ language specification has a special case that provides the answer: the compiler looks to see if the overloaded operator has an int parameter. If the overloaded operator has an int parameter, the operator is a postfix overload. If the overloaded operator has no parameter, the operator is a prefix overload.

```cpp
Digit& operator++(); // prefix has no parameter
Digit& operator--(); // prefix has no parameter

Digit operator++(int); // postfix has an int parameter
Digit operator--(int); // postfix has an int parameter
```

There are a few interesting things going on here. First, note that we’ve distinguished the prefix from the postfix operators by providing an integer dummy parameter on the postfix version. Second, because the dummy parameter is not used in the function implementation, we have not even given it a name. This tells the compiler to treat this variable as a placeholder, which means it won’t warn us that we declared a variable but never used it.

Third, note that the prefix and postfix operators do the same job -- they both increment or decrement the object. The difference between the two is in the value they return. The overloaded prefix operators return the object after it has been incremented or decremented.

The postfix operators, on the other hand, need to return the state of the object _before_ it is incremented or decremented.

In the postfix case we return a copy of the variable, while in the prefix one a reference to the original is returned

Also note that this means the postfix operators are typically less efficient than the prefix operators because of the added overhead of instantiating a temporary variable and returning by value instead of reference.

Remember that you can use prefix operator within postfix.

---
# 9. Overloading the subscript operator
```cpp
class IntList
{
private:
    int m_list[10]{};

public:
    int& operator[] (int index)
    {
        return m_list[index];
    }
};
```
The above is the best way to give access to the inside array.

Note that although you can provide a default value for the function parameter, actually using operator[] without a subscript inside is not considered a valid syntax, so there’s no point. It won't work even if you provide default value.

C++23 adds support for overloading operator[] with multiple subscripts.

#### Why operator[] returns a reference
To support the assignment operator.

### Overloaded operator[] for const objects
In the above IntList example, operator[] is non-const, and we can use it as an l-value to change the state of non-const objects. However, what if our IntList object was const? In this case, we wouldn’t be able to call the non-const version of operator[] because that would allow us to potentially change the state of a const object.

The good news is that we can define a non-const and a const version of operator[] separately. The non-const version will be used with non-const objects, and the const version with const-objects.

```cpp
int& operator[] (int index){
	return m_list[index];
}
// For const objects: can only be used for access
// This function could also return by value if the type is cheap to copy
const int& operator[] (int index) const{
	return m_list[index];
}
```

### Removing duplicate code between const and non-const overloads
Normally we’d simply implement one function in terms of the other (e.g. have one function call the other). But that’s a bit tricky in this case. The const version of the function can’t call the non-const version of the function, because that would require discarding the const of a const object. And while the non-const version of the function can call the const version of the function, the const version of the function returns a const reference, when we need to return a non-const reference. Fortunately, there is a way to work around this.
```cpp
int& operator[] (int index){
	// use std::as_const to get a const version of `this` (as a reference)
	// so we can call the const version of operator[]
	// then const_cast to discard the const on the returned reference
	return const_cast<int&>(std::as_const(*this)[index]);
}

const int& operator[] (int index) const{
	return m_list[index];
}
```

Normally using `const_cast` to remove const is something we want to avoid, but in this case it’s acceptable. If the non-const overload was called, then we know we’re working with a non-const object. It’s okay to remove the const on a const reference to a non-const object.

In C++23, we can do even better by making use of several features we cover later:
```cpp
auto&& operator[](this auto&& self, int index){
	// Complex code goes here
	return self.m_list[index];
}
```

### Detecting index validity
We can overload the operator to check index validity in normal arrays(make array inside a class).

### Pointers to objects and overloaded operator[] don’t mix
If you try to call operator[] on a pointer to an object, C++ will assume you’re trying to index an array of objects of that type.
```cpp
class IntList
{
private:
    int m_list[10]{};

public:
    int& operator[] (int index)
    {
        return m_list[index];
    }
};
IntList* list{ new IntList{} }; // assume 
list [2] = 3; // error: this will assume we're accessing index 2 of an array of IntLists
delete list;
```

Because we can’t assign an integer to an IntList, this won’t compile. However, if assigning an integer was valid, this would compile and run, with undefined results.

The proper syntax would be to dereference the pointer first (making sure to use parenthesis since operator[] has higher precedence than operator*), then call operator[].

### The function parameter does not need to be an integral type
As mentioned above, C++ passes what the user types between the hard braces as an argument to the overloaded function. In most cases, this will be an integral value. However, this is not required -- and in fact, you can define that your overloaded operator[] take a value of any type you desire. You could define your overloaded operator[] to take a double, a std::string, or whatever else you like.

A example to show a point:
```cpp
struct StudentGrade
{
	std::string name{};
	char grade{};
};

class GradeMap
{
private:
	std::vector<StudentGrade> m_map{};

public:
	char& operator[](std::string_view name);
};

char& GradeMap::operator[](std::string_view name)
{
	auto found{ std::find_if(m_map.begin(), m_map.end(),
				[name](const auto& student) { // this is a lambda that captures name from the surrounding scope
					return (student.name == name); // so we can use name here
				}) };

	if (found != m_map.end())
	{
		return found->grade;
	}
	return m_map.emplace_back(std::string{name}).grade;
}
int main()
{
	GradeMap grades{};

	char& gradeJoe{ grades["Joe"] }; // does an emplace_back
	gradeJoe = 'A';

	char& gradeFrank{ grades["Frank"] }; // does a emplace_back
	gradeFrank = 'B';

	std::cout << "Joe has a grade of " << gradeJoe << '\n';
	std::cout << "Frank has a grade of " << gradeFrank << '\n';

	return 0;
}
```

This might not work as expected because when a new element is added to the array, the vector may grow in size and allocates new memory(most probably somewhere else) so the old gradeJoe pointer will be left dangling.

---
# 10. Overloading the parenthesis operator
All of the overloaded operators you have seen so far let you define the type of the operator’s parameters, but not the number of parameters (which is fixed based on the type of the operator).
The parenthesis operator (operator()) is a particularly interesting operator in that it allows you to vary both the type AND number of parameters it takes.

There are two things to keep in mind: 
first, the parenthesis operator must be implemented as a member function. 
Second, in non-object-oriented C++, the () operator is used to call functions. In the case of classes, operator() is just a normal operator that calls a function (named operator()) like any other overloaded operator.

Prior to C++23, operator[] is limited to a single parameter, and therefore is not sufficient to let us directly index a two-dimensional array. Therefore we could declare a version of operator() that takes two parameters to allow access.(obviously putting the 2-d array in a class, lets say matrix).
Remember these are normal functions only so we can define it with different parameters and parameter types.

### Having fun with functors
Operator() is also commonly overloaded to implement **functors** (or **function object**), which are classes that operate like functions. The advantage of a functor over a normal function is that functors can store data in member variables (since they are classes).

```cpp
class Accumulator
{
private:
    int m_counter{ 0 };

public:
    int operator() (int i) { return (m_counter += i); }

    void reset() { m_counter = 0; } // optional
};
Accumulator acc{};
std::cout << acc(1) << '\n'; // prints 1
std::cout << acc(3) << '\n'; // prints 4

Accumulator acc2{};
std::cout << acc2(10) << '\n'; // prints 10
std::cout << acc2(20) << '\n'; // prints 30
```
Note that using our Accumulator looks just like making a normal function call.

---
# 11. Overloading typecasts
Let’s show how we overload a typecast to define a conversion from `Cents` to `int`:
```cpp
class Cents
{
private:
    int m_cents{};
public:
    Cents(int cents=0)
        : m_cents{ cents }
    {
    }

    // Overloaded int cast
    operator int() const { return m_cents; }

    int getCents() const { return m_cents; }
    void setCents(int cents) { m_cents = cents; }
};
```
There are a few things worth noting here:
- Overloaded typecasts must be non-static members, and should be `const` so they can be used with const objects.
- Overloaded typecasts do not have explicit parameters, as there is no way to pass explicit arguments to them. They do still have a hidden `*this` parameter, pointing to the implicit object (which is the object to be converted).
- Overloaded typecast do not declare a return type. The name of the conversion (e.g. int) is used as the return type, as it is the only return type allowed. This prevents redundancy in the declaration.

Now the compiler will be able to implicitly and explicitly convert Cents to int.
You can provide overloaded typecasts for any data type you wish, including your own program-defined data types!

### Explicit typecasts
Just like we can make constructors `explicit` so that they can’t be used for implicit conversions, we can also make our overloaded typecasts `explicit` for the same reason. Explicit typecasts can only be invoked by casting (e.g. `static_cast`) or by a form of direct initialization (either parenthesis or brace). They are not considered when doing copy-initialization.
```cpp
explicit operator int() const { return m_cents; } // now marked as explicit
```

Typecasts should be generally be marked as explicit. Exceptions can be made in cases where the conversion inexpensively converts to a similar user-defined type.

### When to use converting constructors vs overloaded typecasts
Overloaded typecasts and converting constructors perform similar functions:
- A converting constructor is a member function of class type B that defines how B is created from A.
- An overloaded typecast is a member function of class type A that defines how A is converted to B.

Since both ways require defining a member function, they can only be used for class types that can be modified. In cases where A and B are both class types that can be modified, we could use either. But since we need only one, which should we prefer?
In general, a converting constructor should be preferred to an overloaded typecast. All other things equal, it is cleaner for a class type to own its own construction, rather than rely on another class to create and initialize it.

There are a few cases where an overloaded typecast should be used instead:

- When providing a conversion to a fundamental type (since you can’t define constructors for these types). Most conventionally, these are used to provide a conversion to `bool` for cases where it makes sense to be able to use an object in a conditional statement.
- When the conversion returns a reference or const reference.
- When providing a conversion to a type you can’t add members to.
- When you do not want the type being constructed to be aware of the type being converted from. This can be helpful for avoiding circular dependencies.

For an example of that last bullet, `std::string` has a constructor to create a `std::string` from a `std::string_view`. This means `<string>` must include `<string_view>`. If `std::string_view` had a constructor to create a `std::string_view` from a `std::string`, then `<string_view>` would need to include `<string>`, and this would result in a circular dependency between headers.

When a converting constructor and an overloaded typecast are both defined for the same conversion, both are considered during overload resolution. Depending on whether the overloaded typecast is const, the object being converted is const, and what type of cast or initialization is used (copy vs direct), either function might be chosen (which can result a typecast being selected over a converting constructor), or the result can be ambiguous (resulting in a compile error). For this reason, you should avoid defining both an overloaded typecast and a converting constructor that can serve the same conversion.

When you need to define how convert type A to type B:
- If B is a class type you can modify, prefer using a converting constructor to create B from A.
- Otherwise, if A is a class type you can modify, use an overloaded typecast to convert A to B.
- Otherwise use a non-member function to convert A to B.

When each wins, please checkout

---
# 12. Overloading the assignment operator

### Copy assignment vs Copy constructor
The purpose of the copy constructor and the copy assignment operator are almost equivalent -- both copy one object to another. However, the copy constructor initializes new objects, whereas the assignment operator replaces the contents of existing objects.
Summarizing:
- If a new object has to be created before the copying can occur, the copy constructor is used (note: this includes passing or returning objects by value).
- If a new object does not have to be created before the copying can occur, the assignment operator is used.

### Overloading the assignment operator
Overloading the copy assignment operator (operator=) is fairly straightforward, with one specific caveat that we’ll get to. The copy assignment operator must be overloaded as a member function.

```cpp
// A simplistic implementation of operator= (see better implementation below)
Fraction& Fraction::operator= (const Fraction& fraction)
{
    // do the copy
    m_numerator = fraction.m_numerator;
    m_denominator = fraction.m_denominator;

    // return the existing object so we can chain this operator
    return *this;
}
```

### Issues due to self-assignment
Here’s where things start to get a little more interesting. C++ allows self-assignment:
```cpp
f1 = f1; // self assignment
```

This will call f1.operator=(f1), and under the simplistic implementation above, all of the members will be assigned to themselves. In this particular example, the self-assignment causes each member to be assigned to itself, which has no overall impact, other than wasting time. In most cases, a self-assignment doesn’t need to do anything at all!

However, in cases where an assignment operator needs to dynamically assign memory, self-assignment can actually be dangerous:

```cpp
class MyString
{
private:
	char* m_data {};
	int m_length {};

public:
	MyString(const char* data = nullptr, int length = 0 )
		: m_length { std::max(length, 0) }
	{
		if (length)
		{
			m_data = new char[static_cast<std::size_t>(length)];
			std::copy_n(data, length, m_data); // copy length elements of data into m_data
		}
	}
	~MyString()
	{
		delete[] m_data;
	}

	MyString(const MyString&) = default; // some compilers (gcc) warn if you have pointer members but no declared copy constructor

	// Overloaded assignment
	MyString& operator= (const MyString& str);

	friend std::ostream& operator<<(std::ostream& out, const MyString& s);
};

std::ostream& operator<<(std::ostream& out, const MyString& s)
{
	out << s.m_data;
	return out;
}

// A simplistic implementation of operator= (do not use)
MyString& MyString::operator= (const MyString& str)
{
	// if data exists in the current string, delete it
	if (m_data) delete[] m_data;

	m_length = str.m_length;
	m_data = nullptr;

	// allocate a new array of the appropriate length
	if (m_length)
		m_data = new char[static_cast<std::size_t>(str.m_length)];

	std::copy_n(str.m_data, m_length, m_data); // copies m_length elements of str.m_data into m_data

	// return the existing object so we can chain this operator
	return *this;
}
```
I think the problem is pretty obvious, that the data is deleted.

Fortunately, we can detect when self-assignment occurs.
```cpp
if (this == &str)
	return *this;
```

By checking if the address of our implicit object is the same as the address of the object being passed in as a parameter, we can have our assignment operator just return immediately without doing any other work.

### When not to handle self-assignment
Typically the self-assignment check is skipped for copy constructors. Because the object being copy constructed is newly created, the only case where the newly created object can be equal to the object being copied is when you try to initialize a newly defined object with itself

Second, the self-assignment check may be omitted in classes that can naturally handle self-assignment.(where nothing like the deletions before are happening, the fraction example above).

Because self-assignment is a rare event, some prominent C++ gurus recommend omitting the self-assignment guard even in classes that would benefit from it. We do not recommend this, as we believe it’s a better practice to code defensively and then selectively optimize later.

### The copy and swap idiom
If your class needs any of
- a **copy constructor**,
- an **assignment operator**,
- or a **destructor**,
defined explictly, then it is likely to need **all three of them**.

From C++11 on, an object has 2 extra special member functions: the move constructor and move assignment. The rule of five states to implement these functions as well.

Copy and swap idiom for copy assignment operator.
### A failed solution

Here's how a naive implementation might look:

```cpp
// the hard part
dumb_array& operator=(const dumb_array& other)
{
    if (this != &other) // (1)
    {
        // get rid of the old data...
        delete [] mArray; // (2)
        mArray = nullptr; // (2) *(see footnote for rationale)

        // ...and put in the new
        mSize = other.mSize; // (3)
        mArray = mSize ? new int[mSize] : nullptr; // (3)
        std::copy(other.mArray, other.mArray + mSize, mArray); // (3)
    }

    return *this;
}
```
And we say we're finished; this now manages an array, without leaks. However, it suffers from three problems, marked sequentially in the code as `(n)`.

1. The first is the self-assignment test. This check serves two purposes: it's an easy way to prevent us from running needless code on self-assignment, and it protects us from subtle bugs (such as deleting the array only to try and copy it). But in all other cases it merely serves to slow the program down, and act as noise in the code; self-assignment rarely occurs, so most of the time this check is a waste. It would be better if the operator could work properly without it.    
2. The second is that it only provides a basic exception guarantee. If `new int[mSize]` fails, `*this` will have been modified. (Namely, the size is wrong and the data is gone!) For a strong exception guarantee, it would need to be something akin to:
```cpp
dumb_array& operator=(const dumb_array& other)
 {
     if (this != &other) // (1)
     {
         // get the new data ready before we replace the old
         std::size_t newSize = other.mSize;
         int* newArray = newSize ? new int[newSize]() : nullptr; // (3)
         std::copy(other.mArray, other.mArray + newSize, newArray); // (3)

         // replace the old data (all are non-throwing)
         delete [] mArray;
         mSize = newSize;
         mArray = newArray;
     }

     return *this;
 }

```
3. The code has expanded! Which leads us to the third problem: code duplication.

### A successful solution
We need to add swap functionality to our class, and we do that as follows:
```cpp
class dumb_array
{
public:
    // ...

    friend void swap(dumb_array& first, dumb_array& second) // nothrow
    {
        // enable ADL (not necessary in our case, but good practice)
        using std::swap;

        // by swapping the members of two objects,
        // the two objects are effectively swapped
        swap(first.mSize, second.mSize);
        swap(first.mArray, second.mArray);
    }

    // ...
};

```
Without further ado, our assignment operator is:
```cpp
// we directly not take the other as reference, so copy is already made there
dumb_array& operator=(dumb_array other) // (1)
{
    swap(*this, other); // (2)

    return *this;
}
```
The swap function also makes it easy to make the move constructor.(didn't study it till now so maybe something wrong, check again later).

### The implicit copy assignment operator
Unlike other operators, the compiler will provide an implicit public copy assignment operator for your class if you do not provide a user-defined one. This assignment operator does memberwise assignment (which is essentially the same as the memberwise initialization that default copy constructors do).

Just like other constructors and operators, you can prevent assignments from being made by making your copy assignment operator private or using the delete keyword.

Note that if your class has const members, the compiler will instead define the implicit `operator=` as deleted. This is because const members can’t be assigned, so the compiler will assume your class should not be assignable.

---
# 13. Shallow vs. deep copying
### Shallow copying
Because C++ does not know much about your class, the default copy constructor and default assignment operators it provides use a copying method known as a memberwise copy (also known as a **shallow copy**). This means that C++ copies each member of the class individually (using the assignment operator for overloaded operator=, and direct initialization for the copy constructor). When classes are simple (e.g. do not contain any dynamically allocated memory), this works very well.
But with dynamically allocated memory, many problems arise.
### Deep copying
One answer to this problem is to do a deep copy on any non-null pointers being copied. A **deep copy** allocates memory for the copy and then copies the actual value, so that the copy lives in distinct memory from the source. This way, the copy and source are distinct and will not affect each other in any way. Doing deep copies requires that we write our own copy constructors and overloaded assignment operators.

### The rule of three
Remember the rule of three? If a class requires a user-defined destructor, copy constructor, or copy assignment operator, then it probably requires all three. This is why. If we’re user-defining any of these functions, it’s probably because we’re dealing with dynamic memory allocation. We need the copy constructor and copy assignment to handle deep copies, and the destructor to deallocate memory.

---
# 14. Overloading operators and function templates
Bakwas just ki templates might use operators which are not defined for the class, in those cases define those operators for the classes.

