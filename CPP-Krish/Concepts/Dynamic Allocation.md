
The memory that a program uses is typically divided into a few different areas, called segments:
- The code segment (also called a text segment), where the compiled program sits in memory. The code segment is typically read-only.
- The bss segment (also called the uninitialized data segment), where zero-initialized global and static variables are stored.
- The data segment (also called the initialized data segment), where initialized global and static variables are stored.
- The heap, where dynamically allocated variables are allocated from.
- The call stack, where function parameters, local variables, and other function-related information are stored.

---
### Types of Memory Allocation

C++ supports three basic types of memory allocation:

- **Static memory allocation** happens for static and global variables. Memory for these types of variables is allocated once when your program is run and persists throughout the life of your program.
- **Automatic memory allocation** happens for function parameters and local variables. Memory for these types of variables is allocated when the relevant block is entered, and freed when the block is exited.
- **Dynamic memory allocation**

Both static and automatic allocation have two things in common:
- The size of the variable / array must be known at compile time.
- Memory allocation and deallocation happens automatically (when the variable is instantiated / destroyed).

Most normal variables (including fixed arrays) are allocated in **stack**. The amount of stack memory for a program is generally small, Visual Studio defaults the stack size to 1MB. If you exceed this number, stack overflow will result, and the operating system will probably close down the program.

---
### Dynamic Memory Allocation

It is a way for running programs to request memory from the operating system when needed. It is allocated from a much larger pool of memory managed by the operating system called the **heap**. On modern machines, the heap can be gigabytes in size.

Allocating a _single_ variable dynamically:

```cpp
int* ptr{ new int }; 
*ptr = 7;

int* ptr1{ new int (5) }; // use direct initialization
int* ptr2{ new int { 6 } }; // use uniform initialization
```

Note that accessing heap-allocated objects is generally slower than accessing stack-allocated objects. Because the compiler knows the address of stack-allocated objects, it can go directly to that address to get a value. Heap allocated objects are typically accessed via pointer. This requires two steps: one to get the address of the object (from the pointer), and another to get the value.

The allocation and deallocation for stack objects is done automatically. There is no need for us to deal with memory addresses, the code the compiler writes can do this for us. The allocation and deallocation for heap objects is not done automatically.

We need to explicitly tell C++ to free the memory for reuse, when we are done. 
Set deleted pointers to nullptr unless they are going out of scope immediately afterward.

```cpp
// assume ptr has previously been allocated with operator new
delete ptr; // return the memory pointed to by ptr to the operating system
ptr = nullptr; // set ptr to be a null pointer
```


The delete operator does not _actually_ delete anything. It just returns the memory being pointed to back to the operating system. The pointer variable still has the same scope as before, and can be assigned a new value (e.g. `nullptr`) just like any other variable.

