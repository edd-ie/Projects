#Cpp 


# Keywords
- [noexcept](https://en.cppreference.com/w/cpp/language/noexcept) - function declared not to throw any exception >> C++11
- [nodiscard](https://en.cppreference.com/w/cpp/language/attributes/nodiscard) - return value of the function must be use, compiler warning >> C++17 
- [maybe_unused](https://en.cppreference.com/w/cpp/language/attributes/maybe_unused)- return value of the function can be ignored >> C++17 
- [nullptr](https://en.cppreference.com/w/cpp/language/nullptr) - pointer to null >> C++11

## using

Ways to ease the pain with cryptic template types
- `typedef` :
```c++
typedef std::map<std::string, std::complex<double>> complex_map;
```

- `using` >> <font color="7cfc00">C++11 and higher</font> 
```c++
using complex_map = std::map<std::string, std::complex<double>>;

int main() {
    // Declare a map using the type alias
    complex_map my_data;

    // Insert some data into the map
    my_data["point1"] = std::complex<double>(1.0, 2.0);
    my_data["point2"] = std::complex<double>(3.0, -4.0);

    return 0;
}

```

## nodiscard
#warnings 
A way to **indicate to the compiler and to other developers that the return value of the function is intended to be used and should not be ignored.**
- Version >> <font color="7cfc00">C++17 and higher</font>
- A hint or a warning mechanism. 
- If a function is marked `[[nodiscard]]` and the caller doesn't do anything with its return value (e.g., assign it to a variable, use it in an expression), the compiler can issue a warning.

```c++
[[nodiscard]] int readFile(const std::string& filename, std::string& content);

int main() {
    std::string data;
    readFile("my_file.txt", data); // Compiler might warn that the return value is ignored
    // Oops! We didn't check if the file was read successfully.
    std::cout << "File content: " << data << std::endl;
    return 0;
}
```
# Class
#oop 

#Inheritance -hollow arrowhead pointing to base class
- is a relationship
#Composition - solid diamond pointing to containing class
- has a relationship (strong)
- The contained object's lifecycle is **dependent** on the containing object.
- Often described as a "owns-a" relationship.
#Aggregation  - hollow diamond pointing to containing class
- has a relationship (weak)
- The contained object can exist even if the containing object is destroyed, and vice-versa.
- `University` has `Students`." A student can exist even if they are no longer enrolled at that particular university, and the university can exist even if a particular student graduates or leaves.

| Feature      | Inheritance (is-a)  | Composition (has-a - strong) | Aggregation (has-a - weak) |
| ------------ | ------------------- | ---------------------------- | -------------------------- |
| Relationship | Type of             | Part of, owns                | Part of, uses              |
| Lifecycle    | Dependent (subtype) | Dependent                    | Independent                |
| Association  | Strong (subtype)    | Strong                       | Weak                       |
Good read - [Uniform Initialization](https://www.geeksforgeeks.org/uniform-initialization-in-c/)
## Polymorphism
#polymorphism 
<font color=#30D5C8><strong>Polymorphism :</strong></font> the ability of objects to take on many forms
- <font color=#30D5C8><strong>Dynamic polymorphism :</strong></font> 
	- #Runtime Polymorphism or Late Binding
	- A derived object does not need to be known until runtime specific type of a derived object does not need to be known until runtime
	- Achieved through:
		- Virtual/Abstract functions - `virtual`
	- Function calls are <font color=#ffcba4><strong>dynamically bound</strong></font> - The address of the function to be executed is determined when the program is running
	- Slightly **slower** at runtime due to the overhead of <font color=#ffcba4><strong>dynamic dispatch</strong></font> (*looking up the function in the vtable*).
	- More flexible at runtime as the behaviour can adapt based on the actual object type.
	- **Use Cases:** 
		- Scenarios involving inheritance hierarchies where you want to treat objects of different derived classes uniformly through a base class interface

- <font color=#30D5C8><strong>Static polymorphism :</strong></font> 
	- #Compile-time Polymorphism or Early Binding
	- Achieved through:
		1. Function Overloading 
		2. Operator Overloading
		3. Templates #generic 
	- Function calls are <font color=#ffcba4><strong>statically bound</strong></font> - The address of the function to be executed is known before the program runs.
	- Generally **faster** at runtime because the function call is resolved directly.
	- Less flexible at runtime as the behaviour is determined at compile time.
	- **Use case**;
		- Situations where the types involved are known at compile time

### Virtual functions
#abstract #virtual
Functions declared in a base class that are intended to be overridden (redefined) by derived classes to provide specific implementations.

<font color=#ffcba4><strong>Optional Overriding :</strong></font> A virtual function in a base class can have a default implementation. 
- Derived classes can _choose_ to override this implementation if their behaviour needs to be different.
- Else, it inherits and uses the base class's implementation.
```c++
class Shape{
public:
	virtual std::string what_am_i() const;
	virtual ~Shape() = default;     // Note: virtual default destructor
}
```

### Abstract base class
Classes that cannot be instantiated without a derived object.

<font color=#ffcba4><strong>Pure virtual :</strong></font> If a base class declares a virtual function as **pure virtual** (using `= 0`), like `virtual void calculateArea() = 0;`, then the base class becomes an <font color=#30D5C8><strong>abstract base class.</strong></font> 
- Subclasses <font color=#ff2800><strong>must provide an implementation</strong></font> for all pure virtual functions they inherit to be concrete (instantiable)

The <font color=#ffcba4><strong>override</strong></font> specifier  
- Used in the derived class to explicitly indicate that you are overriding a virtual function.
- Helps the compiler catch errors if the signature doesn't match.
- C++ version >> <font color="7cfc00">C++11 and higher</font>

```c++
class Shape{
 public:
    virtual std::string what_am_i() const;   // Returns "Shape"
    virtual double area() const = 0;         // Pure virtual
    virtual ~Shape() = default;            // Note: virtual default destructor
 };

class Circle final : public Shape{
 public:
    Circle(double radius);
    std::string what_am_i() const override;    // Returns "Circle"
    double area() const override;
 private:
    double radius_;
 };
```

# Pointers
## RAII
#RAII - Resource Acquisition is initialization
A programming technique which says the <font color=#ffcba4>lifetime of a resource is guaranteed to be tied to its owner.</font> 
When an owning object goes out of scope:
- The owner is responsible for destroying its resource and returning the memory it occupied to the system.
## Clone
Refer to <font color=#ffcba4><strong>Rule of 3</strong></font>
Prior to C++11 and the advent of smart pointers, a common approach to implementing RAII
- Implementing `clone()` methods on the resource classes
- referred to as a [virtual constructor](https://oreil.ly/5K04S)
- Pass it to a variable as a reference.
- Preventing it from being deleted or modified externally.

```c++
class Resource{
	int val;
public:
	Resource(int x):val(x){}
	~Resource() = default;
	Resource* clone(){
		return new Resource(*this); // copy constructor
	}
}

class Usage{
	Resource * store;
public:
	Usage(const Resource& val):store(val.clone()){}
}

// Usage
Resource a{10}
Usage b{a};
```


# Smart Pointers
#smart_pointer
A wrapper class over a pointer with an operator like `*` and `->` overloaded.
Similar to normal pointer but, it can deallocate and free destroyed object memory.
- `Destructor` is automatically called when an object goes out of scope, the dynamically allocated memory would automatically be deleted
- C++ version >> <font color="7cfc00">C++11 and higher</font>

Efficient as possible both in terms of memory and performance.

Smart pointers are defined in the `std` namespace in the [memory](https://learn.microsoft.com/en-us/cpp/standard-library/memory?view=msvc-170) header file.
```c++
#include <memory>
```

**NOTE:** ==*Always create smart pointers on a separate line of code*. ==
- Never in a parameter list, so that a subtle resource leak won't occur due to certain parameter list allocation rules. 

Smart pointers have a `reset` member function that *releases ownership of the pointer*. This is useful when you want to free the memory owned by the smart pointer before the smart pointer goes out of scope

## Unique pointer
#unique_ptr 
<font color=#ffcba4><strong>Use case :</strong></font> Sole (unique) ownership of the referenced address. 
- Assures that only one unique pointer to a particular memory location can exist at one time.
- Possession of a unique pointer is transferred using move semantics
- Cannot be copied.
- Simplest way to avoid leaks
- increases readability, and has <font color=#7cfc00>zero or near zero run-time cost</font>.

<font color=#30D5C8>Syntax :</font>`std::unique_ptr`, with template parameter `T`
- For <font color="7cfc00">C++14 and higher</font> recommended initialization is using : `std::make_unique

```c++ title:unique_ptr.cpp
#include <memory>     // For Standard Library smart pointers
#include <utility>    // std::move

int main(){
	std::unique_ptr<string> ptr1;
	// . . .
	ptr1 = std::make_unique<string>("To err is human");
	// Alternatively (and better when possible), on one line:
	auto ptr2 = std::make_unique<string>("To moo, bovine");

	// access by derefence
	std::cout << *ptr2;

	/* Transfer ownership */
	auto ptr3 = std::move(pt2);
}
```

<font color=#30D5C8>New data :</font> assigning new data automatically deletes the old one.
<font color=#30D5C8>Delete :</font> if no longer needed before going out of scope, it can be deleted and set to `null` explicitly with its `reset()`

```c++ title:ptrReset.cpp
#include <iostream>
#include <memory>

int main() {
    // Create a unique_ptr owning an integer
    auto ptr1 = std::make_unique<int>(10);

    // Assign a new integer to ptr1
    ptr1.reset(new int(20)); // same as ptr1 = std::make_unique<int>(20);
	ptr1.reset(); // same as ptr1 = nullptr
}
```

<font color=#30D5C8>Access :</font> 
1. `obj.get()` - The `std::unique_ptr::get()` method returns a raw pointer to the object owned by the `unique_ptr`.
	- You must not delete the raw pointer returned by `get()`.
2. **`&(*y)`** :
	1. `*y` - dereferences the unique pointer, giving you a reference to the managed `Obj` object. 
	2. `&` - takes the address of that object, effectively providing a raw pointer.

```c++
int main() {
    auto y = std::make_unique<Obj>(10, "Hello", 3.14);

    // 1. Access the raw pointer using get()
    useObjValues(y.get());

    // 2.  Use a reference
     useObjValues(&*y);
}
```

<font color=#30D5C8>Moving :</font> 
`ptr2` is set to a `null` state, as `ptr3` now assumes exclusive ownership of the second `std::string object`.
- Attempting to use `*ptr2` results in <font color=#ff2800><strong>runtime error</strong></font>.
- If unsure always check if the point is still valid before use.
```c++ title:ptr_confirmation.cpp
class Obj {
    int x;
    std::string y;
    double z;
public:
    Obj(int a, std::string b, double c) : x(a), y(b), z(c) {}
    ~Obj() = default;
    void print() const { 
	    std::cout << "Obj values: x = " << x << ", y = " << y ; 
    }
};

// transfer ownership
void fx(std::make_unique<Obj> x){
	auto store = std::move(x)
	if (store) 
		store->print();
}

// Function to use the values from the Obj.  Critically, it takes a raw pointer or a reference.
void gx(const Obj* obj) { // Or void useObjValues(const Obj& obj)
    if (obj)
        std::cout << "Using Obj values: x = " << obj->getX() ;
}

int main(){
	auto y = std::make_unique<Obj>(10, "Hello", 3.14);

	// 1. Access the raw pointer using get()
    gx(y.get());

    // 2.  Use a reference
     gx(&*y);

    // y still owns the Obj.
    y->print();
    
	fx(std::move(y));
	
	// y is now nullptr 
	if(y) 
		y->print(); // will not print 
	return 0;
}
```

### Clone
- Refer to <font color=#ffcba4><strong>Rule of 5</strong></font>
```c++
#include <memory>

class Resource{
	int val;
public:
	Resource(int x):val(x){}
	~Resource() = default;
	std::unique_ptr<Resource> clone(){
		return std::make_unique<Resource>(*this); // copy constructor
	}
}

class Usage{
	std::unique_ptr<Resource> store;
public:
	Usage(std::unique_ptr<Resource> val):store(std::move(val)){}
}

// Usage
Resource a{10}
Usage b{a};
```



## Shared pointer
#shared_ptr 
A safer alternative to *raw pointers* because:
- Pointed-to memory will not be released until the last remaining shared pointer to a given shared reference goes out of scope.

<font color=#30D5C8>Reference counting :</font> count of the number of active shared pointers to a common block of heap memory.
- Pointed object persist until the count reaches zero
- At this point will the memory be deallocated.
- Preventing a memory leak and possible undefined behaviour at runtime.
- see the current count `.use_count()`

<font color=#ffcba4><strong>Use case :</strong></font>  only when shared ownership of a resource is required, such as :
- An object that provides live updated market data that is shared by multiple trading tasks.
- Multithreading situations in which a pointed-to object is shared by at least two threads and you don’t know which one will be responsible for the object’s destruction.

<font color=#30D5C8>Syntax :</font>`std::shared_ptr`, with template parameter `T`
- For <font color="7cfc00">C++14 and higher</font> recommended initialization is using : `std::make_shared`

```c++
std::shared_ptr<std::string> sp1; 
// . . . 
sp1 = std::make_shared("To err is expected"); 

// Single line 
auto sp2 = std::make_shared<string>("To nap is supine");

// accessed by dereferencing:
cout << format("Contents of sp_two: {}\n", *sp2);

// How many pointers are there
cout << format("sp_one count? {}\n", sp1.use_count()); // count = 1
```

<font color=#30D5C8>Multiple pointers :</font> same as raw pointers.
<font color=#30D5C8>Delete :</font> delete and set to `null` explicitly with its `reset()` decreasing the count.
```c++
auto sp_one_2 = sp1; 
auto sp_one_3 = sp_one_2;

sp_one_2.reset(); 
auto count = sp1.use_count(); // = 2
```

<font color=#ff0800>Note :</font> If the pointers are in a function, once the function is executed and goes out of scope all `shared_ptr` *will be destroyed & memory deallocated*

<font color=#30D5C8>Trade-offs :</font>
1. Greater performance overhead than raw pointers and unique pointers
2. In [Multithreaded](https://www.oreilly.com/library/view/c-concurrency-in/9781617294693/) applications with data that is no longer just read-only, and the underlying object is shared by two or more threads.
	- precautions need to be in place when working with multithreaded code.
	- The value of `use_count()` is not usable in this case.


## Weak pointer
#weak_ptr 
<font color=#ffcba4><strong>Use case :</strong></font> breaking cyclic dependencies that can arise where the reference count of a shared pointer never reaches zero.

## Resources
#Cpp #research 
- [Smart pointers (Modern C++) | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170)
- [Smart Pointers in C++ - GeeksforGeeks](https://www.geeksforgeeks.org/smart-pointers-cpp/)
- [smart pointers - cppreference.com](https://en.cppreference.com/book/intro/smart_pointers)


# Reference
Storing or transferring addresses

## L-value
#lvalue_reference #lvalue
An expression that:
- **Has a persistent identity:** You can take its address (using the `&` operator).
- **Typically persists beyond a single expression:** It usually represents a named variable or an object that exists in memory.
```c++
int x = 10;     // x is an lvalue
int& ref_x = x; // ref_x is an lvalue (it refers to x)
std::cout << x; // x is used as an lvalue (its value is read)
```

## R-value
#rvalue
An expression that:
- **Is typically temporary:** It often represents a temporary object that will be destroyed at the end of the full expression in which it appears.
- **Does not necessarily have a persistent identity:** You usually cannot take its address directly (though there are exceptions with named rvalue references).
```c++
10             // A literal is an rvalue
x + 5          // The result of an arithmetic operation is often an rvalue
std::string("hello") // A temporary string object is an rvalue
foo()          // If foo() returns an object by value, the result is an rvalue
```

## R-value reference
#rvalue_reference 
A type of reference in C++ that is declared using **double ampersands (`&&`)**. It allows you to bind to an rvalue.
```c++
int&& rref_int = 20;     // rref_int now refers to the temporary rvalue 20
std::string&& ref_str = std::string("world"); // rref_str refers to the temporary string
```


# Perfect Forwarding
#rvalue_reference #template #generic
Write generic functions that can forward their arguments to other functions while preserving their original value category (whether they were #lvalue or #rvalue). 
- This is crucial for writing efficient and flexible wrapper functions.

#rvalue_reference, in conjunction with **template argument deduction** and `std::forward`, enable perfect forwarding.

```c++
#include <iostream>
#include <string>
#include <utility> // For std::forward

void process(const std::string& s) {
    std::cout << "Processing lvalue: " << s << std::endl;
}

void process(std::string&& s) {
    std::cout << "Processing rvalue: " << s << std::endl;
}

template <typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg)); // Perfectly forward arg to process
}

int main() {
    std::string str = "hello";
    wrapper(str);                // Calls process(const std::string&)
    wrapper(std::string("world")); // Calls process(std::string&&)
    return 0;
}
```


# Swap
Switch values of 2 variables
- included in `<utility>`
- safe when swapping two pointers, as only the memory addresses are being exchanged.

Adding swap to a class:
- declare it as a new void member function
- It takes in the object to be copied by <font color=#ffcba4>non-const reference</font>
- apply the `std::swap(.)` function for each data member
- could be declared with the C++11 `noexcept` keyword

```c++
#include <utility>

class Exe{
	int val;
	Obj* ptr;
public:
	Exe(int x, const Obj& y):val(x), ptr(y.clone()){}
	void swap(Exe& src) noexcept;
}

// implementation:
void Exe::swap(Exe& src) noexcept{
	using std::swap;
	swap(val, src.val);
	swap(ptr, src.ptr);
}
```

<font color=#ff0800>Note :</font> As long as the class has both a copy constructor and destructor (with clean-up of any member resources) defined, the copy-and-swap idiom can be applied.

<font color=#30D5C8><strong>noexcept</strong></font> : indicate that the function doesn't throw an exception.
- add clarity to the code
- frees the compiler from making additional code that affect optimization
- if code that throws an exception is place inside, compiler will issue a warning of the conflict

# Operator
#compareTo 
Overload the original function of an operator.
- Reference for symbols that can be overloaded: [operator overloading - cppreference.com](https://en.cppreference.com/w/cpp/language/operators)

Example version >> <font color=#7cfc00><strong>since C++23</strong></font>	
```c++
struct SwapThem
{
    template<typename T>
    static void operator()(T& left, T& right) 
    {
        std::ranges::swap(left, right);
    }
 
    template<typename T>
    static void operator[](T& left, T& right)
    {
        std::ranges::swap(left, right);
    } 
};
inline constexpr SwapThem swap_them{};
 
void foo()
{
    int a = 1, b = 2;
 
    swap_them(a, b); // OK
    swap_them[a, b]; // OK
 
    SwapThem{}(a, b); // OK
    SwapThem{}[a, b]; // OK
 
    SwapThem::operator()(a, b); // OK
    SwapThem::operator[](a, b); // OK
 
    SwapThem(a, b); // error, invalid construction
    SwapThem[a, b]; // error
}
```

### Space ship operator
#compareTo 
Implement java's #compareTo functionality in C++
- Version >> <font color=#7cfc00><strong>since C++20</strong></font>	
```c++
#include <compare>

class Example{
	int num, denom;
public:
	Example(int x, int y):num(x), denom(y){}

	bool operator == (const Example& other) const = default;

	std::strong_ordering operator <==> (const Exmaple& other) const{
		if(num*other.denom < other.num*denom)
			return std::strong_ordering::less;
		else if(*this = other)
			return std::strong_ordering::equivalent;
		
		return std::strong_ordering::greater;
	}
};
```

With definitions for both `==` and `<=>` in place, we can then apply any of the six comparative operators as usual:

```c++
Example f_01{1, 5};
Example f_02{1, 2};

if(f_01 <= f_02){
    // <=> now properly evaluates the inequality 1/5 < 1/2
    // . . .
} else{
    // . . .
}
```

Using the `strong_ordering` return type means:
- any two values can be compared

The Standard Library also has a `std::partial_ordering` type that should be used for 
- floating-point types such as double and float. 
- This is because a floating type (e.g., double) can hold noncomparable assignments such as infinity and `NaN`.

<font color=#ffcba4><strong>Note :</strong></font> floating types should never be compared directly in defining `==`.
- Float values x and y should be determined by whether y falls within a certain tolerance of x.
- Alternative use `std::weak_ordering` 

```c++
#include <iostream>
#include <cmath>
#include <compare>

std::weak_ordering compare_doubles(double a, double b) {
    if (std::isnan(a) || std::isnan(b)) {
        if (std::isnan(a) && std::isnan(b)) {
            return std::weakly_equal; // Consider NaN == NaN for weak ordering
        } else if (std::isnan(a)) {
            return std::weakly_greater; // Convention: NaN is "greater" than non-NaN
        } else {
            return std::weakly_less;
        }
    } else if (a < b) {
        return std::weakly_less;
    } else if (a > b) {
        return std::weakly_greater;
    } else {
        return std::weakly_equal;
    }
}


// compare char not caring about case a==A
std::weak_ordering compare_chars_case_insensitive(char a, char b) {
	// convert uppercase to lowercase
    char lower_a = std::tolower(static_cast<unsigned char>(a));
    char lower_b = std::tolower(static_cast<unsigned char>(b));

    if (lower_a < lower_b) {
        return std::weakly_less;
    } else if (lower_a > lower_b) {
        return std::weakly_greater;
    } else {
        return std::weakly_equal;
    }
}
```


# Move semantics
Transfer ownership of an object, as opposed to copying it, thus avoiding a potentially nontrivial performance hit due to object copy.
- Reference - [Engineering Distinguished Speaker Series: Howard Hinnant](https://www.youtube.com/watch?v=vLinb2fgkHk)

Example, passing a vector/string as a parameter :
- Passing by reference allows for fast access but the object doesn't own it but is unsafe as it can be altered externally
- To own before C++11, required deep copying the data improving safety at cost of performance

## Rule of 3
If any one of these 3 special functions is defined, include definitions for the remaining 2. 
- This can include just declaring them as `default` or `delete` where appropriate, and leaving generation of the implementations to the compiler.

Special functions:
1. copy constructor
2. copy assignment operator 
3. destructor

## Rule of 5
If any one of these five special functions is defined, include definitions for the remaining four. 
- This can include just declaring them as `default` or `delete` where appropriate, and leaving generation of the implementations to the compiler.
- For <font color="7cfc00">C++11 and higher</font>.
- Reference - [The Rule of Zero, Five, or Six](https://www.modernescpp.com/index.php/c-core-guidelines-constructors-assignments-and-desctructors/)

Special functions:
1. copy constructor
2. copy assignment operator 
3. move constructor
4. move assignment operator 
5. destructor

<font color=#cd76ea>Corollary :</font>
1. [Rule of Zero](https://www.fluentcpp.com/2019/04/23/the-rule-of-zero-zero-constructor-zero-calorie/) - If you can avoid defining the five special class functions, do 
2. If you `default` or `delete` any copy, move, or destructor function, `default` or `delete` them all.
## Copy constructor
#lvalue_reference
This deep copies an object to another creating a new one in the process and retaining the original
- It was added in <font color=#7cfc00>C++03</font>.
```c++

class Obj{
	int id;
public:
	Obj(int val):id(val){}
	int getId() return id;
};

// in main()
Obj exe1{100} 
Obj exe2{exe1} // copy constructor
```

## Copy operator
```c++
class Exe{
	int x;
public:
	Exe(int x):x(x){}
	Exe* clone(){
		return new Exe(*this); // copy constructor 
	}
	~Exe() = default;
}

class Obj {  
    double val;  
    Exe* exePtr;
public:
	Obj(const Exe& ref, double x):val(x), exePtr(x.clone()){}

	// copy operator
	Obj operator =(const Obj& src){
		if (this != &src) { // Self-assignment check
	        delete exePtr;
	        exePtr = src.exePtr->clone();
	        val = val.expirationTime;
	    }
	    return *this;
	}
};

// usage
Exe

```

### Copy and Swap

Implementation of the copy assignment operator using the **"copy and swap"** idiom, in conjunction with your provided `swap` function, is often recommended option, reasons:
1. **Exception Safety (Strong Guarantee):** The copy and swap idiom provides a strong exception safety guarantee. If an exception occurs during the creation of the copy (`OptionInfo src = *this;`), the original object (`*this`) remains in its original, valid state. This is because the assignment only happens after the copy is successfully constructed.

2. **Self-Assignment Safety:** This implementation inherently handles self-assignment (`option = option;`) correctly. When you pass `*this` by value into the `src` parameter, a copy is made. If it's a self-assignment, you're essentially swapping the object with its own copy, which has no ill effects.

3. **Code Reusability and Clarity:** It leverages the `swap` function, which encapsulates the logic for exchanging the member variables. This makes the copy assignment operator concise and easier to understand.

4. **Reduced Code Duplication:** The core logic of copying resources is handled by the copy constructor (implicitly called when passing by value), and the swapping logic is in the `swap` member function. This reduces code duplication compared to a traditional copy assignment operator.

```c++
#include <utility>

class Obj {  
    double val;  
    Exe* exePtr;
public:
	Obj(const Exe& ref, double x):val(x), exePtr(x.clone()){}

	// swap
	void swap(Obj& src) noexcept{
		using std::swap;
		swap(val, src.val);
		swap(exePtr, src.exePtr);
	}

	// copy operator
	Obj operator=(Obj src) { 
		swap(src); 
		return *this; 
	}
};
```

**How it works step-by-step:**

1. **`OptionInfo& OptionInfo::operator=(OptionInfo src)`:**
    - The parameter `src` is passed **by value**. This is the crucial part. When the copy assignment operator is called, the compiler creates a _copy_ of the right-hand side object (`rhs` in a typical `operator=`) and names it `src`. This copy is made using the **copy constructor** of `OptionInfo`.

2. **`swap(src);`:**
    - Inside the assignment operator, you call the `swap` member function, exchanging the members of the current object (`*this`) with the members of the newly created copy `src`.

3. **`return *this;`:**
    - Finally, you return a reference to the modified current object, as is standard for assignment operators.

## Disable Copy
The [delete](https://en.cppreference.com/w/cpp/keyword/delete) keyword can instead be utilized to explicitly disable copy operations, both externally and internally
- It was added in <font color=#7cfc00>C++11</font>.
```c++
class Resource{
	int val;
public:
	Resource(int x):val(x){}
	~Resource() = default;
	
	// Copy operations disabled both externally and internally:
	Resource(const Resource& src) = delete; 
	Resource& operator =(const Resource& src) = delete
}
```

## Move constructor
#rvalue_reference
In this case we won’t need the original object after the duplicate is created, we can more efficiently move the original to the target instead of copying
- eliminate the performance penalty of object copy
- It was added in <font color=#7cfc00>C++11</font>.
- found in `#include <utility>`

```c++
#include <utitlity>
	...
// in main()
Obj exe1{100} 

Obj exe3{std::move(exe1)}   // move constructor
Obj exe4 = std::move(exec2) // move constructor
```

The move constructor can be **either user-defined or implicitly generated by the C++ compiler** under certain conditions.

### Compiler-Generated Move Constructor:
The C++ compiler will implicitly generate a move constructor for a class if **all** of the following conditions are met:

1. **No user-declared copy constructor:** You haven't defined your own copy constructor.
2. **No user-declared move constructor:** You haven't defined your own move constructor.
3. **No user-declared copy assignment operator:** You haven't defined your own copy assignment operator.
4. **No user-declared move assignment operator:** You haven't defined your own move assignment operator.
5. **No user-declared destructor:** You haven't defined your own destructor.

If all these conditions are true, the compiler-generated move constructor will perform a **member-wise move** of the non-static data members of the class. This means:

- For members of built-in types (like `int`, `float`, pointers), a bitwise copy occurs.
- For members that are objects of class types, the move constructor of that member class will be called (if it exists and is accessible). If the member class doesn't have a move constructor, its copy constructor will be called instead.

### User-Defined Move Constructor:

You would typically define your own move constructor in situations where the default member-wise move semantics are not sufficient or correct, especially when your class manages resources like dynamically allocated memory.
1. Transferring ownership of pointers or handles.
2. Setting the pointers or handles of the source object to a safe state (e.g., `nullptr`) to prevent double deletion.
```c++
#include <iostream>
#include <vector>

class MyVector {
public:
    int* data;
    size_t size;

    MyVector(size_t sz) : size(sz), data(new int[sz]) {
        std::cout << "Constructed with size: " << size << std::endl;
    }

    ~MyVector() {
        delete[] data;
        std::cout << "Destructed (size: " << size << ")" << std::endl;
    }

    MyVector(MyVector&& other) noexcept : data(other.data), size(other.size) {
        std::cout << "Move constructed (from size: " << other.size << ")" << std::endl;
        other.data = nullptr;
        other.size = 0;
    }
};

int main() {
    MyVector v1(10);
    MyVector v2 = std::move(v1); // Move from v1 to v2

    std::cout << "Size of v1 after move: " << v1.size << std::endl;

    return 0; // Destructors of v1 and v2 will be called safely
}
```

### Uses
#### 1. A class as a class variable: 

```c++
class Enclose { 
	SimpleClass sc_; 
public: 
	Enclose(const SimpleClass& sc); // Copy Constructor 
	Enclose(SimpleClass&& sc); // Move Constructor
	SimpleClass get_sc() const; // Accessor 
}; 

// Implementation: 
Enclose::Enclose(const SimpleClass& sc) :sc_{sc} {} // Constructor 1 
Enclose::Enclose(SimpleClass&& sc) :sc_{std::move(sc)} {} // Constructor 2 

SimpleClass Enclose::get_sc() const { // Accessor 
	return sc_; 
}
```

Alternative with 1 constructor **PREFERED** :
```c++
class EncloseSingleConstructor { 
	SimpleClass sc_; 
public: 
	EncloseSingleConstructor(SimpleClass sc); 
	SimpleClass get_sc() const; 
}; 

// Constructor implementation: 
EncloseSingleConstructor::EncloseSingleConstructor(SimpleClass sc) : sc_{std::move(sc)}{}

```

#### 2. Passing a data structure as a parameter
```c++
PatternAnalytics::PatternAnalytics(std::vector data_in /*, . . .*/) : data{std::move(data_in)} {}
```

#### 3. Anonymous Temporary Objects
If we will not need to reuse the object being input, we could call this function with an anonymous temporary #rvalue argument

```c++
void r_val_ref(SimpleClass&& sc) { 
	cout << "r_val_ref (&&): k = " << sc.get_val() << "\n"; 
}


// Use;
r_val_ref(SimpleClass{20}); // rvalue argument
```

<font color=#ff0800><strong>Warning</strong></font> : don't use move semantics for return value i.e.:
```c++
// Don't do this: 
SimpleClass f(int n) { 
	return std::move(SimpleClass{n}); 
} 
// or this: 
SimpleClass g(int n) { 
	SimpleClass sc{n}; 
	return std::move(sc); 
}
```

It is not good practice and can result in suboptimal results. Instead, we should simply return the object as usual, as <font color=#ffcba4>guaranteed copy elision</font> .

```c++
SimpleClass f(int n){
    return {n}; // Same as 'return SimpleClass{n}',
                // using uniform initialization and
                // automatic type deduction, introduced in C++11.
}
SimpleClass g(int n){
    SimpleClass sc{n};
    return sc;
}
```

**Resource** : [Return value optimizations - Fluent C++](https://www.fluentcpp.com/2016/11/28/return-value-optimizations/)


### Disable
```c++
class Resource{
	int val;
public:
	Resource(int x):val(x){}
	~Resource() = default;
	
	// Move operations disabled both externally and internally:
	Resource(const Resource&& src) = delete; 
	Resource& operator =(const Resource&& src) = delete
}
```


## Move assignment operator

A special member function that's called when you assign one object to another using the `=` operator, and the source object is an #rvalue : _typically temporary object_
- Instead of copying the data operator <font color=#ffcba4>transfers the ownership</font> of the source's resources to the destination object.
- Sets the temporary object's pointer to `nullptr` to prevent it from deleting the memory when it goes out of scope
- Avoids the overhead of a deep copy
- Optimizing resource management, especially when dealing with objects that hold significant amounts of data.

### Syntax
```c++
MyClass& operator=(MyClass&& other) noexcept {
    if (this != &other) {
        // 1. Release any resources held by the current object
        delete[] data_;
        data_ = nullptr;
        size_ = 0;

        // 2. "Steal" the resources from the other object
        data_ = other.data_;
        size_ = other.size_;

        // 3. Set the other object's resources to a safe state
        other.data_ = nullptr;
        other.size_ = 0;
    }
    return *this;
}
```

## Example
```c++
#include <iostream>
#include <vector>

class DataHolder {
	int* data_;
    size_t size_;
public:
    DataHolder() : data_(nullptr), size_(0) {
        std::cout << "Default constructor called" << std::endl;
    }

    DataHolder(size_t size) : size_(size), data_(new int[size]) {
        std::cout << "Parameterized constructor called" << std::endl;
        for (size_t i = 0; i < size_; ++i) {
            data_[i] = i;
        }
    }

    DataHolder(const DataHolder& other) : size_(other.size_), data_(new int[size_]) {
        std::cout << "Copy constructor called" << std::endl;
        for (size_t i = 0; i < size_; ++i) {
            data_[i] = other.data_[i];
        }
    }

    DataHolder(DataHolder&& other) noexcept : data_(other.data_), size_(other.size_) {
        std::cout << "Move constructor called" << std::endl;
        other.data_ = nullptr;
        other.size_ = 0;
    }

    DataHolder& operator=(const DataHolder& other) {
        std::cout << "Copy assignment called" << std::endl;
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new int[size_];
            for (size_t i = 0; i < size_; ++i) {
                data_[i] = other.data_[i];
            }
        }
        return *this;
    }

    DataHolder& operator=(DataHolder&& other) noexcept {
        std::cout << "Move assignment called" << std::endl;
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }

    ~DataHolder() {
        std::cout << "Destructor called" << std::endl;
        delete[] data_;
    }

    void print() const {
        std::cout << "Size: " << size_ << ", Data: [";
        for (size_t i = 0; i < size_; ++i) {
            std::cout << data_[i] << (i == size_ - 1 ? "" : ", ");
        }
        std::cout << "]" << std::endl;
    }
};

DataHolder create_holder(size_t size) {
    return DataHolder(size); // Returns a temporary object (rvalue)
}

int main() {
    DataHolder h1(5);
    DataHolder h2;

    std::cout << "Assigning temporary object to h2:" << std::endl;
    h2 = create_holder(10); // Move assignment will likely be called

    std::cout << "h1: ";
    h1.print();
    std::cout << "h2: ";
    h2.print();

    DataHolder h3;
    std::cout << "Moving h1 to h3:" << std::endl;
    h3 = std::move(h1); // Explicitly using std::move to invoke move assignment

    std::cout << "h1 (after move): ";
    h1.print(); // h1's data will likely be null
    std::cout << "h3: ";
    h3.print();

    return 0;
}
```

### Explanation
When `h2 = create_holder(10);` is executed, the move assignment operator of `DataHolder` will be called (if implemented), efficiently transferring the ownership.

Process breakdown:
- **`create_holder(10)` is called:**
    - Inside the `create_holder` function, the `DataHolder(size_t size)` constructor is indeed called with `size = 10`.
    - This constructor allocates memory for 10 integers, initializes them, and creates a `DataHolder` object on the function's stack (or wherever the compiler deems appropriate for return value optimization).

- **Returning the object:**
    - The `return DataHolder(size);` statement in `create_holder` signals that this newly created `DataHolder` object needs to be returned to the calling function (`main`). This is where the magic of move semantics can happen.

- **Assignment in `main`:**
    - In `main`, you have an _already constructed_ `DataHolder` object named `h2` (it was default-constructed earlier).
    - The line `h2 = create_holder(10);` attempts to _assign_ the temporary `DataHolder` object returned by `create_holder(10)` to the existing object `h2`.

If you hadn't defined a move assignment operator in your `DataHolder` class, the compiler would have fallen back to using the **copy assignment operator** (the one taking a `const DataHolder&`). 

This would involve allocating new memory within `h2` and copying the 10 integers from the temporary object, which is less efficient.


# Lambda expression
#lambda
`Anonymous function` - defines a function within the body of another function
- Can be passed as arguments
- C++ version >> <font color="7cfc00">C++11 and higher</font>
- Resource : [C++ Lambda](https://www.programiz.com/cpp-programming/lambda-expression)
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

Accessing class variables in the lambda function:
```c++
Example::fx(){
	...
	// ax^2 + bx + c
	auto quadratic_value = [this](double x) { 
		return x * (a*x + b) + c; 
	};
	...
}
```

In certain thread-based situations, it is possible that instead of the this pointer
- a copy of the active object, `*this`, might be necessary as the lambda capture argument. 
- C++ version >> <font color="7cfc00">C++17 and higher</font>
```c++
Example::fx(){
	...
	// ax^2 + bx + c
	auto quadratic_value = [*this](double x) { 
		return x * (a*x + b) + c; 
	};
	...
}
```
# Enumerated Constants
#Cpp #enum
Efficient way of passing codes compared to using strings
- C++ version >> <font color="7cfc00">C++11 and higher</font>

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


# Standard Template Library
#STL 
A subset of C++’s Standard Library that houses a set of container classes
- Provides a group of algorithms applicable to those containers and other containers that follow the same coding conventions.
- Reference - [C++ in 2005](https://www.stroustrup.com/DnE2005.pdf)

STL containers are powerful due to their relationship with STL *algorithms*.
Algorithms are expressed in terms abstraction called <font color=#eab676><strong>iterators</strong></font>, rather than on containers:
- <font color=#30D5C8><strong>Containers :</strong></font> define how data is organized. 
- <font color=#30D5C8><strong>Iterators :</strong></font> describe how organized data can be traversed. 
- <font color=#30D5C8><strong>Algorithms :</strong></font> describe what you can do on the data.

## Templates
#template #generic 
 Blueprints for functions and classes
 - Facilitate generic programming in C++
 - Primarily written in header files
 - Potentially improving runtime performance.

<font color=#eab676><strong>Specialization</strong></font> - compiler generating specific code for the Types using the template.
- saving the trouble of manually writing that code for each Type.

<font color=#30D5C8><strong>Function :</strong></font>
```c++
/* Multiplication example */
// integers:
 int int_square(int k)
    return k * k;
    
 // real numbers (double):
 double dbl_square(double x)
    return x * x;

 // Obj:
 Obj obj_square(const Obj& f)
    return f * f;    // operator* on Obj is defined

/**** Alternative ****/
template <typename T>
 T tmpl_square(const T& t){
    return t * t;
 }

/******* Use *******/ 
// Primitive types
int sq_int = tmpl_square(4); // tmpl_square(const int&) 
double sq_real = tmpl_square(4.2); // tmpl_square(const double&)

// Objects
Obj frac{2, 3}; 
Obj sq_frac = tmpl_square(frac);
// uniform initialization c++
auto sq_frac_unif = tmpl_square<Obj>({2, 3});

```

<font color=#30D5C8><strong>Class :</strong></font>
```c++
/* Class Declaration */
template <typename T>
export class MyPair{
	T a, b;
public:
    MyPair(const T &first, const T &second) :a(first), b(second) {}
    T get_min() const;
 };


/* Implementation */
template <typename T>
MyPair<T>::MyPair(T first, T second) :a(first), a(second) {}

template <typename T>
T MyPair<T>::get_min() const{
	return a < b ? a : b; // Assuming operator < is defined for type T,
}

/******* Use *******/ 
MyPair<int> mp_int{10, 26};
int min_int = mp_int.get_min();             // OK, returns 10

MyPair<Obj> mp_frac{{3, 2}, {5, 11}};
Obj min_frac = mp_frac.get_min();      //  Returns 5/11
```

Default template parameters:
- can omit the explicit type name from inside the angle brackets
```c++
template <typename T = double>
export class MyPair{
	T a, b;
public:
    MyPair(const T &first, const T &second) :a(first), b(second) {}
    T get_min() const;
 };

/******* Use *******/ 
MyPair<> mp{10.87, 26.243};
int min = mp.get_min();             // OK, returns 10.87
```


Multiple parameters:
-  example  `std::map<K, V> `
```c++
template <typename T, typename U>
export class MyPair{
	T a;
	U b;
public:
    MyPair(const T &first, const U &second) :a(first), b(second) {}
 };

/******* Use *******/ 
MyPair<double, int> mp{10.87, 26};
int min = mp.get_min();             // OK, returns 10.87
```

<font color=#fd6206><strong>Footnote :</strong></font>   We could have expressed the body of`MyPair<T>::get_min()` as
```c++
T retval;                 // 1
retval = a < b? a : b;    // 2
return retval;
```
However, this would have require T to have :
- a copy assignment operator `2`
- a default constructor `1`

When writing generic code:
1. Consider what you ask of the types for which the code will be instantiated 
2. Strive to ask only for what is required, nothing more. 

*This widens the set of types for which the code will be applicable*

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


# Assert
#error #debug #error_handling
Checks if your assumptions about the program's state are true during runtime and will halt execution if is violated.
- Found in the `<cassert>` header (or `<assert.h>` for C-style compatibility)
- Reference: [assert - cppreference.com](https://en.cppreference.com/w/cpp/error/assert)\

```c++
#include <cassert>
#include <iostream>

int main() {
    int age = 25;
    assert(age >= 0); // Assumption: age should always be non-negative

    int* ptr = new int[10];
    assert(ptr != nullptr); // Assumption: memory allocation should succeed

    int index = 5;
    assert(index >= 0 && index < 10); // Assumption: index is within bounds
    ptr[index] = 42;

    delete[] ptr;
    ptr = nullptr;
    assert(ptr == nullptr); // Assumption: pointer should be null after deletion

    std::cout << "Program continues after assertions (if they pass)." << std::endl;
    return 0;
}
```


## NDEBUG
Key to `assert`'s behaviour in different build configurations lies in a pre-processor macro called `NDEBUG` (which stands for <font color=#30D5C8>No Debug</font>).
- **Debug Builds (default during development):** When `NDEBUG` is **not defined**, the `assert` macro is active. It performs the Boolean evaluation and the error reporting/termination if the condition is false.
- **Release Builds (Typically when you compile for deployment):** When you define the `NDEBUG` macro (usually by adding a compiler flag like `-DNDEBUG` during compilation), 
	- The `assert` macro is effectively disabled. 
	- The pre-processor replaces all `assert(condition);` statements with nothing. 
	- This means that the conditions are not evaluated, and there's no performance overhead from the `assert` calls in your release version.

<font color=#ff0800><strong>NOTE : </strong></font> **Don't use `assert` for error handling that you expect in production:** 
- `assert` is for catching bugs during development. 
- For situations where errors are a normal part of the program's operation (e.g., invalid user input, network failures), use proper error handling mechanisms like exceptions or return codes.

# Exceptions
Reference - [Exception Handling - The C++ Programming Language, 4th Edition](https://www.oreilly.com/library/view/the-c-programming/9780133522884/ch13.xhtml#ch13lev1sec2)
Throwing an exception:
```c++
void fx(){
	//...
	throw std::runtime_error{"Error: stop!!"}
}
```
