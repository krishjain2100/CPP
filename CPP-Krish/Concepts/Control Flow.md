When a program is run, the CPU begins execution at the top of `main()`, executes some number of statements (in sequential order by default), and then the program terminates at the end of `main()`. The specific sequence of statements that the CPU executes is called the program’s **execution path** (or **path**, for short). C++ provides a number of different **control flow statements** which allow the programmer to change the normal path of execution through the program.

## Categories of flow control statements

|Category|Meaning|Implemented in C++ by|
|---|---|---|
|Conditional statements|Causes a sequence of code to execute only if some condition is met.|if, else, switch|
|Jumps|Tells the CPU to start executing the statements at some other location.|goto, break, continue|
|Function calls|Jump to some other location and back.|function calls, return|
|Loops|Repeatedly execute some sequence of code zero or more times, until some condition is met.|while, do-while, for, ranged-for|
|Halts|Terminate the program.|std::exit(), std::abort()|
|Exceptions|A special kind of flow control structure designed for error handling.|try, throw, catch|

---
### If Statement

A **conditional statement** is a statement that specifies whether some associated statement(s) should be executed or not. C++ supports two basic kinds of conditionals: `if statements` and `switch statements`

#### Implicit blocks

If the programmer does not declare a block in the statement portion of an `if statement` or `else statement`, the compiler will implicitly declare one. Thus:

```
if (condition)
    true_statement;
else
    false_statement;
```

is actually the equivalent of:

```
if (condition) {
    true_statement;
}
else {
    false_statement;
}
```

`if consteval` (added in C++23) requires the use of blocks. Thus using blocks ensures consistency between `if` and `if consteval`.

#### Common if statement problems

1. Nested if without brackets

```cpp
if (x >= 0) // outer if-statement
        if (x <= 20) // inner if-statement
            std::cout << x << " is between 0 and 20\n";
    // which if statement does this else belong to?
    else
        std::cout << x << " is negative\n";
```

This is a **dangling else** problem. An else-statement is paired up with the last unmatched if-statement in the same block.

```cpp
if (x >= 0) {
	if (x <= 20) // inner if statement
		std::cout << x << " is between 0 and 20\n";
	else // attached to inner if statement
		std::cout << x << " is negative\n";
}
```

2. `==` instead of `=`: `if (x=0)` evaluates to x and then to 0 and then to false, so write `x == 0`.

#### Null statements

A **null statement** is an expression statement that consists of just a semicolon:

```cpp
if (x > 10)
    ; // this is a null statement
```

In Python, the `pass` keyword serves as a null statement. In C++, we can mimic `pass` by using the preprocessor:

```cpp
#define PASS
void foo(int x, int y){
    if (x > y) PASS;
    else PASS;
}
```

#### Constexpr If Statement

C++17 introduces the **constexpr if statement**, which requires the conditional to be a constant expression. The conditional of a constexpr-if-statement will be evaluated at compile-time.
If the constexpr conditional evaluates to `true`, the entire if-else will be replaced by the true-statement. If the constexpr conditional evaluates to `false`, the entire if-else will be replaced by the false-statement (if it exists) or nothing (if there is no else).

For optimization purposes, modern compilers will generally treat non-constexpr if-statements that have constexpr conditionals as if they were constexpr-if-statements. However, they are not required to do so. A compiler that encounters a non-constexpr if-statement with a constexpr conditional may issue a warning advising you to use `if constexpr` instead. This will ensure that compile-time evaluation will occur (even if optimizations are disabled).

---
### Switch Statement

C++ provides an alternative conditional statement called a switch-statement that is specialized to avoid usage of many chained if-else statements.

```cpp
void printDigitName(int x){
	switch (x) {
	case 1:
		std::cout << "One";
		return;// these returns are necessary 
	case 2:
		std::cout << "Two";
		return;
	default:
		std::cout << "Unknown";
		return;
	}
}
```

When an expression is evaluated to produce a value, then one of the following occurs:
- If the expression’s value is equal to the value after any of the case-labels, the statements after the matching case-label are executed.
- If no matching value can be found and a default label exists, the statements after the default label are executed.
- If no matching value can be found and there is no default label, the switch is skipped.

#### Switch condition

Often the expression is just a single variable, but it can be any valid expression.
The condition in a switch **must evaluate to an integral type or an enumerated type or be convertible to one**. Expressions that evaluate to floating point types, strings, and most other non-integral types may not be used here.