When you allocate memory with `malloc` (`new calls it`, the memory manager  allocates a few extra bytes _right before_ your data to store a header. This header tells the OS exactly how large the memory block is so it knows how much to clean up later. When you call `delete ptr`, the memory manager steps backward in RAM to read that hidden header.

If the pointer is to a non-dynamically allocated variable then there is no header and reads some random values to figure out how much to delete, so bad things can happen. Note that deleting a pointer that is not pointing to dynamically allocated memory may cause bad things to happen.

A pointer that is pointing to deallocated memory is called a **dangling pointer**. Dereferencing or deleting a dangling pointer will lead to undefined behavior.  It’s possible that the memory pointed to by a dangling pointer is allocated to another application (or for the operating system’s own usage). Trying to access that memory will cause the operating system to shut the program down.

---
### Failure of `new`

When requesting memory from the operating system, in rare circumstances, the operating system may not have any memory to grant the request with. By default, if new fails, a `bad_alloc` exception is thrown.

In many cases, having new throw an exception (or having your program crash because of unhandled exception) is undesirable, so there’s an alternate form of new which return a null pointer if memory can’t be allocated. This is done by adding the constant `std::nothrow`. And now we will do null checking.

```cpp
int* value { new (std::nothrow) int };
if (!value) {}
```

Deleting a null pointer has no effect. Thus, there is no need for the following:
```cpp
// No need
if (ptr)  delete ptr; 

// Do this
delete ptr
```

---
### Memory Leaks

Dynamically allocated memory stays allocated until it is explicitly deallocated or until the program ends (and the operating system cleans it up). However, the pointers used to hold dynamically allocated memory addresses follow the normal scoping rules for local variables. This mismatch can create obvious problems (like pointer to dynamically allocated memory going out of scope).

This is called a **memory leak**. Memory leaks happen when your program loses the address of some bit of dynamically allocated memory before giving it back to the operating system. Your program can’t delete the dynamically allocated memory, because it no longer knows where it is. The operating system also can’t use this memory, because that memory is considered to be still in use by your program.

A memory leak can occur if a pointer holding the address of the dynamically allocated memory is assigned another value: This can be fixed by deleting the pointer before reassigning it. Relatedly, it is also possible to get a memory leak via double-allocation.

---
### Dynamically Allocating Arrays

We can also dynamically allocate arrays of variables. Unlike a fixed array, where the array size must be fixed at compile time, dynamically allocating an array allows us to choose an array length at runtime. It uses `new[]/delete[]`.

While you can dynamically allocate a `std::array`, you’re usually better off using a `std::vector` in this case ( as the object lives on the stack but the contents live on the heap, so when object goes out of scope, there is only there is automatic cleanup.)

```cpp
std::cout << "Enter a positive integer: ";
std::size_t length{};
std::cin >> length;
int* array{ new int[length]{} }; // value initialised
// do something
delete[] array;
```

The length of dynamically allocated arrays has type `std::size_t`. If you are using a non-constexpr int, you’ll need to `static_cast` to `std::size_t` since that is considered a narrowing conversion and your compiler will warn otherwise.

Prior to C++11, there was no easy way to initialize a dynamic array to a non-zero value (initializer lists only worked for fixed arrays). This means you had to loop through the array and assign element values explicitly.

```cpp
int* array = new int[5];
array[0] = 9; array[1] = 7; array[2] = 5; array[3] = 3; array[4] = 1;
```

However, starting with C++11, it’s now possible to initialize dynamic arrays using initializer lists!

```cpp
int fixedArray[5] = { 9, 7, 5, 3, 1 }; // initialize a fixed array before C++11
int* array{ new int[5]{ 9, 7, 5, 3, 1 } }; // initialize a dynamic array, C++11
// To prevent writing the type twice, we can use auto. 
auto* array{ new int[5]{ 9, 7, 5, 3, 1 } };
```

For consistency, fixed arrays can also be initialized using uniform initialization:

```cpp
int fixedArray[]{ 9, 7, 5, 3, 1 }; // initialize a fixed array in C++11
char fixedArray[]{ "Hello, world!" }; // initialize a fixed array in C++11
// Explicitly stating the size of the array is optional.
```

Using the scalar version of delete on an array will result in undefined behavior, such as data corruption, memory leaks, crashes, or other problems.

How does array delete[] know how much memory to delete?
Array new[] keeps track of how much memory was allocated to a variable by injecting a size cookie before the actual data and after the OS header, so that array delete[] can delete the proper amount. Unfortunately, this size/length isn’t accessible to the programmer. So you have to maintain the length variable you used for allocating.

Dynamically allocating an array allows you to set the array length at the time of allocation. However, C++ does not provide a built-in way to resize an array that has already been allocated.

---
### Pointers to pointers

A pointer to a pointer works just like a normal pointer, you can dereference it to retrieve the value pointed to. You can dereference it again to get to the underlying value. 

```cpp
int value { 5 };
int* ptr { &value };
std::cout << *ptr << '\n'; // 5
int** ptrptr { &ptr };
std::cout << **ptrptr << '\n'; // 5
```

Note that you can not set a pointer to a pointer directly to a value. However, a pointer to a pointer can be set to null:

```cpp
int value { 5 };
int** ptrptr { &&value }; // not valid
int** ptrptr { nullptr }; // valid
```

This is because the address of operator (operator&) requires an lvalue, but `&value` is an rvalue.

It’s also possible to declare a pointer to a pointer to a pointer: `int*** ptrx3;`
Arrays of pointers: `int** array { new int*[10] }; 

---
### Dynamic Multidimensional Arrays

Unlike a two dimensional fixed array, which can be easily declared:  `int array[10][5];`, 
dynamically allocating a 2D array is challenging. You may be tempted to try something like this:

```cpp
int** array { new int[10][5] }; // won’t work!
```

If the rightmost array dimension is constexpr, you can do this (better off using `auto`):

```cpp
int x { 7 }; // non-constant
int (*array)[5] { new int[x][5] }; // rightmost dimension must be constexpr

auto array { new int[x][5] }; 
```
(**DID NOT REALLY UNDERSTAND THE SYNTAX**)

The parenthesis are required so that the compiler knows we want `array` to be a pointer to an array of 5 ints (which in this case is the first row of a 7-row multidimensional array). Without the parenthesis, the compiler would interpret this as `int* array[5]`, which is an array of 5 `int*`.

This doesn’t work if the rightmost array dimension isn’t a compile-time constant. 
To handle that, we allocate an array of pointers. Then we iterate through the array of pointers and allocate a dynamic array for each array element. 

```cpp
int** array { new int*[10] }; 
for (int count { 0 }; count < 10; ++count)
    array[count] = new int[5]; // these are our columns

array[9][4] = 3; 
```

With this method, because each array column is dynamically allocated independently, it’s possible to make dynamically allocated two dimensional arrays that are not rectangular. 

Deallocating a dynamically allocated two-dimensional array using this, requires a loop too.

```cpp
for (int count { 0 }; count < 10; ++count)
    delete[] array[count];
delete[] array; // this needs to be done last else while deleting the data inside you's be accessing deallocated memory
```

Because allocating and deallocating two-dimensional arrays is complex and easy to mess up, it’s often easier to flatten a two-dimensional array.

---