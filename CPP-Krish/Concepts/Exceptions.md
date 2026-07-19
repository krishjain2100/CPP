One of the most common ways to handle potential errors is via return codes. For example:

```cpp
int findFirstChar(std::string_view string, char ch) {
    for (std::size_t index{ 0 }; index < string.length(); ++index)
        if (string[index] == ch)
            return index;
    return -1;
}
```

However, using return codes has a number of drawbacks:
 - Return values can be cryptic, if a function returns -1, it is hard to tell if it is indicating an error, or if it is a valid return value
 - Functions can only return one value, but sometimes you need to return both a function result and a possible error code. Example:

```cpp
double divide(int x, int y) { return static_cast<double>(x)/y; }
```

This will crash if the user passes in `0` for parameter `y`. However, it also needs to return the result of `x/y`. We will have to use an out-parameter. For example:

```cpp
double divide(int x, int y, bool& outSuccess) {
    if (y == 0) {
        outSuccess = false;
        return 0.0;
    }
    outSuccess = true;
    return static_cast<double>(x)/y;
}

int main() {
    bool success {}; 
    double result { divide(5, 3, success) };
    if (!success)  std::cerr << "An error occurred" << std::endl;
    else std::cout << "The answer is " << result << '\n';
}
```

- Return codes do not mix with constructors very well. Constructors have no return type to pass back a status indicator, and passing one back via a reference parameter is messy. Furthermore, the object will still be created and then has to be dealt with or disposed of.

- When an error code is returned to the caller, the caller may not always be equipped to handle the error. If the caller doesn’t want to handle the error, it either has to ignore it (in which case it will be lost forever), or return the error up the stack to the function that called it. This can be messy and lead to many of the same issues noted above.

Exception handling provides a mechanism to decouple handling of errors from the typical control flow of your code. This allows more freedom to handle errors when and how ever is most useful for a given situation, alleviating most (if not all) of the messiness that return codes cause.
Exceptions in C++ are implemented using three keywords that work in conjunction with each other: **throw**, **try**, and **catch**.

---
### Throwing exceptions

A **throw statement** is used to signal that an exception or error case has occurred. This is also commonly called **raising** an exception. To use a throw statement, use the throw keyword, followed by a value of any data type you wish to use to signal that an error has occurred. Typically, this value will be an error code, a description of the problem, or a custom exception class. Here are some examples:

```cpp
throw -1; // throw a literal integer value
throw ENUM_INVALID_INDEX; // throw an enum value
throw "Can not take square root of negative number"; 
// throw a literal C-style (const char*) string
throw dX; // throw a double variable that was previously defined
throw MyException("Fatal Error"); // Throw an object of class MyException
```

---
### Looking for exceptions

We use the **try** keyword to define a block of statements (called a **try block**). The try block acts as an observer, looking for any exceptions that are thrown by any of the statements within the try block. Here’s an example of a try block:

```cpp
try {
    // Statements that may throw exceptions you want to handle go here
    throw -1; // here's a trivial throw statement
}
```

Note that the try block doesn’t define HOW we’re going to handle the exception.

---
### Handling exceptions

The **catch** keyword is used to define a block of code (called a **catch block**) that handles exceptions for a single data type. Here’s an example of a catch block that will catch integer exceptions:
```cpp
catch (int x) {
    std::cerr << "We caught an int exception with value" << x << '\n';
}
```

Try blocks and catch blocks work together, a try block detects any exceptions that are thrown by statements within the try block, and routes them to a catch block with a matching type for handling. A try block must have at least one catch block immediately following it, but may have multiple catch blocks listed in sequence.

Once an exception has been caught by the try block and routed to a matching catch block for handling, the exception is considered handled. After the matching catch block executes, execution then resumes as normal, starting with the first statement after the last catch block.

Catch parameters work just like function parameters, with the parameter being available within the subsequent catch block. Exceptions of fundamental types can be caught by value, but exceptions of non-fundamental types should be caught by const reference to avoid making an unnecessary copy (and, in some cases, to prevent slicing).

Just like with functions, if the parameter is not going to be used in the catch block, the variable name can be omitted. This can help prevent compiler warnings about unused variables:

```cpp
// note: no variable name since we don't use it in the catch block below
catch (double)  {
    std::cerr << "We caught an exception of type double\n";
}
```

