# Resources
- [Smart pointers (Modern C++) | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170)
- [Smart Pointers in C++ - GeeksforGeeks](https://www.geeksforgeeks.org/smart-pointers-cpp/)
- [smart pointers - cppreference.com](https://en.cppreference.com/book/intro/smart_pointers)

# Smart Pointers
#smart_pointer #unique_pr #shared_ptr
A wrapper class over a pointer with an operator like `*` and `->` overloaded.
Similar to normal pointer but, it can deallocate and free destroyed object memory.
- `Destructor` is automatically called when an object goes out of scope, the dynamically allocated memory would automatically be deleted

Efficient as possible both in terms of memory and performance.

Smart pointers are defined in the `std` namespace in the [memory](https://learn.microsoft.com/en-us/cpp/standard-library/memory?view=msvc-170) header file.
```c++
#include <memory>
```

**NOTE:** *Always create smart pointers on a separate line of code*. 
- Never in a parameter list, so that a subtle resource leak won't occur due to certain parameter list allocation rules. 

Smart pointers have a `reset` member function that *releases ownership of the pointer*. This is useful when you want to free the memory owned by the smart pointer before the smart pointer goes out of scope
## Types of Smart Pointers
### unique_ptr

```cpp title:uniquePtr.cpp
// Create the object and pass it to a smart pointer
std::unique_ptr<LargeObject> pLarge = std::make_unique<LargeObject>();

//Call a method on the object
pLarge->DoSomething();

// Pass a reference to a method.
ProcessLargeObject(*pLarge);
```



# Lambda expression
## Resource 
#lambda
- [C++ Lambda](https://www.programiz.com/cpp-programming/lambda-expression)