This because switch statements are designed to be highly optimized.The most common way for compilers to implement switch statements is via [Jump tables](https://en.wikipedia.org/wiki/Branch_table),  and jump tables only work with integral values. The following pseudocode illustrates the concept:

```
 ... validate x /*transform x to 0 (invalid) or 1,2,3,according to value..)*/
       y = x * 4;    /* multiply by branch instruction length (e.g. 4 )*/
       goto next + y;/* branch into 'table' of branch instructions*/
 /* start of branch table */
 next: goto codebad; /* x= 0  (invalid)*/
       goto codeone;/* x= 1*/
       goto codetwo;/* x= 2*/
 ... rest of branch table
 codebad:/* deal with invalid input*/
```

Of course, compilers don’t have to implement switches using jump tables, and sometimes they don’t. There is technically no reason that C++ couldn’t relax the restriction so that other types could be used as well, they just haven’t done so yet (as of C++23).

#### Case labels

**Case label** declared using the `case` keyword and followed by a constant expression. The constant expression must either match the type of the condition or must be convertible to that type.

If the value of the conditional expression equals the expression after a `case label`, execution begins at the first statement after that `case label` and then continues sequentially. Everything after that runs. This is the reason for the returns in the last code otherwise the full code starting from the case's line is executed. 

There is no practical limit to the number of case labels you can have, but all case labels in a switch must be unique.

#### The default label

The **default label**  is declared using the `default` keyword. If the conditional expression does not match any case label and a default label exists, execution begins at the first statement after the default label. The default label is optional, and there can only be one default label per switch statement. By convention, the default case is placed last in the switch block. If the value of the conditional expression does not match any of the case labels, and no default case has been provided, then no cases inside the switch are executed. Execution continues after the end of the switch block.

#### Break

In the above examples, we used return-statements to stop execution of the statements after our labels. However, this also exits the entire function.

A **break statement** (declared using the `break` keyword) tells the compiler that we are done executing statements within the switch, and that execution should continue with the statement after the end of the switch block. This allows us to exit a switch-statement without exiting the entire function.

If you don’t end a set of statements under a label with a `break` or `return` then it is called fall-through. Many compilers and code analysis tools will flag fall-through as a warning.

**Attributes** are a modern C++ feature that allows the programmer to provide the compiler with some additional data about the code. To specify an attribute, the attribute name is placed between double brackets. Attributes are not statements, rather, they can be used almost anywhere where they are contextually relevant. The `[[fallthrough]]` attribute modifies a `null statement` to indicate that fall-through is intentional and no warnings should be triggered.

```cpp
case 2:
	std::cout << 2 << '\n'; // Execution begins here
	[[fallthrough]]; 
	// intentional fallthrough the semicolon is to indicate the null statement
case 3:
	std::cout << 3 << '\n'; // This is also executed
	break;
```

#### Sequential case labels

```cpp
switch (c) {
    case 'a': 
    case 'e': 
    case 'i': 
    case 'o': 
    case 'u': 
    case 'A':
    case 'E':
    case 'I': 
    case 'O':
    case 'U': 
        return true;
    default:
        return false;
    }
```

Remember, execution begins at the first statement after a matching case label. **Case labels aren’t statements (they’re labels), so they don’t count.** The first statement after _all_ of the case statements in the above program is `return true`, so if any case labels match, the function will return `true`. Thus, we can stack case labels to make all of those case labels share the same set of statements afterward. This is not considered fall-through behavior, so use of comments or `[[fallthrough]]` is not needed here.

#### Labels do not define a new scope

With `if statements`, you can only have a single statement after the if-condition, and that statement is considered to be implicitly inside a block. However, with switch statements, the statements after labels are all scoped to the switch block. No implicit blocks are created.

#### Variable declaration and initialization inside case statements

You can declare or define (but not initialize) variables inside the switch, both before and after the case labels:

```cpp
switch (1){
    int a; // okay: definition is allowed before the case labels
    int b{ 5 }; // illegal: initialization is not allowed before the case labels
    a = 100; // legal: but will never run
case 1:
    int y; // okay but bad practice: definition is allowed within a case
    y = 4; // okay: assignment is allowed
    break;
case 2:
    int z{ 4 };//illegal:initialization is not allowed if subsequent cases exist
    y = 5; // okay: y was declared above, so we can use it here too
    break;
case 3:
    break;
}
```

Although variable `y` was defined in `case 1`, it was used in `case 2` as well. All statements inside the switch are considered to be part of the same scope. Thus, a variable declared or defined in one case can be used in a later case, even if the case in which the variable is defined is never executed because the switch jumped over it.

Initialization of variables is disallowed in any case that is not the last case (because the switch could jump over the initializer if there is a subsequent case defined, in which case the variable would be undefined, and accessing it would result in undefined behavior). Initialization is also disallowed before the first case, as those statements will never be executed, as there is no way for the switch to reach them.

Declaration is fine for trivial types like `int`, `float`, `char`, or pointers.
If you try to declare a complex object, C++ will actually block the declaration too, because they have constructors and we can't skip calling constructors because at the end of scope compiler will insert destructor call at compile time itself. So if destructor deallocates memory that constructor had to allocate but didn't then it will occur in a crash.

If a case needs to define and/or initialize a new variable, the best practice is to do so inside an explicit block underneath the case statement

Running case 2 above would also work fine and it would consider y to be declared even though the declaration statement never ran(it will obviously have garbage value until `y = 5` is run)

---
### Goto Statement

 An unconditional jump causes execution to jump to another spot in the code.
In C++, unconditional jumps are implemented via a **goto statement**, and the spot to jump to is identified through use of a **statement label**. Just like with switch case labels, statement labels are conventionally not indented. eg:

```cpp
#include <iostream>
#include <cmath> // for sqrt() function
int main() {
    double x{};
tryAgain: // this is a statement label
    std::cout << "Enter a non-negative number: ";
    std::cin >> x;

    if (x < 0.0)
        goto tryAgain; // this is the goto statement

    std::cout << "The square root of " << x << " is " << std::sqrt(x) << '\n';
    return 0;
}
```

#### Statement labels have function scope

- We covered two kinds of scope: local (block) scope, and file (global) scope. Statement labels utilize a third kind of scope: **function scope**, which means the label is visible throughout the function even before its point of declaration. The goto statement and its corresponding `statement label` must appear in the same function.
- Goto statements can also jump forward

```cpp
void printCats(bool skip) {
    if (skip) goto end; 
    // jump forward;  'end' is visible here due to it having function scope
    std::cout << "cats\n";
end:
    ; // statement labels must be associated with a statement
}
```

- Note that statement labels must be associated with a statement (hence their name: they label a statement). Because the end of the function had no statement, we had to use a null statement so we had a statement to label.
- There are two primary limitations to jumping: 
	1. You can jump only within the bounds of a single function (you can’t jump out of one function and into another)
	2. And if you jump forward, you can’t jump forward over the initialization of any variable that is still in scope at the location being jumped to. Note that you can jump backwards over a variable initialization, and the variable will be re-initialized when the initialization is executed.

#### Avoid using goto

Use of `goto` is shunned in C++ (and other modern high level languages as well). The primary problem with goto is that it allows a programmer to jump around the code arbitrarily. This creates what is known as spaghetti code  making it extremely difficult to follow the logic of such code. Almost any code written using a goto statement can be more clearly written using other constructs in C++, such as if-statements and loops. 

One notable exception is when you need to exit a nested loop but not the entire function, in such a case, a goto to just beyond the loops is probably the cleanest solution.

Avoid goto statements (unless the alternatives are significantly worse for code readability).

---
### While Loop 

Do while statements

```
do
    statement; 
while (condition);
```

Favor while loops over do-while when given an equal choice.

---
### For Loop

As of C++11, there are two different kinds of for-loops. 
- Classic for-statement in this lesson,
- Range-based for-statement (covered later).


```
for (init-statement; condition; end-expression)
   statement;
```

The init-statement is executed first. This only happens once when the loop is initiated. These variables have “loop scope”, which is just a form of block scope where these variables exist from the point of definition through the end of the loop statement.

For-loops typically iterate over only one variable, but sometimes for-loops need to work with multiple variables. To assist with this, the programmer can define multiple variables in the init-statement, and can make use of the comma operator to change the value of multiple variables in the end-expression

We can't directly define variables of multiple types in the for init statement, but there are ways we will discuss later.

---
### Break

The **break statement** causes a while loop, do-while loop, for loop, or switch statement to end, with execution continuing with the next statement after the loop or switch being broken out of.

---
#### Continue

The **continue statement** provides a convenient way to end the current iteration of a loop without terminating the entire loop.

Many textbooks caution readers not to use `break` and `continue` in loops, both because it causes the execution flow to jump around, and because it can make the flow of logic harder to follow. However, used judiciously, `break` and `continue` can help make loops more readable by keeping the number of nested blocks down and reducing the need for complicated looping logic.

There’s a similar argument to be made for return statements. A return statement that is not the last statement in a function is called an **early return**. Many programmers believe early returns should be avoided. A function that only has one return statement at the bottom of the function has a simplicity to it, you can assume the function will take its arguments, do whatever logic it has implemented, and return a result without deviation. Having extra returns complicates the logic. 

The counter-argument is that using early returns allows your function to exit as soon as it is done, which reduces having to read through unnecessary logic and minimizes the need for conditional nested blocks, which makes your code more readable.

---
### Halts

A **halt** is a flow control statement that terminates the program. In C++, halts are implemented as functions (rather than keywords), so our halt statements will be function calls.

#### `std::exit()`

`std::exit()` is a function that causes the program to terminate normally. 
**Normal termination** means the program has exited in an expected way. Note that the term normal termination does not imply anything about whether the program was successful (that’s what the `status code` is for). For example, let’s say you were writing a program where you expected the user to type in a filename to process. If the user typed in an invalid filename, your program would probably return a non-zero `status code` to indicate the failure state, but it would still have a normal termination.

`std::exit()` is called implicitly after function `main()` returns.
`std::exit()` can also be called explicitly from any function to halt the program. 
It is in the `<cstdlib>` header.

`std::exit()` performs a number of cleanup functions. 
First, objects with static storage duration are destroyed. 
Then some other miscellaneous file cleanup is done if any files were used. 
Finally, control is returned back to the OS, with the argument passed to `std::exit()` used as the status code.

`std::exit()` does not clean up any local variables (either in the current function, or in functions up the call stack). This means calling `std::exit()` can be dangerous if your program relies on any local variables cleaning themselves up.

#### `std::atexit`

Because `std::exit()` terminates the program immediately, you may want to manually do some cleanup before terminating. In this context, cleanup means things like closing database or network connections, deallocating any memory you have allocated, writing information to a log file, etc.

C++ offers the `std::atexit()` function, which allows you to pass a callback function which will automatically be called on program termination via `std::exit()`.

```cpp
void cleanup(){  std::cout << "cleanup!\n";}

int main(){
    std::atexit(cleanup);     
    std::cout << 1 << '\n';
    std::exit(0)
    // The following statements never execute
    std::cout << 2 << '\n';
    return 0;
}
```

- Because `std::exit()` is called implicitly when `main()` terminates, this will invoke any functions registered by `std::atexit()` if the program exits that way. 
- The function being registered must take no parameters and have no return value. 
- You can register multiple cleanup functions using `std::atexit()` if you want, and they will be called in reverse order of registration (the last one registered will be called first).

In multi-threaded programs, calling `std::exit()` can cause your program to crash (because the thread calling `std::exit()` will cleanup static objects that may still be accessed by other threads). 

For this reason, C++ has introduced another pair of functions that work similarly to `std::exit()` and `std::atexit()` called `std::quick_exit()` and `std::at_quick_exit()`. 

`std::quick_exit()` terminates the program normally, but does not clean up static objects, and may or may not do other types of cleanup. `std::at_quick_exit()` performs the same role as `std::atexit()` for programs terminated with `std::quick_exit()`.


When an application exits, modern OS will generally clean up any memory that the application does not properly clean up itself. So why bother doing cleanup on exit? There are atleast two reasons:
1. Cleaning up in some cases and not others is inconsistent and can lead to errors. Not cleaning up memory properly can also impact the way certain tools like memory profilers behave (they may be unable to distinguish memory that you inadvertently cleaning up from memory that you intentionally aren’t cleaning up because you don’t have to).
2. There are other kinds of cleanup that may be necessary for your program to behave predictably. For example, if you write data to a file and then unexpectedly exit, that data may not have been flushed to the file yet, and may be lost when the program exits. Closing the file before shutting down helps ensure that all cached data will be written first. Or you may want to send data about the user’s session, or why the program is shutting down to server before the shutdown actually happens.

#### `std::abort` and `std::terminate`

The `std::abort()` function causes your program to terminate abnormally. **Abnormal termination** means the program had some kind of unusual runtime error (like dividing by zero) and the program couldn’t continue to run. `std::abort()` does not do any cleanup.

We will see cases in future lesson [9.6 -- Assert and static_assert](https://www.learncpp.com/cpp-tutorial/assert-and-static_assert/) where `std::abort` is called implicitly.

The `std::terminate()` function is typically used in conjunction with `exceptions`. 
Although `std::terminate` can be called explicitly, it is more often called implicitly when an exception isn’t handled. By default, `std::terminate()` calls `std::abort()`.

- `std::abort()`:*The raw mechanism that actually kills the process.
- **`std::terminate()`:** The C++ error-handling middleman. By default, it just calls `std::abort()`, but you can intercept it to run custom code (like saving logs) right before the program dies.

#### When should you use a halt?

The short answer is “almost never”. Destroying local objects is an important part of C++, and none of the above-mentioned functions clean up local variables. Exceptions are a better and safer mechanism for handling error cases.

---