**Note that the program will not perform implicit conversions or promotions when matching exceptions with catch blocks**. For example, a char exception will not match with an int catch block. An int exception will not match a float catch block. **However, casts from a derived class to one of its parent classes will be performed.**

If an exception is routed to a catch block, it is considered _handled_ even if the catch block is empty. There are four common things that catch blocks do when they catch an exception:

- Catch blocks may print an error (either to the console, or a log file) and then allow the function to proceed.
- Catch blocks may return a value or error code back to the caller.
- Catch block may throw another exception. Because the catch block is outside of the try block, the newly thrown exception in this case is not handled by the preceding try block. it’s handled by the next enclosing try block.
- A catch block in `main()` may be used to catch fatal errors and terminate the program in a clean way.

---
### Exception handling and stack unwinding

Try blocks catch exceptions not only from statements within the try block, but also from functions that are called within the try block.

Why it’s a good idea to pass errors back to the caller?
The problem is that different applications may want to handle errors in different ways. A console application may want to print a text message. A windows application may want to pop up an error dialog. By passing the error out of the function, each application can handle an error from the function in a way that is the most context appropriate for it. 

Mechanism:

- When an exception is thrown, the program first looks to see if the exception can be handled immediately inside the current function. If it can, it does so.

- If not, the program next checks whether the function’s caller can handle the exception. In order for the function’s caller to handle the exception, the call to the current function must be inside a try block, and a matching catch block must be associated. If no match is found, we go further up.

- This process of checking each function up the call stack continues until either a handler is found, or all of the functions on the call stack have been checked and no handler can be found.

- If a matching exception handler is found, then execution jumps from the point where the exception is thrown to the top of the matching catch block. This requires unwinding the stack (removing the current function from the call stack) as many times as necessary to make the function handling the exception the top function on the call stack.

- If no matching exception handler is found, the stack may or may not be unwound. We will talk more about this further down.

- When the current function is removed from the call stack, all local variables are destroyed as usual, but no value is returned. Unwinding the stack destroys local variables in the functions that are unwound (which is good, because it ensures their destructors execute).

---
### Uncaught exceptions

When no exception handler for a function can be found, `std::terminate()` is called, and the application is terminated. In such cases, the call stack may or may not be unwound. If the stack is not unwound, local variables will not be destroyed, and any cleanup expected upon destruction of said variables will not happen.

There is a good reason for not doing unwinding in such cases. An unhandled exception is generally something you want to avoid at all costs. By not unwinding, we preserve the state of the stack that led to the throwing of the unhandled exception, making it easier to debug.

When an exception is unhandled, the operating system will generally notify you that an unhandled exception error has occurred. How it does this depends on the operating system, but possibilities include printing an error message, popping up an error dialog, or simply crashing. 

#### Catch-All Handler

Adding explicit catch handlers for every possible type is tedious, especially for the ones that are expected to be reached only in exceptional cases. Fortunately, there is a **catch-all handler**. It uses the ellipses operator (…) as the type to catch.

```cpp
int main() {
	try { throw 5; }
	catch (double x) { cout << "Yype double exception: " << x << '\n';}
	catch (...)  { cout << "Undetermined type exception \n"; }
}
```

The catch-all handler must be placed last in the catch block chain. This is to ensure that exceptions can be caught by exception handlers tailored to specific data types if those handlers exist. 

Often, the catch-all handler block is left empty, as this will catch any unanticipated exceptions, ensuring that stack unwinding occurs up to this point and preventing the program from terminating, but does no specific error handling. One use for the catch-all handler is to wrap the contents of `main()`:

```cpp
struct GameSession {};
void runGame(GameSession&) { throw 1;}
void saveGame(GameSession&) {}

int main() {
    GameSession session{};
    try { runGame(session); }
    catch(...) { std::cerr << "Abnormal termination\n"; }
    saveGame(session);
     // save the user's game (even if catch-all handler was hit)
}
```

Many debuggers will (or can be configured to) break on unhandled exceptions, allowing us to view the stack at the point where the unhandled exception was thrown. However, if we have a catch-all handler, then all exceptions are handled, and (because the stack is unwound) we lose useful diagnostic information. Therefore, in debug builds, it can be useful to disable the catch-all handler. We can do this via conditional compilation directives. Here’s one way to do that:

