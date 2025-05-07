#Cpp 

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

**NOTE:** ==*Always create smart pointers on a separate line of code*. ==
- Never in a parameter list, so that a subtle resource leak won't occur due to certain parameter list allocation rules. 

Smart pointers have a `reset` member function that *releases ownership of the pointer*. This is useful when you want to free the memory owned by the smart pointer before the smart pointer goes out of scope
## Types
### Unique pointer

```cpp title:uniquePtr.cpp
// Create the object and pass it to a smart pointer
std::unique_ptr<LargeObject> pLarge = std::make_unique<LargeObject>();

//Call a method on the object
pLarge->DoSomething();

// Pass a reference to a method.
ProcessLargeObject(*pLarge);
```

## Resources
#Cpp #research 
- [Smart pointers (Modern C++) | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170)
- [Smart Pointers in C++ - GeeksforGeeks](https://www.geeksforgeeks.org/smart-pointers-cpp/)
- [smart pointers - cppreference.com](https://en.cppreference.com/book/intro/smart_pointers)

# Lambda expression
#lambda
`Anonymous function` - defines a function within the body of another function
- Can be passed as arguments
C++ version >> <font color="7cfc00">C++11 and higher</font>
## Syntax
Without a parameter:
```c++
void hello_world()
{
    auto fx = []{
        std::cout << "Hello World!" << "\n";
    };
    fx();
}
```

with parameters <font color="ffcba4">auto assigned return type</font>:
```c++
void hello_world()
{
    auto gx = [](double x, double y){
	    return x + y;
	};
	
	double z = gx(9.2, 2.6);    // z = 11.8
}
```

with parameters <font color="ffcba4">Explicit return type</font>:
```c++
auto gx = [](double x, double y) -> double{
	return x + y;
};
	
```

## Capture
`[]` - captures (<font color="ffcba4">nonstatic</font>) external variable, including objects, allowing to be used in the lambda function
Can be captured by:
- value
-  non-constant **reference** (<font color="ffcba4">can result to modification</font>)

```c++
vector u{1.0, 1.5, 2.0, 2.5, 3.0, 3.5};
double shift = 0.25;
auto shift_scalar_mult = [&u, shift](double alpha)
{
    for (double& x : u)
    {
        x = alpha * (x + shift);
    }
};

shift_scalar_mult(-1.0);
```
## Resource 
- [C++ Lambda](https://www.programiz.com/cpp-programming/lambda-expression)



# Enumerated Constants
#Cpp #enum
Efficient way of passing codes compared to using strings
C++ version >> <font color="7cfc00">C++11 and higher</font>

```c++
enum GroupName{
	opt1,  // default value = 0
	opt2,  // opt1+1
	opt3    // opt1+2
}
```


## Conflicts:
```c++
enum GroupName2{
	lst1,  // default value = 0
	lst2,  // opt1+1
	lst3    // opt1+2
}
```

`opt1 == lst1` as both begin incrementing from 0

**Solution :** 
```c++
enum GroupName{
	opt1 = 100,  // default value = 0
	opt2,    // 101
	opt3     // 102
};

enum GroupName2{
	lst1 = 200,  // default value = 0
	lst2,    // 201
	lst3     // 202
};
```

## Scoped enums
Avoid conflicts using `enum class` as comparing the differing class results in <font color="ff2800">compiler error</font>
- Both can start with 0 and not affect application, as you can only compare with in the same class
```C++
enum class Bond_Type
{
    Government,
    Corporate,
    Municipal
};

enum class Futures_Contract
{
    Gold,
    Silver,
    Oil
};
```

## switch statement
```c++
switch(type){
	case Bond_Type::Government:
        std::cout << "Government Bond..." << "\n"; 
        // Do stuff...
        break;
    case Bond_Type::Corporate:
        cout << "Corporate Bond..." << "\n";
        // Do stuff...
        break;
    case Bond_Type::Municipal:
        cout << "Municipal Bond..." << "\n";
        // Do stuff...
        break;
    default:
        cout << "Unknown Bond..." << "\n";
        // Check the bond type...
        break;
}
```

# String 
## Formatting
Simple format string ...duuh
```c++
#include <format>
int main{
	int arr[] = {1,2,3}
	std::string x = std::format("a = {0}, b = {2}, c = {1}", arr[0], arr[1], arr[2]); // a = 1, b = 3, c = 2

	x = std::format("a = {}, b = {}, c = {}", arr[0], arr[1], arr[2]); 
	// a = 1, b = 2, c = 3

	x = std::format("a = {2}, b = {1}, c = {1}", arr[0], arr[1], arr[2]); 
	// a = 3, b = 2, c = 2


	enum GroupName{
		opt1 = 100,  // default value = 0
		opt2,    // 101
		opt3     // 102
	};

	cout << std::format("a = {}", static_cast<int>(GroupName::opt1));
	// a = 100
}
```

# Math
A list of common mathematical functions is available on [Common mathematical functions](https://en.cppreference.com/w/cpp/numeric/math)
Contained in the Standard Library, you need :
- the `cmath` header file 
- functions scoped by the `std:: prefix:`
```c++
#include <cmath>

double trigFx(double theta, double phi){
    return std::sin(theta) + std::cos(phi);
}

double exeFx(double theta){
	using std::sin;
    return sin(theta) + 5;
}

```


## Computing polynomials
It is more efficient to apply factoring per Horner’s method and reduce the number of multiplicative operations

Instead of using `std::pow` :
	 `f(X) = 8x^4 + 7x^3 + 4x^2 - 10x - 6`
```c++
double f(double x){
    return 8.0 * std::pow(x, 4) + 7.0 * std::pow(x, 3) +
      4.0 * std::pow(x, 2) + 10.0 * x - 6.0;
}
```

Horner's method:
	 `f(x) = x(x(x(8x + 7) + 4) - 10) - 6`
```c++
double f(double x){
    return x * (x * (x * (8.0 * x + 7.0) + 4.0 * x) - 10.0) - 6.0;
}
```

## Mathematical constants
Resource : [Mathematical constants](https://en.cppreference.com/w/cpp/numeric/constants#:~:text=The%20standard%20library%20specializes%20mathematical%20constant%20variable%20templates,longdouble%2C%20and%20fixed%20width%20floating-point%20types%20%28since%20C%2B%2B23%29%29.)
To use these constants, we must:
- include the `numbers`
- Each must be scoped with the std::numbers namespace.

```c++
#include <numbers>

int main{
	double area =  std::numbers::pi * (4*4);
	// or
	using namespace std::numbers;
	double math_inv_sqrt_two_pi = inv_sqrtpi / sqrt2;	
}
```

<font color=#ff0800>Note : </font>*constants are fixed at compile time rather than computed with each call runtime, using a <font color=#7cfc00>C++11</font> designation called* <font color=#ffcba4>constexpr</font>


# Array
Reference: [std::array - cppreference.com](https://en.cppreference.com/w/cpp/container/array)
A `std::array` container is an array with the number of elements fixed at <font color="ffcba4">compile time</font>. 
- It was added to the Standard Library in <font color=#7cfc00>C++11</font>.
- For newer features version >> <font color="7cfc00">C++17 and higher</font>

```c++
#include <array>

int main{
	std::array<dataType, size_t> nameArray

	std::array<int, 3> a1{{1, 2, 3}};
}
```