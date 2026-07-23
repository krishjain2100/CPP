Input and output functionality is not defined as part of the core C++ language, but rather is provided through the C++ standard library and thus resides in the std namespace. 

#### **The iostream library**

When you include the iostream header, you gain access to a whole hierarchy of classes responsible for providing I/O functionality, including one class that is itself named iostream.

#### **Streams**

I/O in C++ is implemented with streams, which is just a sequence of bytes that can be accessed sequentially. Over time, a stream may produce or consume potentially unlimited amounts of data. Typically we deal with two different types of streams. 

- _Input streams_ are used to hold input from a data producer, such as a keyboard, a file, or a network. For example, the user may press a key on the keyboard while the program is currently not expecting any input. The data is put into an input stream, where it will wait until the program is ready for it.

- _Output streams_ are used to hold output for a particular data consumer, such as a monitor, a file, or a printer. When writing data to an output device, the device may not be ready to accept that data yet, for example, the printer may still be warming up when the program writes data to its output stream. The data will sit in the output stream until the printer begins consuming it.

Some devices, such as files and networks, are capable of being both input and output sources.

Programmer only has to learn how to interact with the streams in order to read and write data to many different kinds of devices. The details about how the stream interfaces with the actual devices they are hooked up to is left up to the environment or OS.

---
### Input/Output in C++

`ios` is a typedef for `std::basic_ios<char>` that defines a bunch of stuff that is common to both input and output streams. We’ll deal with this below.

The `istream` class is the primary class used when dealing with input streams. With input streams, the extraction operator (`>>`) is used to remove values from the stream. When the user presses a key on the keyboard, the key code is placed in an input stream. Your program then extracts the value from the stream so it can be used.

The `ostream` class is the primary class used when dealing with output streams. With output streams, the insertion operator (`<<`) is used to put values in the stream. You insert your values into the stream, and the data consumer (e.g. monitor) uses them.

The **iostream** class can handle both input and output, allowing bidirectional I/O.

---
### Standard streams in C++

A standard stream is a pre-connected stream provided to a computer program by its environment. C++ comes with 4 predefined standard stream objects that have already been set up for your use:

1. `cin` : an `istream` object tied to the standard input (typically the keyboard)
2. `cout` : an `ostream` object tied to the standard output (typically the monitor)
3. `cerr` : an `ostream` object tied to the standard error (typically the monitor), providing unbuffered output
4. `clog` : an `ostream` object tied to the standard error (typically the monitor), providing buffered output

Unbuffered output is typically handled immediately, whereas buffered output is typically stored and written out as a block. Because clog isn’t used very often, it is often omitted from the list of standard streams.

---
### Input with `istream`

When reading strings, one common problem with the extraction operator is how to keep the input from overflowing your buffer. Given the following example:

```cpp
char buf[10]{};
std::cin >> buf;
```

What happens if the user enters 18 characters? The buffer overflows, and bad stuff happens. Generally speaking, it’s a bad idea to make any assumption about how many characters your user will enter.

One way to handle this problem is through use of manipulators. 
A **manipulator** is an object that is used to modify a stream when applied with the extraction (>>) or insertion (<<) operators.  `std::endl` is a manipulator which prints a newline character and flushes any buffered output. C++ provides a manipulator known as `setw` (in the `iomanip` header) that can be used to limit the number of characters read in from a stream. To use `setw()`, simply provide the maximum number of characters to read as a parameter, and insert it into your input statement like such:

```cpp
char buf[10]{};
std::cin >> std::setw(10) >> buf;
```

This program will now only read the first 9 characters out of the stream (leaving room for a terminator). Any remaining characters will be left in the stream until the next extraction.

 `std::setw` works for many types not only strings, now you might wonder how it knows to leave one character for null-terminator? Answer is it doesn't, what it does is set a internal variable width in `std::cin`, the rest of the work is done by the operator for input of C-style strings.
 So if used with integers, it would extract 10 digit number or till whitespace is reached.

**Extraction and whitespace**

Reminder: The extraction operator skips whitespace (blanks, tabs, and newlines). Example:

```cpp
char ch{};
while (std::cin >> ch) std::cout << ch;

```

Input: Hello my name is Alex
Output: HellomynameisAlex

To not discard whitespaces, the `istream` class provides many functions

- **get()**, which simply gets a character from the input stream(including spaces and newline) Example:
```cpp
char ch{};
while (std::cin.get(ch)) std::cout << ch;
```

Input: 
Hello my name 
is Alex
Output: Hello my name 
is Alex
To send the EOF signal and terminate the above loop from your keyboard, you must use a shortcut:
- **macOS / Linux:** Press `Ctrl + D` (usually on an empty line).
- **Windows:** Press `Ctrl + Z` and then press `Enter`.