```cpp

struct GameSession {};
void runGame(GameSession&) { throw 1; }
void saveGame(GameSession&) {}

class DummyException { // a dummy class that can't be instantiated
    DummyException() = delete;
};

int main() {
    GameSession session {};
    try { runGame(session); }
#ifndef NDEBUG
    catch(...) { std::cerr << "Abnormal termination\n"; }
#else
    catch(DummyException) {}
#endif
    saveGame(session); 
}
```

Syntactically, a try block requires at least one associated catch block. So if the catch-all handler is conditionally compiled out, we either need to conditionally compile out the try block, or we need to conditionally compile in another catch block. It’s cleaner to do the latter.

For this purpose, we create class `DummyException` which can’t be instantiated because it has a deleted default constructor and no other constructors. Since we can’t create a `DummyException`, this catch handler will never catch anything. Therefore any exceptions that reach this point will not be handled.

---
### Exceptions and RAII

If a constructor must fail for some reason, simply throw an exception to indicate the object failed to create. In such a case, the object’s construction is aborted, and all class members (which have already been created and initialized prior to the body of the constructor executing) are destructed as per usual. However, the class’s destructor is never called (because the object never finished construction). Because the destructor never executes, you can’t rely on said destructor to clean up any resources that have already been allocated.

So how do we ensure the resources that we’ve already allocated get cleaned up properly if an exception occurs after allocation within the constructor? 

One way would be to wrap any code that can fail in a try block, use a corresponding catch block to catch the exception and do any necessary cleanup, and then re-throw the exception. However, this adds a lot of clutter, and it’s easy to get wrong, particularly if your class allocates multiple resources.

There is a better way. If you do the resource allocations inside the members of the class (rather than in the constructor itself), then those members can clean up after themselves when they are destructed, taking advantage of the fact that class members are destructed even if the constructor fails.

Here’s an example:

```cpp
class Member {
public:
	Member() { std::cerr << "Member allocated some resources\n"; }
	~Member() { std::cerr << "Member cleaned up\n"; }
};

class A {
private:
	int m_x {};
	Member m_member;
public:
	A(int x) : m_x{x} { if (x <= 0) throw 1; }
	~A() { std::cerr << "~A\n"; } // should not be called
};

int main() {
	try { A a{0}; }
	catch (int) { std::cerr << "Oops\n"; }
}

// Output:
	// Member allocated some resources
	// Member cleaned up
	// Oops

```

In the above program, when class A throws an exception, all of the members of A are destructed. m_member’s destructor is called, providing an opportunity to clean up any resources that it allocated.

However, creating a custom class like Member to manage a resource allocation isn’t efficient. Fortunately, the C++ standard library comes with RAII-compliant classes to manage common resource types, such as files (`std::fstream`) and dynamic memory (`std::unique_ptr` and other smart pointers). For example, instead of this:

```cpp
class Foo
private:
    int* ptr; // Foo will handle allocation/deallocation
```

Do this:

```cpp
class Foo
private:
    std::unique_ptr<int> ptr; 
    // std::unique_ptr will handle allocation/deallocation
```

In the former case, if Foo’s constructor were to fail after ptr had allocated its dynamic memory, Foo would be responsible for cleanup, which can be challenging. In the latter case, if Foo’s constructor were to fail after ptr has allocated its dynamic memory, ptr’s destructor would execute and return that memory to the system. Foo doesn’t have to do any explicit cleanup when resource handling is delegated to RAII-compliant members.

---
### Exception classes

An **exception class** is just a normal class that is designed specifically to be thrown as an exception. Let’s design an exception class to be used with our IntArray class:

```cpp
class ArrayException {
private:
	std::string m_error;
public:
	ArrayException(std::string_view error) : m_error{ error } {}
	const std::string& getError() const { return m_error; }
};
```

Example:

```cpp
class IntArray {
private:
	int m_data[3]{}; 
public:
	IntArray() {}
	int getLength() const { return 3; }
	int& operator[](const int index) {
		if (index < 0 || index >= getLength())
			throw ArrayException{ "Invalid index" };
		return m_data[index];
	}
};

int main() {
	IntArray array;
	try { int value{ array[5] }; }
	catch (const ArrayException& e) {
		std::cerr << "An arr exception occurred (" << e.getError() << ")\n";
	}
}
```

