**Software testing** is the process of determining whether or not the software actually works as expected. 
Testing a small part of your code in isolation to ensure that “unit” of code is correct is called **unit testing**. Each **unit test** is designed to ensure that a particular behavior of the unit is correct.

---
### Automating your test functions

We can do better by writing a test function that contains both the tests AND the expected answers and compares them so we don’t have to. One method is to use `assert`, which will cause the program to abort with an error message if any test fails. 

---
### Unit testing frameworks

There are frameworks (called **unit testing frameworks**) that are designed to help simplify the process of writing, maintaining, and executing unit tests. Since these involve third party software, we won’t cover them here, but you should be aware they exist.

---
### Integration testing

Once each of your units has been tested in isolation, they can be integrated into your program and retested to make sure they were integrated properly. This is called an **integration test**. 

---
### Code coverage

**Code coverage** describes how much of the source code of a program is executed while testing. There are many different metrics used for code coverage. 

1. Statement coverage
	It  refers to the percentage of statements in your code that have been exercised by your testing routines.

2. Branch coverage
	It refers to the percentage of branches that have been executed, each possible branch counted separately. An `if statement` has two branches, a branch that executes when the condition is `true`, and a branch that executes when the condition is `false` (even if there is no corresponding `else statement` to execute). A switch statement can have many branches. Multiple cases that feed into the same body don’t need to be tested separately, if one works, they all should.

3. Loop coverage
	Informally called **the 0, 1, 2 test**, it says that you should ensure the loop works properly when it iterates 0 times, 1 time, and 2 times. If it works correctly for the 2-iteration case, it should work correctly for all iterations greater than 2. These three tests therefore cover all possibilities since a loop can’t execute a negative number of times.

---
### Types of Applications

1. **Interactive applications** are those that the user will interact with after running. Most standalone applications, like games and music apps, fall into this category.
2. **Non-interactive applications** are applications that do not require user interaction to operate. The output of these programs may be used as input for another application

Within non-interactive applications, there are two types:
- **Tools** are non-interactive applications that are typically launched in order to produce some immediate result, and then terminate after producing such a result. An example of this is Unix’s grep command, which is a utility that searches text for lines that match some pattern.
- **Services** are non-interactive applications that typically run silently in the background to perform some ongoing function. Example: virus scanner.

---
### Handling invalid input

Here’s a simplified view of how `operator>>` works for input:
1. First, leading whitespace (spaces, tabs, and newlines at the front of the buffer) is discarded from the input buffer. This will discard any unextracted newline character remaining from a prior line of input.
2. If the input buffer is now empty, `operator>>` will wait for the user to enter more data. Leading whitespace is again discarded.
3. `operator>>` then extracts as many consecutive characters as it can, until it encounters either a newline character (representing the end of the line of input) or a character that is not valid for the variable being extracted to.

The result of the extraction is as follows:
- If any characters were extracted in step 3 above, extraction is a success. The extracted characters are converted into a value that is then assigned to the variable.
- If no characters could be extracted in step 3 above, extraction has failed. The object being extracted to is assigned the value `0` (as of C++11), and any future extractions will immediately fail (until `std::cin` is cleared).

---
### Preconditions, invariants, and postconditions

In programming, a **precondition** is any condition that must be true prior to the execution of some section of code (typically the body of a function).
Preconditions for a function are best placed at the top of a function, using an early return to return back to the caller if the precondition isn’t met.

An **invariant** is a condition that must be true while some section of code is executing. This is often used with loops, where the loop body will only execute so long as the invariant is true.

Similarly, a **postcondition** is something that must be true after the execution of some section of code. 

---
### `assert `and `static_assert`

If the program terminates (via `std::exit`) then we will have lost our call stack and any debugging information that might help us isolate the problem. `std::abort` is a better option for such cases, as typically the developer will be given the option to start debugging at the point where the program aborted.

An **assertion** is an expression that will be true unless there is a bug in the program. If the expression evaluates to `true`, the assertion statement does nothing. If the conditional expression evaluates to `false`, an error message is displayed and the program is terminated (via `std::abort`). This error message typically contains the expression that failed as text, along with the name of the code file and the line number of the assertion. This can help with debugging efforts immensely.

In C++, runtime assertions are implemented via the **assert** preprocessor macro, which lives in the `<cassert>` header. Asserts are one of the few preprocessor macros that are considered acceptable to use. An assert that has become out of date is a code correctness issue (unlike comments), so developers are less likely to let them be.

You can  make your assert statements more descriptive by adding a string literal joined by a logical AND:

```cpp
assert(found && "Car could not be found in database");
```

```
Assertion failed: found && "Car could not be found in database", file C:\\VCProjects\\Test.cpp, line 34
```

A string literal always evaluates to Boolean `true`. Thus, logical AND-ing a string literal doesn’t impact the evaluation of the assert.

Assertions are also sometimes used to document cases that were not implemented because they were not needed at the time the programmer wrote the code:

```cpp
assert(moved && "Need to handle case where student was just moved to another classroom");
```

The `assert` macro comes with a small performance cost that is incurred each time the assert condition is checked. Ideally, asserts should never be encountered in production code because your code should already be thoroughly tested. C++ comes with a built-in way to turn off asserts in production code: if the preprocessor macro `NDEBUG` is defined, the assert macro gets disabled.

For testing purposes, you can enable or disable asserts within a given translation unit. To do so, place one of the following on its own line **before** any `#includes`: `#define NDEBUG` (to disable asserts) or `#undef NDEBUG` (to enable asserts). The C++ preprocessor reads your file strictly top-to-bottom, line-by-line, and the `<cassert>` header file actively looks for that `NDEBUG` flag the exact moment it is included.

A **static_assert** is an assertion that is checked at compile-time rather than at runtime. Unlike assert, which is declared in the `<cassert>` header, `static_assert` is a keyword, so no header needs to be included to use it.

```cpp
static_assert(condition, diagnostic_message)
```

A few useful notes about `static_assert`:
- Because `static_assert` is evaluated by the compiler, the condition must be a constant expression.
- `static_assert` can be placed anywhere in the code file (even in the global namespace).
- `static_assert` is not deactivated in release builds (like normal `assert` is).
- Because the compiler does the evaluation, there is no runtime cost to a `static_assert`.

Prior to C++17, the diagnostic message must be supplied as the second parameter. Since C++17, providing a diagnostic message is optional. Favor `static_assert` over `assert()` whenever possible.

Your `assert()` expressions should have no side effects, as the assert expression won’t be evaluated when `NDEBUG` is defined (and thus the side effect won’t be applied). 

Also note that the `abort()` function terminates the program immediately, without a chance to do any further cleanup (e.g. close a file or database). Because of this, asserts should be used only in cases where corruption isn’t likely to occur if the program terminates unexpectedly.

---
