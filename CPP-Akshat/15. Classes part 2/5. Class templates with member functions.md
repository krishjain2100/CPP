```cpp
#include <ios>       // for std::boolalpha
#include <iostream>

template <typename T>
class Pair{
private:
    T m_first{};
    T m_second{};

public:
    // When we define a member function inside the class definition,
    // the template parameter declaration belonging to the class applies
    Pair(const T& first, const T& second)
        : m_first{ first }
        , m_second{ second }
    {
    }

    bool isEqual(const Pair<T>& pair);
};

// When we define a member function outside the class definition,
// we need to resupply a template parameter declaration
template <typename T>
bool Pair<T>::isEqual(const Pair<T>& pair){
    return m_first == pair.m_first && m_second == pair.m_second;
}

int main(){
    Pair p1{ 5, 6 }; // uses CTAD to infer type Pair<int>
    std::cout << std::boolalpha << "isEqual(5, 6): " << p1.isEqual( Pair{5, 6} ) << '\n';
    std::cout << std::boolalpha << "isEqual(5, 7): " << p1.isEqual( Pair{5, 7} ) << '\n';

    return 0;
}
```
Note that when we define a member function inside the class template definition, we don’t need to provide a template parameter declaration for the member function. Such member functions implicitly use the class template parameter declaration.
Second, we don’t need deduction guides for CTAD to work with non-aggregate classes. A matching constructor provides the compiler with the information it needs to deduce the template parameters from the initializers.
Third, let’s look more closely at the case where we define a member function for a class template outside of the class template definition.
Since the member function definition is separate from the class template definition, we need to resupply a template parameter declaration (`template <typename T>`) so the compiler knows what `T` is.
Also, when we define a member function outside of the class, we need to qualify the member function name with the fully templated name of the class template (`Pair<T>::isEqual`, not `Pair::isEqual`).

### Injected class names
We know that the name of a constructor must match the name of the class. But in our class template for `Pair<T>` above, we named our constructor `Pair`, not `Pair<T>`. Somehow this still works, even though the names don’t match.

Within the scope of a class, the unqualified name of the class is called an **injected class name**. In a class template, the injected class name serves as shorthand for the fully templated name.

Because `Pair` is the injected class name of `Pair<T>`, within the scope of our `Pair<T>` class template, any use of `Pair` will be treated as if we had written `Pair<T>` instead. Therefore, although we named the constructor `Pair`, the compiler treats it as if we had written `Pair<T>` instead. The names now match!

This means that we can also define our `isEqual()` member function like this:
```cpp
template <typename T>
bool Pair<T>::isEqual(const Pair& pair) // note the parameter has type Pair, not Pair<T>
{
    return m_first == pair.m_first && m_second == pair.m_second;
}
```
Because this is a definition for a member function of `Pair<T>`, we’re in the scope of the `Pair<T>` class template. Therefore, any use of `Pair` is shorthand for `Pair<T>`!

### Where to define member functions for class templates outside the class
With member functions for class templates, the compiler needs to see both the class definition (to ensure that the member function template is declared as part of the class) and the template member function definition (to know how to instantiate the template). Therefore, we typically want to define both a class and its member function templates in the same location.
When a member function template is defined _outside_ the class definition, it should generally be defined immediately below the class definition. That way, anywhere the class definition can be seen, the member function template definitions immediately below the class definition will also be seen.
In the typical case where a class is defined in a header file, this means any member function templates defined outside the class should also be defined in the same header file, below the class definition.