Using such a class, we can have the exception return a description of the problem that occurred, which provides context for what went wrong. And since ArrayException is its own unique type, we can specifically catch exceptions thrown by the array class and treat them differently from other exceptions if we wish.

Exceptions of a fundamental type can be caught by value since they are cheap to copy.  
Exceptions of a class type should be caught by (const) reference to prevent expensive copying and slicing when dealing with derived exception classes (which we’ll talk about in a moment). 

Catching exceptions by pointer should generally be avoided unless you have a specific reason to do so.

---
### Exceptions and inheritance

Exception handlers will  match classes derived from that specific type as well.

```cpp
class Base {
public: 
	Base() {}
};

class Derived: public Base {
public:
    Derived() {}
};

int main() {
    try { throw Derived(); }
    catch (const Base& base) { std::cerr << "caught Base"; }
    catch (const Derived& derived) { std::cerr << "caught Derived"; }
}

// Ouput:
	// caught Base
```

- Because Derived is derived from Base, Derived is-a Base (they have an is-a relationship). 
- When attempting to find a handler for a raised exception, it is done sequentially.
- In order to make the above work as expected, we need to flip the order of the catch blocks.

---
### `std::exception`

Many of the classes and operators in the standard library throw exception classes on failure. For example: 
- Operator new throw s`std::bad_alloc` if it is unable to allocate enough memory. 
- A failed `dynamic_cast` will throw `std::bad_cast`. 

As of C++20, there are 28 different exception classes that can be thrown, with more being added in each subsequent language standard.

All of these exception classes are derived from `std::exception`. It is a small interface class designed to serve as a base class to any exception thrown by the C++ standard library.

Much of the time, when an exception is thrown by the standard library, we won’t care whether it’s a bad allocation, a bad cast, or something else. Thanks to `std::exception`, we can catch `std::exception` and all of the derived exceptions together in one place.

```cpp
int main() {
    try {
        std::string s;
        s.resize(std::numeric_limits<std::size_t>::max()); 
        // will trigger a std::length_error or allocation exception
    }
    catch (const std::exception& exception) {
        std::cerr << "Standard exception: " << exception.what() << '\n';
    } // Standard exception: string too long
}
```

Note that `std::exception` has a virtual member function `what()` that returns a C-style string description of the exception. Most derived classes override the `what()` function to change the message. This string is meant to be used for descriptive text only, do not use it for comparisons, as it is not guaranteed to be the same across compilers.

Sometimes we’ll want to handle a specific type of exception differently. In this case, we can add a handler for that specific type, and let all the others fall through to the base handler. Consider:

Nothing throws a `std::exception` directly, and neither should you. However, you can throw the other standard exception classes in the standard library. `std::runtime_error` (included as part of the `stdexcept` header) is a popular choice, because it has a generic name, and its constructor takes a customizable message:

```cpp
#include <exception> // for std::exception
#include <iostream>
#include <stdexcept> // for std::runtime_error

int main() {
	try {
		throw std::runtime_error("Bad things happened");
	}
	catch (const std::exception& exception) {
		std::cerr << "Standard exception: " << exception.what() << '\n';
	} // Standard exception: Bad things happened
}
```

#### Deriving your own classes from `std::exception` or `std::runtime_error`

You can derive your own classes from `std::exception`, and override the virtual `what()` const member function. Example:

```cpp
class ArrayException : public std::exception {
private:
	std::string m_error{}; 
public:
	ArrayException(std::string_view error) : m_error{error} {}

	// std::exception::what() returns a const char*, so we must as well
	const char* what() const noexcept override { return m_error.c_str(); }
};
```

Note that virtual function `what()` has specifier noexcept (which means the function promises not to throw exceptions itself). Therefore, our override should also have specifier noexcept.

Because `std::runtime_error` already has string handling capabilities, it’s also a popular base class for derived exception classes. It can take a C-style string parameter, or a `const std::string&` parameter. Example:

```cpp
class ArrayException : public std::runtime_error {
public:
	ArrayException(const std::string& error) : std::runtime_error{ error }  {}
	// no need to override what() since we can just use the parent's what()
};
```