get() also has a string version that takes a maximum number of characters to read(this stops at newline and also doesn't remove the newline from the buffer):

```cpp
char strBuf[11]{};
std::cin.get(strBuf, 11);
std::cout << strBuf << '\n';
```

Input: Hello my name is Alex
Output: Hello my n

Note that we only read the first 10 characters (we had to leave one character for a terminator). The remaining characters were left in the input stream.

Note that `get()` does not read in a newline character and leaves it in the stream. This can cause some unexpected results:

```cpp
char strBuf[11]{};
// Read up to 10 characters
std::cin.get(strBuf, 11);
std::cout << strBuf << '\n';

// Read up to 10 more characters
std::cin.get(strBuf, 11);
std::cout << strBuf << '\n';
```

Input: 
Hello!
myiajkga
Output: Hello! 
and then terminate.

Why didn’t it ask for 10 more characters? Because the first `get()` read up to the newline and then stopped. The second get() saw there was still input in the `cin` stream and tried to read it. But the first character was the newline, so it stopped immediately.

Consequently, there is another function called `getline()` that works similarly to `get()`, but will extract (and discard) the delimiter.

```cpp
char strBuf[11]{};
// Read up to 10 characters
std::cin.getline(strBuf, 11);
std::cout << strBuf << '\n';
// Read up to 10 more characters
std::cin.getline(strBuf, 11);
std::cout << strBuf << '\n';
return 0;
}
```

This code will perform as you expect, even if the user enters a string with a newline in it.

To know how many character were extracted by the last call of `getline()`, use `gcount()`:

```cpp
char strBuf[100]{};
std::cin.getline(strBuf, 100);
std::cout << strBuf << '\n';
std::cout << std::cin.gcount() << " characters were read" << '\n';
```

`gcount()` includes any extracted and discarded delimiters.

There is a special version of `getline()` that lives outside the `istream` class that is used for reading in variables of type `std::string`. It is not a member of either `ostream` or `istream`, and is included in the string header. Example:

```cpp
std::string strBuf{};
std::getline(std::cin, strBuf);
std::cout << strBuf << '\n';
```

**A few more useful `istream` functions**

There are a few more useful input functions:
- `ignore()`discards the first character in the stream.  
- `ignore(int nCount)` discards the first `nCount` characters.  
- `peek()` allows you to read a character from the stream without removing it from the stream.  
- `unget()` returns the last character read, back into the stream so it can be read again by the next call. (guarantees only one consecutive beyond that is entirely dependent on your compiler and how the underlying stream buffer is implemented.)
- `putback(char ch)` allows you to put a character of your choice back into the stream to be read by the next call.

---
### Output with `ostream` and `ios`

Both `istream` and `ostream` were derived from a class called `ios`. One of the jobs of `ios` (and `ios_base`) is to control the formatting options for output. There are two ways to change the formatting options: flags, and manipulators.
**Flags** are boolean variables that can be turned on and off. 
**Manipulators** are objects placed in a stream that affect the way things are input and output.

To switch a flag on, use the `setf()` function, with the appropriate flag as a parameter. For example, by default, C++ does not print a + sign in front of positive numbers. However, by using the `std::ios::showpos` flag, we can change this behavior:

```cpp
std::cout.setf(std::ios::showpos); // turn on the std::ios::showpos flag
std::cout << 27 << '\n'; // +27
```

It is possible to turn on multiple `ios` flags at once using the Bitwise OR (|) operator:

```cpp
std::cout.setf(std::ios::showpos | std::ios::uppercase); // turn on the std::ios::showpos and std::ios::uppercase flag
std::cout << 1234567.89f << '\n'; // +1.23457E+06(e capitalized)
```

To turn a flag off, use the `unsetf()` function:

```cpp
std::cout.setf(std::ios::showpos); // turn on the std::ios::showpos flag
std::cout << 27 << '\n'; // +27
std::cout.unsetf(std::ios::showpos); // turn off the std::ios::showpos flag
std::cout << 28 << '\n'; // 28
```

Many flags belong to groups, called **format group** which is a group of flags that perform similar (sometimes mutually exclusive) formatting options. For example, a format group named `basefield` contains the flags `oct`, `dec`, and `hex`, which controls the base of integral values. By default, the `dec` flag is set. Consequently, if we do this:

```cpp
std::cout.setf(std::ios::hex); // try to turn on hex output
std::cout << 27 << '\n'; // 27
```

It didn’t work. This is because `setf()` only turns flags on, it isn’t smart enough to turn mutually exclusive flags off. Consequently, when we turned `std::hex` on, `std::ios::dec` was still on, and `std::ios::dec` apparently takes precedence. There are two ways to handle this:

- We can turn off `std::ios::dec` so that only `std::hex` is set:

```cpp
std::cout.unsetf(std::ios::dec); // turn off decimal output
std::cout.setf(std::ios::hex); // turn on hexadecimal output
std::cout << 27 << '\n'; // 1b
```

- Use a different form of `setf()` that takes two parameters: the first parameter is the flag to set, and the second is the formatting group it belongs to. When using this form of `setf()`, all of the flags belonging to the group are turned off, and only the flag passed in is turned on. For example:

```cpp
// Turn on std::ios::hex as the only std::ios::basefield flag
std::cout.setf(std::ios::hex, std::ios::basefield);
std::cout << 27 << '\n'; // 1b
```

Using `setf()` and `unsetf()` tends to be awkward, so C++ provides a second way to change the formatting options: manipulators. They are smart enough to turn on and off the appropriate flags. Example:

```cpp
std::cout << std::hex << 27 << '\n'; // 1b // hex
std::cout << 28 << '\n'; // 1c // Still in hex
std::cout << std::dec << 29 << '\n'; // 29 // back to decimal
```

In general, using manipulators is much easier than setting and unsetting flags. 
Many options are available via both flags and manipulators, however, other options are only available via flags or via manipulators, so it’s important to know how to use both.

**Useful formatters**

Flags live in the `std::ios` class, manipulators live in the `std namespace`, and the member functions live in the `std::ostream` class.

| Group               | Flag                | Meaning                                                                         |
| ------------------- | ------------------- | ------------------------------------------------------------------------------- |
|                     | std::ios::boolalpha | If set, booleans print “true” or “false”. <br>If not set, booleans print 0 or 1 |
|                     | std::ios::showpos   | If set, prefix positive numbers with a +                                        |
|                     | std::ios::uppercase | If set, uses upper case letters                                                 |
| std::ios::basefield | std::ios::dec       | Prints values in decimal (default)                                              |
| std::ios::basefield | std::ios::hex       | Prints values in hexadecimal                                                    |
| std::ios::basefield | std::ios::oct       | Prints values in octal                                                          |
| std::ios::basefield | (none)              | Prints values according to leading characters of value                          |

| Manipulator      | Meaning                                  |
| ---------------- | ---------------------------------------- |
| std::boolalpha   | Booleans print “true” or “false”         |
| std::noboolalpha | Booleans print 0 or 1 (default)          |
| std::showpos     | Prefixes positive numbers with a +       |
| std::noshowpos   | Doesn’t prefix positive numbers with a + |
| std::uppercase   | Uses upper case letters                  |
| std::nouppercase | Uses lower case letters                  |
| std::dec         | Prints values in decimal                 |
| std::hex         | Prints values in hexadecimal             |
| std::oct         | Prints values in octal                   |

**Precision, notation, and decimal points**

Using manipulators (or flags), it is possible to change the precision and format with which floating point numbers are displayed.

|Group|Flag|Meaning|
|---|---|---|
|std::ios::floatfield|std::ios::fixed|Uses decimal notation for floating-point numbers|
|std::ios::floatfield|std::ios::scientific|Uses scientific notation for floating-point numbers|
|std::ios::floatfield|(none)|Uses fixed for numbers with few digits, scientific otherwise|
|std::ios::floatfield|std::ios::showpoint|Always show a decimal point and trailing 0’s for floating-point values|

| Manipulator            | Meaning                                                                      |
| ---------------------- | ---------------------------------------------------------------------------- |
| std::fixed             | Use decimal notation for values                                              |
| std::scientific        | Use scientific notation for values                                           |
| std::showpoint         | Show a decimal point and trailing 0’s for floating-point values              |
| std::noshowpoint       | Don’t show a decimal point and trailing 0’s for floating-point values        |
| std::setprecision(int) | Sets the precision of floating-point numbers (defined in the iomanip header) |

|Member function|Meaning|
|---|---|
|std::ios_base::precision()|Returns the current precision of floating-point numbers|
|std::ios_base::precision(int)|Sets the precision of floating-point numbers and returns old precision|

If fixed or scientific notation is used, precision determines how many decimal places in the fraction is displayed. Note that if the precision is less than the number of significant digits, the number will be rounded.

```cpp
std::cout << std::fixed << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n'; // 123.456
std::cout << std::setprecision(4) << 123.456 << '\n'; // 123.4560
std::cout << std::setprecision(5) << 123.456 << '\n'; // 123.45600
std::cout << std::setprecision(6) << 123.456 << '\n'; // 123.456000
std::cout << std::setprecision(7) << 123.456 << '\n'; // 123.4560000

std::cout << std::scientific << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n'; // 1.235e+002
std::cout << std::setprecision(4) << 123.456 << '\n'; // 1.2346e+002
std::cout << std::setprecision(5) << 123.456 << '\n'; // 1.23456e+002
std::cout << std::setprecision(6) << 123.456 << '\n'; // 1.234560e+002
std::cout << std::setprecision(7) << 123.456 << '\n'; // 1.2345600e+002
```

If neither fixed nor scientific are being used, precision determines how many significant digits should be displayed. Again, if the precision is less than the number of significant digits, the number will be rounded.

```cpp
std::cout << std::setprecision(3) << 123.456 << '\n'; // 123
std::cout << std::setprecision(4) << 123.456 << '\n'; // 123.5
std::cout << std::setprecision(5) << 123.456 << '\n'; // 123.46
std::cout << std::setprecision(6) << 123.456 << '\n'; // 123.456
std::cout << std::setprecision(7) << 123.456 << '\n'; // 123.456
```

Using the showpoint manipulator or flag, you can make the stream write a decimal point and trailing zeros.

```cpp
std::cout << std::showpoint << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n'; // 123.
std::cout << std::setprecision(4) << 123.456 << '\n'; // 123.5
std::cout << std::setprecision(5) << 123.456 << '\n'; // 123.46
std::cout << std::setprecision(6) << 123.456 << '\n'; // 123.456
std::cout << std::setprecision(7) << 123.456 << '\n'; // 123.4560
```


**Width, fill characters, and justification**

Typically when you print numbers, the numbers are printed without any regard to the space around them. However, it is possible to left or right justify the printing of numbers. In order to do this, we have to first define a field width, which defines the number of output spaces a value will have. If the actual number printed is smaller than the field width, it will be left or right justified (as specified). If the actual number is larger than the field width, it will overflow the field.

| Group                 | Flag               | Meaning                                                              |
| --------------------- | ------------------ | -------------------------------------------------------------------- |
| std::ios::adjustfield | std::ios::internal | Left-justifies the sign of the number, and right-justifies the value |
| std::ios::adjustfield | std::ios::left     | Left-justifies the sign and value                                    |
| std::ios::adjustfield | std::ios::right    | Right-justifies the sign and value (default)                         |

|Manipulator|Meaning|
|---|---|
|std::internal|Left-justifies the sign of the number, and right-justifies the value|
|std::left|Left-justifies the sign and value|
|std::right|Right-justifies the sign and value|
|std::setfill(char)|Sets the parameter as the fill character (defined in the iomanip header)|
|std::setw(int)|Sets the field width for input and output to the parameter (defined in the iomanip header)|

|Member function|Meaning|
|---|---|
|std::basic_ostream::fill()|Returns the current fill character|
|std::basic_ostream::fill(char)|Sets the fill character and returns the old fill character|
|std::ios_base::width()|Returns the current field width|
|std::ios_base::width(int)|Sets the current field width and returns old field width|

In order to use any of these formatters, we first have to set a field width. This can be done via the `width(int)` member function, or the `setw()` manipulator. Note that right justification is the default.

```cpp
std::cout << -12345 << '\n'; // print default value with no field width
std::cout << std::setw(10) << -12345 << '\n'; // print default with field width
std::cout << std::setw(10) << std::left << -12345 << '\n'; 
// print left justified
std::cout << std::setw(10) << std::right << -12345 << '\n'; 
// print right justified
std::cout << std::setw(10) << std::internal << -12345 << '\n'; 
// print internally justified
```

This produces the result:
```
-12345
    -12345
-12345
    -12345
-    12345
```

Note: `setw()` and `width()` only affect the next output statement. They are not persistent like some other flags/manipulators.

Now, let’s set a fill character and do the same example:

```cpp
std::cout.fill('*');
std::cout << -12345 << '\n'; // print default value with no field width
std::cout << std::setw(10) << -12345 << '\n'; // print default with field width
std::cout << std::setw(10) << std::left << -12345 << '\n'; // print left justified
std::cout << std::setw(10) << std::right << -12345 << '\n'; // print right justified
std::cout << std::setw(10) << std::internal << -12345 << '\n'; // print internally justified
```

This produces the output: all the blank spaces in the field have been filled with the fill character.

```
-12345
****-12345
-12345****
****-12345
-****12345
```

---
### Stream classes for strings

There is another set of classes called the `stream classes` for strings that allow you to use the insertions `(<<)` and extraction `(>>)` operators to work with strings. Like `istream` and `ostream`, the string streams provide a buffer to hold data. However, unlike `cin` and `cout`, these streams are not connected to an I/O channel (such as a keyboard, monitor, etc…). 
One of the primary uses of string streams is to buffer output for display at a later time, or to process input line-by-line.

There are six stream classes for strings: 
- `istringstream` (derived from `istream`), `ostringstream` (derived from `ostream`), `stringstream` (derived from `iostream`) are used for reading and writing normal characters width strings. 
- `wistringstream`, `wostringstream`, and `wstringstream` are used for reading and writing wide character strings. 

To use the string streams, you need to `#include` the `sstream` header.
There are two ways to get data into a `stringstream`:

1. Use the insertion (<<) operator:

```cpp
std::stringstream os {};
os << "en garde!\n"; // insert "en garde!" into the stringstream
```

2. Use the str(string) function to set the value of the buffer:

```cpp
std::stringstream os {};
os.str("en garde!"); // set the stringstream buffer to "en garde!"
```

There are similarly two ways to get data out of a `stringstream`:

1. Use the `str()` function to retrieve the results of the buffer:

```cpp
std::stringstream os {};
os << "12345 67.89\n";
std::cout << os.str(); // 12345 67.89
```

2. Use the extraction (>>) operator:

```cpp
std::stringstream os {};
os << "12345 67.89"; // insert a string of numbers into the stream
std::string strValue {};
os >> strValue;
std::string strValue2 {};
os >> strValue2;
// print the numbers separated by a dash
std::cout << strValue << " - " << strValue2 << '\n'; // 12345 - 67.89
std::cout << os.str();// 12345 67.89
```

Note that the `>>` operator iterates through the string , each successive use of `>>` returns the next extractable value in the stream. On the other hand, `str()` returns the whole value of the stream, even if the `>>` has already been used on the stream.


**Conversion between strings and numbers**

Because the insertion and extraction operators know how to work with all of the basic data types, we can use them in order to convert strings to numbers or vice versa. 

Converting numbers into a string:

```cpp
std::stringstream os {};
constexpr int nValue { 12345 };
constexpr double dValue { 67.89 };
os << nValue << ' ' << dValue;
std::string strValue1, strValue2;
os >> strValue1 >> strValue2;
std::cout << strValue1 << ' ' << strValue2 << '\n'; // 12345 67.89
```

Convert a numerical string to a number:

```cpp
std::stringstream os {};
os << "12345 67.89"; // insert a string of numbers into the stream
int nValue {};
double dValue {};
os >> nValue >> dValue;
std::cout << nValue << ' ' << dValue << '\n'; // 12345 67.89
```

**Clearing a `stringstream` for reuse**

1. Set it to the empty string using str() with a blank C-style string:

```cpp
std::stringstream os {};
os << "Hello ";
os.str(""); // erase the buffer
os << "World!"; 
std::cout << os.str(); // World!
```

2. Set it to the empty string using str() with a blank std::string object:

```cpp
std::stringstream os {};
os << "Hello ";
os.str(std::string{}); // erase the buffer
os << "World!";
std::cout << os.str(); // World!
```

When clearing out a `stringstream`, it is also generally a good idea to call the `clear()` function:

```cpp
std::stringstream os {};
os << "Hello ";
os.str(""); // erase the buffer
os.clear(); // reset error flags
os << "World!";
std::cout << os.str();
```

`clear()` resets any error flags that may have been set and returns the stream back to the ok state. We will talk more about the stream state and error flags below.

---
### Stream states 

The `ios_base` class contains several state flags that are used to signal various conditions that may occur when using streams:

| Flag    | Meaning                                                                                              |
| ------- | ---------------------------------------------------------------------------------------------------- |
| goodbit | Everything is okay                                                                                   |
| badbit  | Some kind of fatal error occurred (e.g. the program tried to read past the end of a file)            |
| eofbit  | The stream has reached the end of a file                                                             |
| failbit | A non-fatal error occurred (e.g. the user entered letters when the program was expecting an integer) |

Although these flags live in `ios_base`, because `ios` is derived from `ios_base` and `ios` takes less typing than `ios_base`, they are generally accessed through `ios` (e.g. as `std::ios::failbit`). `ios` provides a number of member functions in order to conveniently access these states:

|Member function|Meaning|
|---|---|
|good()|Returns true if the goodbit is set (the stream is ok)|
|bad()|Returns true if the badbit is set (a fatal error occurred)|
|eof()|Returns true if the eofbit is set (the stream is at the end of a file)|
|fail()|Returns true if the failbit is set (a non-fatal error occurred)|
|clear()|Clears all flags and restores the stream to the goodbit state|
|clear(state)|Clears all flags and sets the state flag passed in|
|rdstate()|Returns the currently set flags|
|setstate(state)|Sets the state flag passed in|

The most commonly dealt with bit is the failbit, which is set when the user enters invalid input:

```cpp
std::cout << "Enter your age: ";
int age {};
std::cin >> age;
```

If the user enters non-numeric data, such as “Alex”, `cin` will be unable to extract anything to age, and the `failbit` will be set. If an error occurs and a stream is set to anything other than `goodbit`, further stream operations on that stream will be ignored. This condition can be cleared by calling the `clear()` function.

---
### Input validation

Input validation can generally be broken down into two types: string and numeric.

With string validation, we accept all user input as a string, and then accept or reject that string depending on whether it is formatted appropriately. In most languages, this is done via regular expressions. The C++ standard library has a [regular expression library](https://en.cppreference.com/w/cpp/regex) as well. Because regular expressions are slow compared to manual string validation, they should only be used if performance (compile-time and run-time) is of no concern or manual validation is too cumbersome.

With numerical validation, we are typically concerned with making sure the number the user enters is within a particular range (e.g. between 0 and 20). However, unlike with string validation, it’s possible for the user to enter things that aren’t numbers at all

To help us out, the following functions live in the `cctype` header:

|Function|Meaning|
|---|---|
|std::isalnum(int)|Returns non-zero if the parameter is a letter or a digit|
|std::isalpha(int)|Returns non-zero if the parameter is a letter|
|std::iscntrl(int)|Returns non-zero if the parameter is a control character|
|std::isdigit(int)|Returns non-zero if the parameter is a digit|
|std::isgraph(int)|Returns non-zero if the parameter is printable character that is not whitespace|
|std::isprint(int)|Returns non-zero if the parameter is printable character (including whitespace)|
|std::ispunct(int)|Returns non-zero if the parameter is neither alphanumeric nor whitespace|
|std::isspace(int)|Returns non-zero if the parameter is whitespace|
|std::isxdigit(int)|Returns non-zero if the parameter is a hexadecimal digit (0-9, a-f, A-F)|

---
### String validation

Example: our validation criteria will be that the user enters only alphabetic characters or spaces. If anything else is encountered, the input will be rejected.

When it comes to variable length inputs, the best way to validate strings (besides using a regular expression library) is to step through each character of the string and ensure it meets the validation criteria. That’s exactly what `std::all_of` does for us.

```cpp
bool isValidName(std::string_view name) {
	return std::ranges::all_of(name, [](char ch) {
		return std::isalpha(ch) || std::isspace(ch);
	});
}

int main() {
	std::string name{};
	do {
	    std::cout << "Enter your name: ";
	    std::getline(std::cin, name); // get the entire line, including spaces
	} while (!isValidName(name));
	std::cout << "Hello " << name << "!\n";
}
```

Now we are going to ask the user to enter their phone number. A phone number is a fixed length but the validation criteria differ depending on the position of the character. We’re going to write a function that will check the user’s input against a predetermined template to see whether it matches. The template will work as follows:

- `#` will match any digit in the user input.  
- `@` will match any alphabetic character in the user input.  
- `_` will match any whitespace.  
- `?` will match anything.  
- Otherwise, the characters in the user input and the template must match exactly.


```cpp
bool inputMatches(std::string_view input, std::string_view pattern) {
    if (input.length() != pattern.length()) { return false; }
    static const std::map<char, int (*)(int)> validators{
      { '#', &std::isdigit },
      { '_', &std::isspace },
      { '@', &std::isalpha },
      { '?', [](int) { return 1; } }
    };
    return std::ranges::equal(input, pattern, [](char ch, char mask) -> bool {
        auto found{ validators.find(mask) };
        if (found != validators.end()) {
            return (*found->second)(ch);
        }
        return ch == mask;
    }); 
}

int main() {
    std::string phoneNumber{};
    do {
        std::cout << "Enter a phone number (###) ###-####: ";
        std::getline(std::cin, phoneNumber);
    } while (!inputMatches(phoneNumber, "(###) ###-####"));
    std::cout << "You entered: " << phoneNumber << '\n';
}
```

However, this function is still subject to several constraints: if `#`, `@`,  `_`, and `?` are valid characters in the user input, this function won’t work, because those symbols have been given special meanings. Also, unlike with regular expressions, there is no template symbol that means “a variable number of characters can be entered”. Thus, such a template could not be used to ensure the user enters two words separated by a whitespace, because it can not handle the fact that the words are of variable lengths. For such problems, the non-template approach is generally more appropriate.

---
### Numeric validation

The obvious way to proceed is to use the extraction operator to extract input to a numeric type. By checking the failbit, we can then tell whether the user entered a number or not:

```cpp

int main() {
    int age{};
    while (true) {
        std::cout << "Enter your age: ";
        std::cin >> age;
        if (std::cin.fail())  {
            std::cin.clear(); 
            // reset the state bits back to goodbit so we can use ignore()
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); 
            // clear out the bad input from the stream
            continue; // try again
        }
        if (age <= 0)  continue;
        break;
    }
    std::cout << "You entered: " << age << '\n';
}
```

There’s one more case we haven’t tested for, and that’s when the user enters a string that starts with numbers but then contains letters (e.g. “34abcd56”). In this case, the starting numbers (34) will be extracted into age, the remainder of the string (“abcd56”) will be left in the input stream, and the `failbit` will NOT be set. This causes two potential problems:
	
1. If you want this to be valid input, you now have garbage in your stream.
2. If you don’t want this to be valid input, it is not rejected (and you have garbage in your stream).

Let’s fix the first problem:

```cpp
int main() {
    int age{};
    while (true) {
        std::cout << "Enter your age: ";
        std::cin >> age;
        if (std::cin.fail())  {
            std::cin.clear(); 
            // reset the state bits back to goodbit so we can use ignore()
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); 
            // clear out the bad input from the stream
            continue; 
        }
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); 
        // clear out any additional input from the stream
        if (age <= 0) continue;
	    break;
    }
    std::cout << "You entered: " << age << '\n';
}
```

If you don’t want such input to be valid, we’ll have to do a little extra work. Fortunately, the previous solution gets us half way there. We can use the `gcount()` function to determine how many characters were ignored. If our input was valid, `gcount()` should return 1 (the newline character that was discarded). If it returns more than 1, the user entered something that wasn’t extracted properly, and we should ask them for new input:

```cpp
int main() {
    int age{};
    while (true) {
        std::cout << "Enter your age: ";
        std::cin >> age;
        if (std::cin.fail())  {
            std::cin.clear();
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); 
            continue; 
        }
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
        if (std::cin.gcount() > 1) continue;
        // if we cleared out more than one additional character
        if (age <= 0) continue;
        break;
    }
    std::cout << "You entered: " << age << '\n';
}
```

---
### Numeric validation as a string

Another way to process numeric input is to read it in as a string, then try to convert it to a numeric type:

```cpp
std::optional<int> extractAge(std::string_view age) {
    int result{};
    const auto end{ age.data() + age.length() }; 
    // get end iterator of underlying C-style string
    // Try to parse an int from age
    // If we got an error of some kind...
    if (std::from_chars(age.data(), end, result).ec != std::errc{}) { 
	    return {}; // return nothing
    }
    if (result <= 0)  { return {}; } // return nothing
    return result; // return an int value
}

int main() {
    int age{};
    while (true) {
        std::cout << "Enter your age: ";
        std::string strAge{};
        if (!std::getline(std::cin >> std::ws, strAge)) {
            // If we failed, clean up and try again
            std::cin.clear();
            std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            continue;
        }
        auto extracted{ extractAge(strAge) };  // Try to extract the age
		if (!extracted) continue; // If we failed, try again
        age = *extracted; // get the value
        break;
    }
  std::cout << "You entered: " << age << '\n';
}
```

---
### Basic file I/O

File I/O in C++ works very similarly to normal I/O (with a few minor added complexities). 
There are 3 basic file I/O classes in C++: 
- `ifstream` (derived from `istream`)
- `ofstream` (derived from `ostream`), 
- `fstream` (derived from `iostream`). 

These classes do file input, output, and input/output respectively. 
To use the file I/O classes, you will need to include the fstream header.

Unlike the `cout`, `cin`, `cerr`, and `clog` streams, which are already ready for use, file streams have to be explicitly set up by the programmer. However, this is simple.
To open a file for reading and/or writing, simply instantiate an object of the appropriate file I/O class, with the name of the file as a parameter. Then use the insertion (<<) or extraction (>>) operator to write to or read data from the file. To close a file: explicitly call the close() function, or just let the file I/O variable go out of scope (file I/O class destructor will close the file for you).

### **File output**

We’re going to use the `ofstream` class:

```cpp
#include <fstream>
#include <iostream>
int main() {
    std::ofstream outf{ "Sample.txt" };
    // If we couldn't open the output file stream for writing
    if (!outf) {
        std::cerr << "Uh oh, Sample.txt could not be opened for writing!\n";
        return 1;
    }
    outf << "This is line 1\n";
    outf << "This is line 2\n";
    // When outf goes out of scope, the ofstream
    // destructor will close the file
}
```

Note that it is also possible to use the `put()` function to write a single character to the file.

#### **File input**

Note that `ifstream` returns a 0 if we’ve reached the end of the file (EOF). We’ll use this fact to determine how much to read.

```cpp
#include <fstream>
#include <iostream>
#include <string>
int main() {
    std::ifstream inf{ "Sample.txt" };
    if (!inf) {
        std::cerr << "Uh oh, Sample.txt could not be opened for reading!\n";
        return 1;
    }
    std::string strInput{};
    while (inf >> strInput) std::cout << strInput << '\n';
    // When inf goes out of scope, the ifstream
    // destructor will close the file
}
// Output
	// This
	// is
	// line
	// 1
	// This
	// is
	// line
	// 2
```

Remember that the extraction operator breaks on whitespace. In order to read in entire lines, we’ll have to use the `getline()` function.

#### **Buffered output**

Output in C++ may be buffered. This means that anything that is output to a file stream may not be written to disk immediately. Instead, several output operations may be batched and handled together. This is done for performance reasons. When a buffer is written to disk, this is called **flushing** the buffer. One way to cause the buffer to be flushed is to close the file, the contents of the buffer will be flushed to disk, and then the file will be closed.

Buffering is usually not a problem, but in certain circumstance it can cause complications.
The main culprit in this case is when there is data in the buffer, and then program terminates immediately (either by crashing, or by calling `exit()`). In these cases, the destructors for the file stream classes are not executed the data in the buffer is lost forever.

It is possible to flush the buffer manually using the `ostream::flush()` function or sending `std::flush` to the output stream. 

`std::endl`; also flushes the output stream. Consequently, overuse of `std::endl` can have performance impacts when doing buffered I/O where flushes are expensive (such as writing to a file). For this reason, performance conscious programmers will often use `‘\n’` to insert a newline into the output stream, to avoid unnecessary flushing of the buffer.

#### **File modes**

The original file is completely overwritten each time the program is run. Now we want to append some more data to the end of the file The file stream constructors take an optional second parameter that allows you to specify information about how the file should be opened. This parameter is called mode, and the valid flags that it accepts live in the `ios` class.

| Ios file mode | Meaning                                                                                       |
| ------------- | --------------------------------------------------------------------------------------------- |
| app           | Opens the file in append mode(cannot move back to change original content)                    |
| ate           | Seeks to the end of the file before reading/writing(can move back to change original content) |
| binary        | Opens the file in binary mode (instead of text mode)                                          |
| in            | Opens the file in read mode (default for ifstream)                                            |
| out           | Opens the file in write mode (default for ofstream)                                           |
| trunc         | Erases the file if it already exists                                                          |

It is possible to specify multiple flags by bitwise ORing them together (using the | operator). `ifstream` defaults to `std::ios::in` file mode. 
`ofstream` defaults to `std::ios::out` file mode.
`fstream` defaults to `std::ios::in` | `std::ios::out` file mode, meaning you can both read and write by default.

Due to the way `fstream` was designed, it may fail if `std::ios::in` is used and the file being opened does not exist. If you need to create a new file using `fstream`, use `std::ios::out` mode only.

#### **Explicitly opening files using open()**

It’s possible to explicitly open a file stream using `open()`. It works just like the file stream constructors, it takes a file name and an optional file mode. For example:

```cpp
std::ofstream outf{ "Sample.txt" };
outf << "This is line 1\n";
outf << "This is line 2\n";
outf.close(); // explicitly close the file

// Oops, we forgot something
outf.open("Sample.txt", std::ios::app);
outf << "This is line 3\n";
outf.close();
```

---
### Random file I/O

#### The file pointer

Each file stream class contains a file pointer that is used to keep track of the current read/write position within the file. When something is read from or written to a file, the reading/writing happens at the file pointer’s current location. By default, when opening a file for reading or writing, the file pointer is set to the beginning of the file. However, if a file is opened in append mode, the file pointer is moved to the end of the file, so that writing does not overwrite any of the current contents of the file.

#### Random file access with `seekg()` and `seekp()`

It is also possible to do random file access, i.e., skip around to various points in the file to read its contents.  This is is done by manipulating the file pointer using either `seekg()` function (for input) and `seekp()` function (for output). g stands for “get” and the p for “put”. 
For some types of streams, `seekg()` (changing the read position) and `seekp()` (changing the write position) operate independently, however, with file streams, the read and write position are always identical, so they can be used interchangeably.

These functions take two parameters. 
- The first parameter is an offset that determines how many bytes to move the file pointer.
- The second parameter is an `ios` flag that specifies what the offset parameter should be offset from.

| Ios seek flag | Meaning                                                            |
| ------------- | ------------------------------------------------------------------ |
| beg           | The offset is relative to the beginning of the file (default)      |
| cur           | The offset is relative to the current location of the file pointer |
| end           | The offset is relative to the end of the file                      |

A positive offset means move the file pointer towards the end of the file, whereas a negative offset means move the file pointer towards the beginning of the file.

In a text file, seeking to a position other than the beginning of the file may result in unexpected behavior. In programming, a newline (‘\n’) is actually an abstraction:

- On Windows, a newline is represented as sequential CR (carriage return) and LF (line feed) characters (thus taking 2 bytes of storage).
- On Unix, a newline is represented as a LF (line feed) character (thus taking 1 byte of storage).

Seeking past a newline in either direction takes a variable number of bytes depending on how the file was encoded, which means results will vary depending on which encoding is used. Also on some operating systems, files may be padded with trailing zero bytes (bytes that have value 0). Seeking to the end of the file (or an offset from the end of the file) will produce different results on such files.

`seekg()` and `seekp()` are better used on binary files. You can open a file in binary mode via:

```cpp
std::ifstream inf {"Sample.txt", std::ifstream::binary};
```

Two other useful functions are `tellg()` and `tellp()`, which return the absolute position of the file pointer. This can be used to determine the size of a file:

```cpp
std::ifstream inf {"Sample.txt"};
inf.seekg(0, std::ios::end); // move to end of file
std::cout << inf.tellg();
```

On the author’s machine, this prints: 64
which is how long sample.txt is in bytes (assuming a newline after the last line).
The result of `64` in the prior example occurred on Windows. If you run the example on Unix, you’ll get `60` instead, due to the smaller newline representation. You may get something else if your file is padded with trailing zero bytes.

#### Reading and writing a file at the same time using `fstream`

The `fstream` class is _almost_ capable of both reading and writing a file at the same time.
The caveat here is that it is not possible to switch between reading and writing arbitrarily. Once a read or write has taken place, the only way to switch between the two is to perform an operation that modifies the file position (e.g. a seek). If you do not do this, any number of strange and bizarre things may occur. If you don’t actually want to move the file pointer (because it’s already in the spot you want), you can always seek to the current position:


```cpp
// assume iofile is an object of type fstream
iofile.seekg(iofile.tellg(), std::ios::beg); // seek to current file position
```

Note: Although it may seem that `iofile.seekg(0, std::ios::cur)` would also work, it appears some compilers may optimize this away.

`fstream` uses a buffer. If you suddenly switch from writing to reading without warning the stream, your just written data, might still be sitting in the output buffer waiting to go to the disk. If you immediately try to read, the stream might pull data from the disk _before_ your writes were actually saved, meaning you read old, outdated garbage.

1. If there is any unwritten data sitting in the write buffer, the seek command forces the system to dump it onto the actual disk immediately.
2. It throws away any pre-fetched data sitting in the read buffer, forcing the system to pull fresh data from the disk on your next read.
3. It resets the internal state machine, ensuring both the read and write mechanisms agree on the exact byte location in the file.

Also, unlike `ifstream`, where we could say `while (inf)` to determine if there was more to read, this will not work with `fstream`.

We'll write a program that opens a file, reads its contents, and changes any vowels to `#` symbol.

```cpp
int main() {
    // Note we have to specify both in and out because we're using fstream
    std::fstream iofile{ "Sample.txt", std::ios::in | std::ios::out };
    if (!iofile) {
        std::cerr << "Uh oh, Sample.txt could not be opened!\n";
        return 1;
    }

    char chChar{}; 
    while (iofile.get(chChar)) {
        switch (chChar) {
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
                // Back up one character
                iofile.seekg(-1, std::ios::cur);
                // Because we did a seek, we can now safely do a write, so
                iofile << '#';
                // Now we want to go back to read mode so the next call
                // to get() will perform correctly.  
                iofile.seekg(iofile.tellg(), std::ios::beg);
                break;
        }
    }
}
```

#### Other useful file functions

To delete a file, use the `remove()` function.
Also, the `is_open()` function will return true if the stream is currently open, and false otherwise.

#### A warning about writing pointers to disk

Do not write memory addresses to files. The variables that were originally at those addresses may be at different addresses when you read their values back in from disk during another run of the program, and the addresses will be invalid.

---
