The C++ standard library comes with a bunch of functions to do these things in just a few lines of code. Additionally, these standard library functions come pre-tested, are efficient, work on a variety of different container types, and many support parallelization (the ability to devote multiple CPU threads to the same task in order to complete it faster).

The functionality provided in the algorithms library generally fall into one of three categories:
- **Inspectors**: Used to view (but not modify) data in a container. Examples include searching and counting.
- **Mutators**: Used to modify data in a container. Examples include sorting and shuffling.
- **Facilitators**: Used to generate a result based on values of the data members. Examples include objects that multiply values, or objects that determine what order pairs of elements should be sorted in.

These algorithms live in the [algorithms](https://en.cppreference.com/w/cpp/algorithm) library. 

--
### Using std::find to find an element by value 

It searches for the first occurrence of a value in a container. `std::find` takes 3 parameters: an iterator to the starting element in the sequence, an iterator to the ending element in the sequence, and a value to search for. It returns an iterator pointing to the element (if it is found) or the end of the container (if the element is not found).

For example:

```cpp
#include <algorithm>
#include <array>
#include <iostream>
int main() {
    std::array arr{ 13, 90, 99, 5, 40, 80 };
    std::cout << "Enter a value to search for and replace with: ";
    int search{};
    int replace{};
    std::cin >> search >> replace;

    auto found{ std::find(arr.begin(), arr.end(), search) };
    if (found == arr.end()) {
        std::cout << "Could not find " << search << '\n';
    } else {
        *found = replace;
    }
}
```

---
### Using std::find_if to find an element that matches some condition

The `std::find_if` function works similarly to `std::find`, but we pass in a callable object, such as a function pointer (or a lambda) too. For each element being iterated over, `std::find_if` will call this function (passing the element as an argument to the function), and the function can return `true` if a match is found, or `false` otherwise.

Example to check if any elements contain the substring “nut”:

```cpp
bool containsNut(std::string_view str) {
    return str.find("nut") != std::string_view::npos;
}

int main() {
    std::array<std::string_view, 4> arr{ "apple", "banana", "walnut", "lemon" };
    auto found{ std::find_if(arr.begin(), arr.end(), containsNut) };
    if (found == arr.end()) {
        std::cout << "No nuts\n";
    } else {
        std::cout << "Found " << *found << '\n';
    }
}
```

---
### Using std::count and std::count_if to count how many occurrences there are

`std::count` and `std::count_if` search for all occurrences of an element or an element fulfilling a condition. 

Example to count how many elements contain the substring “nut”:

```cpp
#include <algorithm>
#include <array>
#include <iostream>
#include <string_view>

bool containsNut(std::string_view str) {
	return str.find("nut") != std::string_view::npos;
}

int main() {
	std::array<std::string_view, 5> arr{ "apple", "banana", "walnut", "lemon", "peanut" };
	auto nuts{ std::count_if(arr.begin(), arr.end(), containsNut) };
	std::cout << "Counted " << nuts << " nut(s)\n";
}
```

---
### Using std::sort to custom sort 

Because sorting in descending order is so common, C++ provides a custom type (named `std::greater`) for that too (which is part of the functional  header).

```cpp
std::sort(arr.begin(), arr.end(), std::greater{}); 
// use the standard library greater comparison
// Before C++17, we had to specify the element type when we create std::greater
std::sort(arr.begin(), arr.end(), std::greater<int>{}); 
// use the standard library greater comparison
```

Note that the `std::greater{}` needs the curly braces because it is not a callable function. It’s a type, and in order to use it, we need to instantiate an object of that type. The curly braces instantiate an anonymous object of that type (which then gets passed as an argument to std::sort).

---
### Using std::for_each to do something to all elements of a container

It takes a list as input and applies a custom function to every element.
Example to  double all the numbers in an array:

```cpp
void doubleNumber(int& i) { i *= 2; }

int main() {
    std::array arr{ 1, 2, 3, 4 };
    std::for_each(arr.begin(), arr.end(), doubleNumber);
}
```

This seems unnecessary because equivalent code with a range-based for-loop is shorter and easier. Let’s compare `std::for_each` to a range-based for-loop.

```cpp
std::ranges::for_each(arr, doubleNumber); 
// Since C++20, we don't have to use begin() and end().
std::for_each(arr.begin(), arr.end(), doubleNumber); // Before C++20
for (auto& i : arr) {
    doubleNumber(i);
}
```

Additionally, `std::for_each` can skip elements at the beginning or end of a container, for example to skip the first element of `arr`, `std::next` can be used to advance begin to the next element. This isn’t possible with a range-based for-loop.

```cpp
std::for_each(std::next(arr.begin()), arr.end(), doubleNumber);
```

Like many algorithms, `std::for_each` can be parallelized to achieve faster processing.

---
### Performance and order of execution

Many of the algorithms in the algorithms library make some kind of guarantee about how they will execute. Typically these are either performance guarantees, or guarantees about the order in which they will execute. For example, `std::for_each` guarantees that each element will only be accessed once, and that the elements will be accessed in forwards sequential order.

While most algorithms provide some kind of performance guarantee, fewer have order of execution guarantees. For such algorithms, we need to be careful not to make assumptions about the order in which elements will be accessed or processed.

For example, if we were using a standard library algorithm to multiply the first value by 1, the second value by 2, the third by 3, etc… we’d want to avoid using any algorithms that didn’t guarantee a forwards sequential execution order!

The following algorithms guarantee sequential execution: `std::for_each`, `std::copy`, `std::copy_backward`, `std::move`, and `std::move_backward`. Many other algorithms (particular those that use a forward iterator) are implicitly sequential due to the forward iterator requirement.

---
### Ranges in C++20

Having to explicitly pass `arr.begin()` and `arr.end()` to every algorithm is a bit annoying. 
C++20 adds _ranges_, which allow us to simply pass `arr`. 

---
