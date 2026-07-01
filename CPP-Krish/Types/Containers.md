In general programming, a **container** is a data type that provides storage for a collection of unnamed objects (called **elements**). The elements of a container are unnamed. 

It is why plain structs typically aren’t considered containers, their data members require unique names.

In most programming languages (including C++), containers are **homogenous**, meaning the elements of a container are required to have the same type **Heterogenous** containers allow elements to be different types. Heterogeneous containers are typically supported by scripting languages (such as Python).

The **Containers library** is a part of the C++ standard library that contains various class types that implement some common types of containers. A class type that implements a container is sometimes called a **container class**. Only the class types in the Containers library are considered to be containers in C++.

The following types are containers under the general programming definition, but are not considered to be containers by the C++ standard:
- C-style arrays
- `std::string`
- `std::vector<bool>`

To be a container in C++, the container must implement all of the requirements listed [here](https://en.cppreference.com/w/cpp/named_req/Container). Note that these requirements include the implementation of certain member functions, this implies that C++ containers must be class types. The types listed above do not implement all of these requirements.

Arrays are one of the few container types that allow for **random access**, meaning any element in the container can be accessed directly (as opposed to sequential access, where elements must be accessed in a particular order).

---
