Initialisation types:
1. copy: eg int a=5;
2. direct: eg int a(5);
3. list: 
	1. direct-list: eg int a{5};(most preferred)
	2. copy-list: eg int a={5};

| Initialization Type        | Example        | Note                                                    |
| -------------------------- | -------------- | ------------------------------------------------------- |
| Default-initialization     | int x;         | In most cases, leaves variable with indeterminate value |
| Copy-initialization        | int x = 5;     |                                                         |
| Direct-initialization      | int x ( 5 );   |                                                         |
| Direct-list-initialization | int x { 5 };   | Narrowing conversions disallowed                        |
| Copy-list-initialization   | int x = { 5 }; | Narrowing conversions disallowed                        |
| Value-initialization       | int x {};      | Usually performs zero-initialization                    |


Intricacies:
1. assigning 4.5 to int works in the first two(assigns 4) but gives error in list
2. when assigning with empty {}, In most cases, value-initialization will implicitly initialize the variable to zero (or whatever value is closest to zero for a given type). In cases where zeroing occurs, this is called **zero-initialization**.
3. For class types, value-initialization (and default-initialization) may instead initialize the object to predefined default values, which may be non-zero.
---
Instantiation: The term **instantiation** is a fancy word that means a variable has been created (allocated) and initialized (this includes default initialization). An instantiated object is sometimes called an **instance**. Most often, this term is applied to class type objects, but it is occasionally applied to objects of other types as well.

---
[[maybe_unused]]: C++17 introduced the `[[maybe_unused]]` attribute, which allows us to tell the compiler that we’re okay with a variable being unused. The compiler will not generate unused variable warnings for such variables.

---
Default-initialization is when a variable initialization has no initializer (e.g. `int x;`). In most cases, the variable is left with an indeterminate value.
Value-initialization is when a variable initialization has an empty brace initializer (e.g. `int x{};`). In most cases this will perform zero-initialization.
You should prefer value-initialization, as it initializes the variable to a consistent value.

---
## Input-output
- Best practice: Output a newline whenever a line of output is complete. Reason: after running an executable from the command line, some operating systems do not output a new line before showing the command prompt again. If our program does not end with the cursor on a new line, the command prompt may appear appended to the prior line of output, rather than at the start of a new line as the user would expect.
### `std::cout` is buffered
- Statements in our program request that output be sent to the console. However, that output is typically not sent to the console immediately. Instead, the requested output “gets in line”, and is stored in a region of memory set aside to collect such requests (called a **buffer**). Periodically, the buffer is **flushed**, meaning all of the data collected in the buffer is transferred to its destination (in this case, the console).
- Here are the only times `std::cout` flushes naturally without `std::endl`:
	- **The buffer reaches maximum capacity:** The OS decides the buffer is full (usually after several kilobytes of data) and pushes it out to make room.
	- **The program terminates:** When your code finishes successfully, the OS cleans up and flushes all remaining buffers.
	- **You read from `std::cin`:** By default, `std::cin` and `std::cout` are "tied" together. If your program stops to wait for input, C++ assumes you probably printed a prompt for the user, so it politely flushes the output buffer so the user can read the prompt before typing.
	- There is no timer to flush in cpp, this brings us to why we use endl in interactive problems in cp
#### Why use endl in cp?
You might be thinking: _Wait, if `std::cin` automatically flushes `std::cout`, shouldn't interactive problems work fine since I always use `cin` to read the judge's response?_

They would, except for one massive catch: **`cin.tie(NULL)`**.
In competitive programming, you almost certainly include `cin.tie(NULL)` (along with `ios_base::sync_with_stdio(false)`) at the very start of your `main` function to make your I/O operations drastically faster.