---
### The lifetime of exceptions

When an exception is thrown, the object being thrown is typically a temporary or local variable that has been allocated on the stack. However, the process of exception handling may unwind the function, causing all variables local to the function to be destroyed. So how does the exception object being thrown survive stack unwinding?

When an exception is thrown, the compiler makes a copy of the exception object to some piece of unspecified memory (outside of the call stack) reserved for handling exceptions. That way, the exception object is persisted regardless of whether or how many times the stack is unwound. The exception is guaranteed to exist until the exception has been handled.

If a move constructor is available then a move operation occurs, else it needs to be copied.
In some cases copy elision can also occur. But if both constructors are absent, compilation error will occur. Also we may need the copy constructor if catch blocks catches by value. So the objects being thrown generally need to be copyable (even if the stack is not actually unwound, becuase of compile time check). Example showing what happens when we try to throw a Derived object that is not copyable:

```cpp
class Base {
public:
    Base() {}
};

class Derived : public Base {
public:
    Derived() {}
    Derived(const Derived&) = delete; // not copyable
};

int main() {
    Derived d{};
    try { throw d; } // compile error: Derived copy constructor was deleted
    catch (const Derived& derived) { std::cerr << "caught Derived"; }
    catch (const Base& base) { std::cerr << "caught Base"; }
}
```

Exception objects should not keep pointers or references to stack-allocated objects. If a thrown exception results in stack unwinding (causing the destruction of stack-allocated objects), these pointers or references may be left dangling.

---
### Rethrowing exceptions

There may be cases where you want to catch an exception, but not want to (or have the ability to) fully handle it at the point where you catch it. This is common when you want to log an error, but pass the issue along to the caller to actually handle. 

One obvious solution is to throw a new exception. Although it may seem weird to throw an exception from a catch block, this is allowed. Even having another try-catch block inside a catch block is allowed. Remember, only exceptions thrown within a try block are eligible to be caught. This means that an exception thrown within a catch block will not be caught by the catch block it’s in. Instead, it will be propagated up the stack to the caller.

Another option is to rethrow the same exception. One way to do this is as follows:

```cpp
int getIntValueFromDatabase(Database* d, std::string table, std::string key) {
    assert(d);
    try {
        return d->getIntValue(table, key); // throws int exception on failure
    }
    catch (int exception) {
        g_log.logError("getIntValueFromDatabase failed");
        throw exception;
    }
}
```

Although this works, this method has a couple of downsides. This doesn’t throw the exact same exception as the one that is caught, rather, it throws a copy-initialized copy of variable exception. Although the compiler is free to elide the copy, it may not, so this could be less performant. But significantly, consider what happens in the following case:

```cpp
int getIntValueFromDatabase(Database* d, std::string table, std::string key) {
    assert(d);
    try {
        return d->getIntValue(table, key); // throws Derived exception on failure
    }
    catch (Base& exception)  {
        g_log.logError("getIntValueFromDatabase failed");
        throw exception; // This throws a Base object, not a Derived object
    }
}
```

In this case, `getIntValue()` throws a Derived object, but the catch block is catching a Base reference. This is fine, as we know we can have a Base reference to a Derived object. 
However, when we throw an exception, the thrown exception is copy-initialized from variable exception. Variable exception has type Base, so the copy-initialized exception also has type Base (not Derived). In other words, our Derived object has been sliced.


Fortunately, there a way to rethrow the exact same exception as the one that was just caught. To do so, simply use the `throw` keyword from within the catch block (with no associated variable). This throw keyword rethrows the exact same exception that was just caught. No copies are made, meaning we don’t have to worry about performance killing copies or slicing.

---
### Function try blocks

Try and catch blocks work well enough in most cases, but there is one particular case in which they are not sufficient. Consider the following example:

```cpp
class A {
private: int m_x;
public:
	A(int x) : m_x{x} {
		if (x <= 0) throw 1; // Exception thrown here
	}
};

class B : public A {
public:
	B(int x) : A{x} { // A initialized in member initializer list of B
		// What happens if creation of A fails and we want to handle it here?
	}
};

int main() {
	try { B b{0}; }
	catch (int) { std::cout << "Oops\n"; }
} // Oops
```

