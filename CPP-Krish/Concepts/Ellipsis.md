Ellipsis are rarely used, potentially dangerous, and we recommend avoiding their use.

Functions that use ellipsis take the form:

```
return_type function_name(argument_list, ...)
```

The _argument_list_ is one or more normal function parameters.
Note that functions that use ellipsis must have at least one non-ellipsis parameter.

The ellipsis must always be the last parameter in the function.
It captures any additional arguments (if there are any). 
Though it is not quite accurate, it is conceptually useful to think of the ellipsis as an array that holds any additional parameters beyond those in the argument_list.

```cpp
#include <cstdarg> // needed to use ellipsis

// The ellipsis must be the last parameter
// count is how many additional arguments we're passing
double findAverage(int count, ...) {
    int sum{ 0 };
    // We access the ellipsis through a va_list, so let's declare one
    std::va_list list;
    // We initialize the va_list using va_start.  
    // The first argument is the list to initialize.  
    // The second argument is the last non-ellipsis parameter.
    va_start(list, count);

    // Loop through all the ellipsis values
    for (int arg{ 0 }; arg < count; ++arg) {
         // We use va_arg to get values out of our ellipsis
         // The first argument is the va_list we're using
         // The second argument is the type of the value
         sum += va_arg(list, int);
    }

    // Cleanup the va_list when we're done.
    va_end(list);
    return static_cast<double>(sum) / count;
}

int main() {
    std::cout << findAverage(5, 1, 2, 3, 4, 5) << '\n';
    std::cout << findAverage(6, 1, 2, 3, 4, 5, 6) << '\n';
}
```

The `<cstdarg>` header defines `va_list, va_arg, va_start, and va_end`, which are macros that we need to use to access the parameters that are part of the ellipsis.

Note that the ellipsis parameter has no name. Instead, we access the values in the ellipsis through a special type known as `va_list`. It is conceptually useful to think of `va_list` as a pointer that points to the ellipsis array. 

First, we declare a `va_list`, called `list`. Then we need to do is make list point to our ellipsis parameters.  We do this by calling `va_start()`.  It takes two parameters: the `va_list` itself, and the name of the _last_ non-ellipsis parameter in the function (i.e, `count` here). Now `va_list` points to the first parameter in the ellipsis.

Now, to get the value of the parameter that `va_list` currently points to, we use `va_arg()`. 
It also takes two parameters: the `va_list` itself, and the type of the parameter we’re trying to access. Note that `va_arg()` also moves the `va_list` to the next parameter in the ellipsis.

Finally, to clean up when we are done, we call `va_end()`, with `va_list` as the parameter.
It sets `va_list` to `NULL`. Also,`va_start()` might allocate  data structures, modify stack pointers, or create temporary memory buffers to keep track of which arguments have been read.
So clean up must also deallocate these memories and restore the pointers.

Note that `va_start()` can be called again any time we want to reset the `va_list` to point to the first parameter in the ellipses again.

---
### Type checking is suspended

With regular function parameters, the compiler uses type checking to ensure the types of the function arguments match the types of the function parameters (or can be implicitly converted so they match). 

However,  ellipsis parameters have no type declarations. 
When using ellipsis, the compiler completely suspends type checking for ellipsis parameters. This means it is possible to send arguments of any type to the ellipsis. 
The downside is that the compiler will no longer be able to warn you if you call the function with ellipsis arguments that do not make sense. It is completely up to the caller to ensure the function is called with ellipsis arguments that the function can handle.

Let’s look at an example of a mistake that is pretty subtle:

```cpp
std::cout << findAverage(6, 1.0, 2, 3, 4, 5, 6) << '\n';
```

This compiles fine, and produces a somewhat surprising result: 1.78782e+008

The problem is that the double we passed in as the first ellipsis argument is 8 bytes, whereas `va_arg(list, int)` will only return 4 bytes of data with each call. Consequently, the first call to `va_arg` will only read the first 4 bytes of the double (producing a garbage result), and the second call to `va_arg` will read the second 4 bytes of the double (producing another garbage result). 

---
### Length Issue

We have to keep track of the number of parameters passed into the ellipsis.

**Method 1: Pass a length parameter**
**Method 2: Use a sentinel value**
**Method 3: Use a decoder string**

The string tells the program how to interpret the parameters

```cpp

double findAverage(std::string_view decoder, ...) {
	double sum{ 0 };
	std::va_list list;
	va_start(list, decoder);
	for (auto codetype: decoder) {
		switch (codetype) {
		case 'i':
			sum += va_arg(list, int);
			break;
		case 'd':
			sum += va_arg(list, double);
			break;
		}
	}
	va_end(list);
	return sum / std::size(decoder);
}

int main() {
	std::cout << findAverage("iiiii", 1, 2, 3, 4, 5) << '\n';
	std::cout << findAverage("iiiiii", 1, 2, 3, 4, 5, 6) << '\n';
	std::cout << findAverage("iiddi", 1, 2, 3.5, 4.5, 5) << '\n';
}
```

**This is what printf does.**

Obviously count/decoder-string is better than sentinel value because we are guaranteed to terminate if done properly, whereas a sentinel might not occur.

---
### Alternatives

To improve upon ellipses-like functionality:
- C++11 introduced `parameter packs` and `variadic templates`, which offers functionality similar to ellipses, but with strong type checking. However, significant usability challenges impeded adoption of this feature.
- In C++17, [fold expressions](https://en.cppreference.com/w/cpp/language/fold) were added, which significantly improves the usability of parameter packs, to the point where they are now a viable option.

---