Doing this intentionally severs the automatic link between `cin` and `cout`. Once you sever that link, you create a classic software **deadlock**:
1. Your program prints a query (it gets trapped in the buffer).
2. Your program calls `cin` to wait for the judge's answer and pauses.
3. The judge is waiting for your query, but because it's stuck in the buffer, the judge sees nothing and pauses.
4. Neither program has a timer, and the buffer isn't full yet.
### **1. The Mechanics**
The `.tie()` method does two things:
- If called with no arguments, it returns a pointer to the output stream it is currently tied to.
- If called with an argument (a pointer to an output stream), it ties the input stream to that new output stream.
By passing `NULL` (or `nullptr`), you are explicitly telling `cin` to tie itself to **nothing**. You are severing the link.
### **2. The Result**
Once the link is severed, `cin` no longer cares what `cout` is doing. When your program hits a `cin >> variable;` statement, it will just wait for input immediately without checking or flushing the `cout` buffer.
#### How does using cin.tie(NULL) increase i/o speed? 
If `cin` and `cout` are tied, every single time you read a number, the program stops to flush the output buffer. Flushing is a computationally expensive hardware operation. Doing it 100,000 times will bring your execution time to a crawl.

By using `cin.tie(NULL)`, you allow `cout` to act like a true buffer. It just quietly collects your 100,000 answers in memory and flushes them in massive, efficient batches (or when the program finishes), saving you a massive amount of execution time.

#### What does ios_base::sync_with_stdio(false) do?
The Default State: Playing Nice
- Because C++ is built on top of C, you can freely mix C++ I/O (`std::cin`, `std::cout`) with C I/O (`scanf`, `printf`) in the same program.
- To ensure that your output appears in the exact order you wrote it—even if you switch back and forth between `cout` and `printf`—the C++ standard library forces C++ streams to **synchronize** with the underlying C streams.
- Essentially, `std::cout` and `printf` share the exact same underlying buffer. They constantly coordinate to make sure they aren't stepping on each other's toes.
#### **What `sync_with_stdio(false)` Does**
This synchronization overhead is incredibly slow. By setting this flag to `false`, you are telling the C++ compiler:
> _"I promise I am only going to use C++ I/O streams. Stop coordinating with the C streams."_
Once you do this:
1. C++ streams are freed from the C standard library.
2. `cin` and `cout` are allowed to use their own independent, highly optimized internal buffers.
3. Your read and write speeds instantly become massively faster—often matching or even beating `scanf` and `printf`.
4. So after this just make sure not to use cout and printf together, cin and scanf together, cause they no longer share the same buffer and the output might no be what you intended
#### I think it should be pretty clear by now as to why endl is slower than '\n'
---
- cin >> x >> y can input multiple variables on a single line, just make sure values entered should be separated by whitespace (spaces, tabs, or newlines).
- The C++ I/O library does not provide a way to accept keyboard input without the user having to press _enter_. If this is something you desire, you’ll have to use a third party library.
### `std::cin` is buffered
Inputting data is also a two stage process:

- The individual characters you enter as input are added to the end of an input buffer (inside `std::cin`). The enter key (pressed to submit the data) is also stored as a `'\n'` character.
- The extraction operator ‘>>’ removes characters from the front of the input buffer and converts them into a value that is assigned (via copy-assignment) to the associated variable. This variable can then be used in subsequent statements.
- `std::cin` is buffered because it allows us to separate the entering of input from the extract of input. We can enter input once and then perform multiple extraction requests on it.
### How cin actually works
### **1. The Safety Check (Is the machine broken?)**
Before doing anything, the worker checks if the conveyor belt has a giant red "OUT OF ORDER" sign on it. If a previous operation failed (we'll explain how that happens in a second) and you haven't manually cleared the error, `std::cin` refuses to do any work. It instantly aborts and moves on to the next line of your code.
### **2. Taking out the Trash (Discarding leading whitespace)**
If the machine is working, the worker looks at the conveyor belt. Their first job is to throw away all the "junk" at the very front. In C++, "junk" is whitespace: spaces, tabs, and "Enter" key presses (newlines) from previous inputs. They will keep throwing away spaces until they finally see a real character. _(This is exactly why your `5 6` example worked perfectly earlier!)_
### **3. Waiting for Parts (Pausing the program)**
What if the worker throws away all the junk spaces, and now the conveyor belt is completely empty? They pause the entire factory. Your program freezes, waiting for the user to type something on the keyboard and press Enter to send more items down the belt.
### **4. Grabbing the Goods (The Extraction)**
Once there are actual non-whitespace characters in front of them, the worker starts grabbing them and putting them together. But they are extremely strict about when to stop. They will stop grabbing characters the exact millisecond they see:
- A whitespace character (space, tab, newline).
- A character that doesn't fit the box. For example, if your variable is an `int`, and the worker sees the letter `A` or a decimal point `.`, they stop immediately.
### **The Three Possible Outcomes**
After going through that checklist, one of three things happens:
- **Outcome A (Aborted early):** If the machine was already broken in Step 1, absolutely nothing happens. The variable stays exactly as it was.
- **Outcome B (Success!):** The worker successfully grabbed valid characters in Step 4. They convert those characters into the right format (like turning the characters `'4'` and `'2'` into the actual integer `42`) and put that value into your variable.
- **Outcome C (Total Failure):** This is the dangerous one. Let's say you asked for an `int`, but the user typed `"Hello"`. The worker looks at the `'H'`, realizes it's completely invalid for an `int`, and cannot extract _any_ characters.
    - C++ punishes this severely. It forces the value of your variable to become `0`.
    - Worse, the worker hits the giant red emergency stop button. `std::cin` enters a "fail state" (a broken machine).
    - The `"Hello"` is left stuck on the conveyor belt, and **every single `cin` statement in your program from that point forward will be ignored** until you write specific code to clear the error state and flush the bad input.
    - We'll discuss later how to handle this error
eg: 
```cpp
int x{};
std::cin >> x;
```

Here’s what happens in a three different input cases:

- If the user types `5a` and enter, `5a\n` will be added to the buffer. `5` will be extracted, converted to an integer, and assigned to variable `x`. `a\n` will be left in the input buffer for the next extraction.
- If the user types ‘b’ and enter, `b\n` would be added to the buffer. Because `b` is not a valid integer, no characters can be extracted, so this is an extraction failure. Variable `x` would be set to `0`, and future extractions will fail until the input stream is cleared.
- If `std::cin` is not in a good state due to a prior failed extraction, nothing happens here. The value of variable `x` is not altered.
Implementation-defined behavior and unspecified behavior

A specific compiler and the associated standard library it comes with are called an **implementation** (as these are what actually implements the C++ language). In some cases, the C++ language standard allows the implementation to determine how some aspect of the language will behave, so that the compiler can choose a behavior that is efficient for a given platform. Behavior that is defined by the implementation is called **implementation-defined behavior**. Implementation-defined behavior must be documented and consistent for a given implementation.

Let’s look at a simple example of implementation-defined behavior:

```cpp
#include <iostream>

int main()
{
	std::cout << sizeof(int) << '\n'; // print how many bytes of memory an int value takes

	return 0;
}
```

On most platforms, this will produce `4`, but on others it may produce `2`.
**Unspecified behavior** is almost identical to implementation-defined behavior in that the behavior is left up to the implementation to define, but the implementation is not required to document the behavior.
### Keywords
Here is a list of all the C++ keywords (through C++23):

- alignas
- alignof
- and
- and_eq
- asm
- auto
- bitand
- bitor
- bool
- break
- case
- catch
- char
- char8_t (since C++20)
- char16_t
- char32_t
- class
- compl
- concept (since C++20)
- const
- consteval (since C++20)
- constexpr
- constinit (since C++20)
- const_cast
- continue
- co_await (since C++20)
- co_return (since C++20)
- co_yield (since C++20)
- decltype
- default
- delete
- do
- double
- dynamic_cast
- else
- enum
- explicit
- export
- extern
- false
- float
- for
- friend
- goto
- if
- inline
- int
- long
- mutable
- namespace
- new
- noexcept
- not
- not_eq
- nullptr
- operator
- or
- or_eq
- private
- protected
- public
- register
- reinterpret_cast
- requires (since C++20)
- return
- short
- signed
- sizeof
- static
- static_assert
- static_cast
- struct
- switch
- template
- this
- thread_local
- throw
- true
- try
- typedef
- typeid
- typename
- union
- unsigned
- using
- virtual
- void
- volatile
- wchar_t
- while
- xor
- xor_eq
C++ also defines special identifiers: _override_, _final_, _import_, and _module_. These have a specific meaning when used in certain contexts but are not reserved otherwise.
Along with a set of operators, these keywords and special identifiers define the entire language of C++ (preprocessor commands excluded).

### Identifier naming rules [](https://www.learncpp.com/cpp-tutorial/keywords-and-naming-identifiers/#rules)

As a reminder, the name of a variable (or function, type, or other kind of item) is called an identifier. C++ gives you a lot of flexibility to name identifiers as you wish. However, there are a few rules that must be followed when naming identifiers:

- The identifier can not be a keyword. Keywords are reserved.
- The identifier can only be composed of letters (lower or upper case), numbers, and the underscore character. That means the name can not contain symbols (except the underscore) nor whitespace (spaces or tabs).
- The identifier must begin with a letter (lower or upper case) or an underscore. It can not start with a number.
- C++ is case sensitive, and thus distinguishes between lower and upper case letters. `nvalue` is different than `nValue` is different than `NVALUE`.
#### Some language elements must be whitespace-separated
When whitespace is required as a separator, the compiler doesn’t care how much whitespace is used, as long as some exists.

The following variable definitions are all valid:

```cpp
int x;
int                y;
            int
z;
```

In certain cases, newlines are used as a separator. Single-line comments are terminated by a newline.
Preprocessor directives (e.g. `#include <iostream>`) must be placed on separate lines:

```cpp
#include <iostream>
#include <string>
```
Newlines are not allowed in quoted text:
```cpp
std::cout << "Hello
     world!"; // Not allowed!
```

Quoted text separated by nothing but whitespace (spaces, tabs, or newlines) will be concatenated:

```cpp
std::cout << "Hello "
     "world!"; // prints "Hello world!"
```

Unlike some other languages, C++ does not enforce any kind of formatting restrictions on the programmer. For this reason, we say that C++ is a whitespace-independent language.

### Literals
Consider the following two statements:
```cpp
std::cout << "Hello world!";
int x { 5 };
```
What are ‘”Hello world!”‘ and ‘5’? They are literals. A **literal** (also known as a **literal constant**) is a fixed value that has been inserted directly into the source code.

### Operators
The number of operands that an operator takes as input is called the operator’s **arity**. Few people know what this word means, so don’t drop it in a conversation and expect anybody to have any idea what you’re talking about. Operators in C++ come in four different arities:

**Unary** operators act on one operand. An example of a unary operator is the `-` operator. For example, given `-5`, `operator-` takes literal operand `5` and flips its sign to produce new output value `-5`.

**Binary** operators act on two operands (often called _left_ and _right_, as the left operand appears on the left side of the operator, and the right operand appears on the right side of the operator). An example of a binary operator is the `+` operator. For example, given `3 + 4`, `operator+` takes the left operand `3` and the right operand `4` and applies mathematical addition to produce new output value `7`. The insertion (`<<`) and extraction (`>>`) operators are binary operators, taking `std::cout` or `std::cin` on the left side, and the value to output or variable to input to on the right side.

**Ternary** operators act on three operands. There is only one of these in C++ (the conditional operator), which we’ll cover later.

**Nullary** operators act on zero operands. There is also only one of these in C++ (the throw operator), which we’ll also cover later.

### Nomenclature-Side Effect

In common language, the term “side effect” is typically used to mean a secondary (often negative or unexpected) result of some other thing happening (such as taking medicine). For example, a common side effect of taking oral antibiotics is diarrhea. As such, we often think of side effects as things we want to avoid, or things that are incidental to the primary goal.

In C++, the term “side effect” has a different meaning: it is an observable effect of an operator or function beyond producing a return value.

Since assignment has the observable effect of changing the value of an object, this is considered a side effect. We use certain operators (e.g. the assignment operator) primarily for their side effects (rather than the return value those operators produce). In such cases, the side effect is both beneficial and predictable (and it is the return value that is often incidental).

For the operators we call primarily for their side effects (e.g. `operator=` or `operator<<`), it’s not always obvious what return values they produce (if any). For example, what return value would you expect `x = 5` to have?

Both `operator=` and `operator<<` (when used to output values to the console) return their left operand. Thus, `x = 5` returns `x`, and `std::cout << 5` returns `std::cout`. This is done so that these operators can be chained.

For example, `x = y = 5` evaluates as `x = (y = 5)`. First `y = 5` assigns `5` to `y`. This operation then returns `y`, which can then be assigned to `x`.

`std::cout << "Hello " << "world!"` evaluates as `(std::cout << "Hello ") << "world!"`. This first prints `"Hello "` to the console. This operation returns `std::cout`, which can then be used to print `"world!"` to the console as well.

### Expressions
In C++, the result of an expression is one of the following:
- A value (most commonly)
- An object or a function. We discuss expressions that return objects in lesson [12.2 -- Value categories (lvalues and rvalues)](https://www.learncpp.com/cpp-tutorial/value-categories-lvalues-and-rvalues/).
- Nothing. These are the result of non-value returning function calls (covered in lesson [2.3 -- Void functions (non-value returning functions)](https://www.learncpp.com/cpp-tutorial/void-functions-non-value-returning-functions/)) that are called only for their side effects
For now, to keep things simple, we’ll assume expressions are evaluated to produce values.

Expressions involving operators with side effects are a little more tricky:
```cpp
x = 5           // x = 5 has side effect of assigning 5 to x, evaluates to x
x = 2 + 3       // has side effect of assigning 5 to x, evaluates to x
std::cout << x  // has side effect of printing value of x to console, evaluates to std::cout
```
- Expressions do not end in a semicolon, and cannot be compiled by themselves. For example, if you were to try compiling the expression `x = 5`, your compiler would complain (probably about a missing semicolon). Rather, expressions are always evaluated as part of statements.
- An **expression statement** is a statement that consists of an expression followed by a semicolon. When the expression statement is executed, the expression will be evaluated.
- When an expression is used in an expression statement, any result generated by the expression is discarded (because it is not used).
- We can also make expression statements that compile but have no effect. For example, the expression statement (`2 * 3;`) is an expression statement whose expression evaluates to the result value of _6_, which is then discarded. While syntactically valid, such expression statements are useless. Some compilers (such as gcc and Clang) will produce warnings if they can detect that an expression statement is useless.
### Subexpressions, full expressions, and compound expressions

We occasionally need to talk about specific kinds of expressions. For this purpose, we will define some related terms.
Consider the following expressions:
```cpp
2               // 2 is a literal that evaluates to value 2
2 + 3           // 2 + 3 uses operator+ to evaluate to value 5
x = 4 + 5       // 4 + 5 evaluates to value 9, which is then assigned to variable x
```

- Simplifying a bit, a **subexpression** is an expression used as an operand. For example, the subexpressions of `x = 4 + 5` are `x` and `4 + 5`. The subexpressions of `4 + 5` are `4` and `5`.
- A **full expression** is an expression that is not a subexpression. All three expressions above (`2`, `2 + 3`, and `x = 4 + 5`) are full expressions.
- In casual language, a **compound expression** is an expression that contains two or more uses of operators. `x = 4 + 5` is a compound expression because it contains two uses of operators (`operator=` and `operator+`). `2` and `2 + 3` are not compound expressions.
