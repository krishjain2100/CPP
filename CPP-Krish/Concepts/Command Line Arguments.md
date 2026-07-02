**Command line arguments** are optional string arguments that are passed by the operating system to the program when it is launched. The program can then use them as input (or ignore them). They provide a way for people or programs to provide inputs to a _program_.

In order to pass command line arguments to WordCount, we simply list the command line arguments after the executable name.  program can have multiple command line arguments, separated by spaces:

```
WordCount Myfile.txt Myotherfile.txt
```

If you are running your program from an IDE, the IDE should provide a way to enter command line arguments (or else you'll have to type them over and over when debugging).


To access them from within our C++ program, we use a different form of main() which takes two arguments (named `argc` and `argv` by convention) as follows:

```cpp
int main(int argc, char* argv[])
```

You will sometimes also see it written as:

```cpp
int main(int argc, char** argv)
```

Even though these are treated identically, we prefer the first representation because it’s intuitively easier to understand.

`argc`: 
	It is an integer parameter containing a count of the number of arguments passed to the program. It will always be at least 1, because the first argument is always the name of the program itself. On some operating systems, `argv[0]` can end up as an empty string instead of the program's name.

`argv`: 
	It is where the actual argument values are stored (argv = argument values, though the proper name is argument vectors). It is just a C-style array of char pointers (each of which points to a C-style string). The length of this array is argc.


```cpp
int main(int argc, char* argv[]) {
    std::cout << "There are " << argc << " arguments:\n";
    for (int count{ 0 }; count < argc; ++count) {
        std::cout << count << ' ' << argv[count] << '\n';
    }
}
```

Note that we cannot use a range-based for-loop to iterate through `argv`, since range-based for-loops don’t work on decayed C-style arrays.

To use a command line argument as a number, you must convert it from a string to a number. Unfortunately, this is tough.

```cpp
#include <iostream>
#include <sstream> // for std::stringstream
#include <string>

int main(int argc, char* argv[]){
	if (argc <= 1) {
		// We'll conditionalize our response on whether argv[0] is empty or not.
		if (argv[0])std::cout << "Usage: " << argv[0] << " <number>" << '\n';
		else std::cout << "Usage: <program name> <number>" << '\n';
		return 1;
	}

	std::stringstream convert{ argv[1] }; // set up a stringstream variable
	// named convert, initialized with the input from argv[1]
	int myint{};
	if (!(convert >> myint)) // do the conversion
		myint = 0; // if conversion fails, set myint to a default value
		
	std::cout << "Got integer: " << myint << '\n';
}
```

`std::stringstream` works much like `std::cin`. In this case, we’re initializing it with the value of `argv[1]`, so that we can use operator>> to extract the value to an integer variable (the same as we would with `std::cin`).

---
### Role of OS

When you type something at the command line (or run your program from the IDE), it is the operating system’s responsibility to translate and route that request. This also involves parsing any arguments to determine how they should be handled and passed to the application. Generally, operating systems have special rules about how special characters like double quotes and backslashes are handled.

For example:
```
MyArgs Hello world!
```

Prints:
```
There are 3 arguments:
0 C:\MyArgs
1 Hello
2 world!
```

Typically, strings passed in double quotes are considered to be part of the same string:
```
MyArgs "Hello world!"
```

Prints:
```
There are 2 arguments:
0 C:\MyArgs
1 Hello world!
```

Most operating systems will allow you to include a literal double quote by backslashing the double quote:
```
MyArgs \"Hello world!\"
```

Prints:
```
There are 3 arguments:
0 C:\MyArgs
1 "Hello
2 world!"
```

---