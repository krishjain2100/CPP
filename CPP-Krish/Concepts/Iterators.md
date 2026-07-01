Looping using indexes is more typing than needed if we only use the index to access elements. Also it also only works if the container (e.g. the array) provides direct access to elements (which some do not, such as lists). Pointer arithmetic also only works if elements are consecutive in memory (which is true for arrays, but not true for other types of containers, such as lists, trees, and maps). Range-based for-loops work for all kinds of different structures (arrays, lists, trees, maps) but the mechanism for iterating through our container is hidden. They use iterators.

An **iterator** is an object designed to traverse through a container (e.g. the values in an array, or the characters in a string), providing access to each element along the way.

A container may provide different kinds of iterators. For example, an array container might offer a forwards iterator that walks through the array in forward order, and a reverse iterator that walks through the array in reverse order.

Once the appropriate type of iterator is created, the programmer can then use the interface provided by the iterator to traverse and access elements without having to worry about what kind of traversal is being done or how the data is being stored in the container. And because C++ iterators typically use the same interface for traversal (operator++ to move to the next element) and access (operator* to access the current element), we can iterate through a wide variety of different container types using a consistent method.

All standard library containers offer direct support for iteration. They implement`begin()` and `end()` functions. The `iterator` header also contains two generic functions (`std::begin` and `std::end`) that can be used. For containers that support iterators, they are defined in the header files for those containers (e.g.`<array>`, `<vector>`).

```cpp
#include <array>   // includes <iterator>
std::array array{ 1, 2, 3 };
auto begin{ std::begin(array) };
auto end{ std::end(array) };
```

We’ll re-visit the type of iterators in a later chapter.

All we need are four things: the begin point, the end point, operator++ to move the iterator to the next element (or the end), and operator* to get the value of the current element.

With iterators, it is conventional to use `operator!=` to test whether the iterator has reached the end element, instead of `operator<` because some iterator types are not relationally comparable. `operator!=` works with all iterator types.

#### Range Based For Loops

All types that have both `begin()` and `end()` member functions, or that can be used with `std::begin()` and `std::end()`, are usable in range-based for-loops.  Behind the scenes, the range-based for-loop calls `begin()` and `end()` of the type to iterate over. 
- `std::array` has `begin` and `end` member functions, so we can use it in a range-based loop. 
- C-style fixed arrays can be used with `std::begin` and `std::end` functions, so we can loop through them with a range-based loop as well. 
- Dynamic C-style arrays (or decayed C-style arrays) don’t work though, because there is no `std::end` function for them (because the type information doesn’t contain the array’s length).

---
### Iterator invalidation (dangling iterators)

Iterators can be left dangling if the elements being iterated over change address or are destroyed. When this happens, we say the iterator has been invalidated. Accessing an invalidated iterator produces undefined behavior.

Some operations that modify containers (such as adding an element to a `std::vector`) can have the side effect of causing the elements in the container to change addresses becuase of reallocation. When this happens, existing iterators to those elements will be invalidated.

Since range-based for-loops use iterators behind the scenes, we must be careful not to do anything that invalidates the iterators of the container we are traversing.

Here’s another example of iterator invalidation:

```cpp
std::vector v{ 1, 2, 3, 4, 5, 6, 7 };
auto it{ v.begin() };
++it; 
std::cout << *it << '\n'; // ok: prints 2
v.erase(it); 
// erase() invalidates iterators to the erased element (and subsequent elements)
// so iterator "it" is now invalidated
++it; // undefined behavior
std::cout << *it << '\n'; // undefined behavior
```

Invalidated iterators can be revalidated by assigning a valid iterator to them (e.g. `begin()`, `end()`, or the return value of some other function that returns an iterator).

The `erase()` function returns an iterator to the element one past the erased element (or `end()` if the last element was removed). Therefore, we can fix the above code like this:

```cpp
std::vector v{ 1, 2, 3, 4, 5, 6, 7 };
auto it{ v.begin() };
++it; 
std::cout << *it << '\n'; // 2
it = v.erase(it);
std::cout << *it << '\n'; // now ok, prints 3
```

---