The call to base constructor A happens via the member initializer list, before the B constructor’s body is called. In this situation, we have to use a **function try block** They are designed to allow you to establish an exception handler around the body of an entire function, rather than around a block of code:

```cpp
class B : public A {
public:
	B(int x) try : A{x} {}
	catch (...) {
	// Exceptions from member initializer list 
	// or from constructor body are caught here
	    std::cerr << "Exception caught\n";
	    throw; // rethrow the existing exception
	}
};

int main() {
	try { B b{0}; }
	catch (int) { std::cout << "Oops\n"; }
}
// Output:
	// Exception caught
	// Oops
```

Note the addition of the `try` keyword before the member initializer list. This indicates that everything after that point (until the end of the function) should be considered inside of the try block. Also the associated catch block is at the same level of indentation as the entire function. Any exception thrown between the try keyword and the end of the function body will be eligible to be caught here.

Although function level try blocks can be used with non-member functions as well, they typically aren’t. They are almost exclusively used with constructors.

With a regular catch block (inside a function), we have three options:  
- throw a new exception
- rethrow the current exception
- resolve the exception (by either a return statement, or by letting control reach the end of the catch block).

The limitations and behavior of function-level catch blocks for different kinds of function:

|Function type|Can resolve exceptions  <br>via return statement|Behavior at end of catch block|
|---|---|---|
|Constructor|No, must throw or rethrow|Implicit rethrow|
|Destructor|Yes|Implicit rethrow|
|Non-value returning function|Yes|Resolve exception|
|Value-returning function|Yes|Undefined behavior|

You may be tempted to use a function try block as a way to clean up a class that had partially allocated resources before failing. However, referring to members of the failed object is considered undefined behavior since the object is _dead_ before the catch block executes. This means that you can’t use function try to clean up after a class.

Function try is useful primarily for either logging failures before passing the exception up the stack, or for changing the type of exception thrown.

---
### Exception dangers and downsides

1. **Cleaning up resources**

One of the biggest problems that new programmers run into when using exceptions is the issue of cleaning up resources when an exception occurs. Consider the following example:

```cpp
try {
    openFile(filename);
    writeFile(filename, data);
    closeFile(filename);
}
catch (const FileException& exception) {
    std::cerr << "Failed to write to file: " << exception.what() << '\n';
}
```

This example should be rewritten as follows:

```cpp
try {
    openFile(filename);
    writeFile(filename, data);
}
catch (const FileException& exception) {
    std::cerr << "Failed to write to file: " << exception.what() << '\n';
}
closeFile(filename);
```

2. **Exceptions and destructors**

Unlike constructors, exceptions should _never_ be thrown in destructors.
Suppose an exception is thrown out of a destructor during the stack unwinding process. Now the  compiler doesn’t know whether to continue the stack unwinding process or handle the new exception. The end result is that your program will be terminated immediately.
Consequently, the best course of action is to abstain from using exceptions in destructors altogether. Write a message to a log file instead.

3. **Performance concerns**

Exceptions do come with a small performance price to pay. They increase the size of your executable, and they may also cause it to run slower due to the additional checking that has to be performed. However, the main performance penalty for exceptions happens when an exception is actually thrown. In this case, the stack must be unwound and an appropriate exception handler found, which is a relatively expensive operation.

As a note, some modern computer architectures support an exception model called zero-cost exceptions. Zero-cost exceptions have no additional runtime cost in the non-error case (which is the case we most care about performance). However, they incur an even larger penalty in the case where an exception is found.

4. **So when to use exceptions?**

Exception handling is best used when all of the following are true:
- The error being handled is likely to occur only infrequently.
- The error is serious and execution could not continue otherwise.
- The error cannot be handled at the place where it occurs.
- There isn’t a good alternative way to return an error code back to the caller.

As an example, let’s consider the case where you’ve written a function that expects the user to pass in the name of a file on disk. Your function will open this file, read some data, close the file, and pass back some result to the caller. Now, let’s say the user passes in the name of a file that doesn’t exist, or a null string.

In this case, the first three bullets above are trivially met. 
The fourth bullet is the key. It depends on the details of your program. If so (e.g. you can return a null pointer, or a status code to indicate failure), that’s probably the better choice. If not, then an exception would be reasonable.

---

