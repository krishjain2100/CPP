variables with static duration (eg: global variables) are zero-initialised by default.


const globals have internal linkage
non-const globals have external linkage
functions have external linkage
Inline functions and variables have external linkage by default
static data memebrs have external linkage


constexpr functions are inline
constexpr variables are not inline
But static constexpr data memebers variables are  inline
Member functions defined inside the class are inline and so are static member functions defined inside



Use `std::cin.get(x)` does not ignore whitespaces


