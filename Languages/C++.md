#Cpp 

# Resources
#research #documentation
- [cppreference.com](https://en.cppreference.com/w/)
- [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)

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

*This widens the set of types for which the code will be applicable* : 


<font color=#30D5C8><strong>Compile-time integral values :</strong></font>
- Such as with the fixed-size sequential STL container
```c++
std::array<T, N>  // N = positive integer
```

Templates can even accept templates as parameters - `expression templates`

## Containers
Resource - [Back to Basics: Iterators in C++ - Nicolai Josuttis - CppCon 2023](https://www.youtube.com/watch?v=26aW6aBVpk0)

| Member function | Description                                                                     |
| --------------- | ------------------------------------------------------------------------------- |
| `begin()`       | Returns an iterator that points to the first element<br>                        |
| `end()`         | Returns an iterator that points to the address after the last element<br>       |
| `cbegin()`      | Returns a `const` iterator that points to the position of the first element<br> |
| `cend()`        | Returns a `const` iterator that points to the adress after the last element<br> |
| `size()`        | Returns the number of elements in a container                                   |
| `empty()`       | Returns true if a container is empty<br>                                        |
| `clear()`       | Clears all elements from a container                                            |
<font color=#fd6206><strong>Footnote :</strong></font> Member functions that apply to all STL containers

Containers can be divided into at least two categories:
1. Sequential
2. Associative
### Sequential containers
These emphasize sequential traversal of the elements.

#### `vector<T>` 
#vector
- dynamic array guaranteed to be allocated in contiguous memory
- All heap memory allocation, management, and replacement is handled internally 
- efficient <font color=#ffcba4>appending and deletion</font> of data to the <font color=#ffcba4>back of the container.</font>
- efficient <font color=#ffcba4>traversal</font> of data <font color=#ffcba4>sequentially.</font>
- <font color=#cd76ea>When in doubt, use a vector</font>

<font color=#eab676><strong>Initialization {.} vs (.):</strong></font> 
`{.}` - inserts an entry/initialization
`(.)` - define the size of the vector
Reference -  [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Variable_and_Array_Initialization)
```C++
std::vector<int> v1{10};    // size() === 1,  v[0] = 10
std::vector<int> v2(10);    // size() === 10

std::vector<int> x1(100, 1);  // A vector containing 100 items: All 1s.
std::vector<int> x2{100, 1};  // A vector containing 2 items: 100 and 1.
```

<font color=#eab676><strong>.data() :</strong></font>
- For <font color="7cfc00">C++11 and higher</font> 
- Returns the memory address of its first element.

<font color=#eab676><strong>.push_back(.) :</strong></font>
-  Adds to the end of the vector.
- Use the inefficient object copy constructor.
	- If copy constructor is set to `delete` code will not compile.
- For larger object elements, the performance hit could add up
```c++
vector bank;

BankAccount ba01{1000.00, 0.021}; 
bank.push_back(ba01);
```
- If the object is no longer need outside the vector use `std::move(.)`
```c++
#include <utility>
vector bank;

BankAccount ba01{1000.00, 0.021}; 
bank.push_back(std::move(ba01));
```

<font color=#eab676><strong>.emplace_back(.) :</strong></font> 
- For <font color="7cfc00">C++11 and higher</font> 
- Resolved this problem from `.push_back(.)`
- Pass the arguments to build the object without constructing it.
	- Saving one construction (and one destruction) with every call
	- mostly useful if the object to add to the container has not been constructed yet
```c++
vector bank;

bank.emplace_back(1000.00, 0.021);
```

<font color=#eab676><strong>Polymorphism :</strong></font> 
When a collection of derived objects need to be determined dynamically, in a vector
- can be accomplished by defining a vector with a `std::unique_ptr` template parameter
- Point to objects of derived classes via the (often abstract) base class type
- push back each pointer onto the vector
- When the vector goes out of scope, each `unique_ptr` cleans up after itself.
```c++
vector<std::unique_ptr<Payoff>> payoffs;

payoffs.push_back(std::make_unique<CallPayoff>(100.0));
payoffs.push_back(std::make_unique<PutPayoff>(100.0));
```

<font color=#eab676><strong>.size() :</strong></font> 
- The current number of elements stored in the container.

<font color=#eab676><strong>.capacity() :</strong></font> 
- The actual size of objects that can be stored before resizing

<font color=#fd6206><strong>Allocation and contiguous memory.</strong></font>
- Vector models a dynamic array that can grow when it is full
- Insertion cost can be higher when at container’s capacity
	- Thus contents have to be copied or moved to a new and bigger storage location
- Can reduce the costs of these potentially costly moments, vector offers functions that let the programmer’s code decide when to change the size/capacity:

If you know at least approximately how many objects will be stored in a container, you can call these functions at selected (noncritical) moments in a program to ensure that enough space will be available

<font color=#ffcba4><strong>Note :</strong></font> during resize/reallocation, vector calls `move constructor` only if marked as <font color=#30D5C8><strong>noexcept</strong></font>, otherwise it will call each object’s `copy constructor`. 
- `move` operations are faster (often significantly faster) than `copy`.

<font color=#eab676><strong>.reserve() :</strong></font> 
- changes vector capacity but not its size (number of elements).
```c++
 std::vector <MyClass> u;
 u.reserve(1'000);   // The apostrophe used as a proxy for a comma
                     // was added to the Standard in C++14
                     // creates a vector of 1000 capacity but size 0
```

<font color=#eab676><strong>.resize() :</strong></font> 
-  Changes both the size (number of elements) and the capacity by conceptually adding default values at the end 
	- `0` for fundamental types such as `int` or `double`
	- `nullptr` for pointers
	- `default-constructed objects` for classes with such a constructor).

<font color=#eab676><strong>.clear() :</strong></font> 
Destroy each object stored in the container and reset its size to 0, capacity remains the same.


<font color=#eab676><strong>.front(), .back(), pop_back() :</strong></font> 
- 1st two return a reference of the objects in the position
- `pop_back()` removes the objects from the vector and returns it

<font color=#eab676><strong>.at(.) vs [.] :</strong></font> 
Both used to access at specific indexes
`.at(.)` -  provides bounds checking by *throwing an exception* for invalid indexes
- For an error message:
```c++
#include <stdexcept>    // for exception class std::out_of_range

vector<int> v {0, 1, 2, 3, 4, 5};     // v.size() === 5
try{
	int n = v.at(10);   // at(.) throws out_of_range exception
}
catch (const std::out_of_range& e){
	std::cout << e.what() << "\n\n";
}
```

<font color=#ff0800><strong>Note :</strong></font> if you believe an index could end up out of bounds, you can consider using `at(.)` instead of `[.]`.
- checking the bounds on every access has a cost, when only when there is a doubt with respect to the validity


#### `deque<T>`
- efficient <font color=#ffcba4>appending and deletion</font> of data from the <font color=#ffcba4>front of the container.</font>
- storage is not guaranteed to be in contiguous memory
- traversal of that container is generally less efficient as `vector`

 <font color=#eab676><strong>push_front() & push_back() :</strong></font> 
-Appends an element to the front/back of a deque
 
 <font color=#eab676><strong>pop_front() & pop_back() :</strong></font> 
Removes the first element in the front/back a deque

```c++
 std::deque<int> on_deque{0, 1, 2, 3};
 
// Push new elements onto the front:
on_deque.push_front(-1);
on_deque.push_front(-2);
on_deque.push_back(4);
// on_deque now contains: -2 -1 0 1 2 3 4

// Remove both first and last element:
 on_deque.pop_front();
 on_deque.pop_back();
 // on_deque now contains: -1 0 1 2 3
```

<font color=#eab676><strong>emplace_front(.) & emplace_back(.) :</strong></font> 
- Construct the object in position instead of copying.
```c++
std::deque recs; 
recs.emplace_back(3.0, 2.0); 
recs.emplace_front(4.0, 3.0);
```

<font color=#eab676><strong>.at(.) vs [.] :</strong></font> 
- Similar to `std::vector<T>`


#### `list<T>`
- Doubly linked list of nodes.
- efficient <font color=#ffcba4>appending and deletion</font> of data from the <font color=#ffcba4>within the container.</font>
- does not provide random access via the `[.]` operator or an `at(.)`
- The only way to reach element `i` in a list is to iterate through the first `i elements`
- Uses <font color=#ff2800>higher memory allocation</font>
- <font color=#ff2800>Slower access</font> due to need of iteration
- doesn't have <font color=#eab676><strong>.at(.) or [.] :</strong></font> 

<font color=#eab676><strong>emplace_front(.) & emplace_back(.) :</strong></font> 
- Construct the object in position instead of copying.

 <font color=#eab676><strong>push_front() & push_back() :</strong></font> 
-Appends an element to the front/back of a deque
 
 <font color=#eab676><strong>pop_front() & pop_back() :</strong></font> 
Removes the first element in the front/back a deque


#### `array<T, N>`
- A fixed-size array of `N` elements.
- For <font color="7cfc00">C++11 and higher</font> 
- value of `N` must be explicitly known at compile time.
- stored contiguously and are not dynamically allocated
- most efficient <font color=#ffcba4>traversal</font> of data <font color=#ffcba4>sequentially</font> for static sized data.
- It's explicit fixed-size declaration requirement at compile time makes it unsuitable for dynamically sized data
- Good alternative for <font color=#30D5C8>raw/c-type arrays</font> 

Supports:
- `size()`, `at(.)`, `front()`, `back()`, and `empty()`, and the `[.]` operator
Doesn't support:
- `push_back(.)`, `push_front(.)`, `pop_front(.)`, or `pop_back(.)` as it's size is constant

```c++
int arr_01[]{0, 1, 2, 3};      // raw array
std::array<int, 4> arr_02{0, 1, 2, 3};

/*** C++ 17 & higher ***/
std::array arr_03{0, 1, 2, 3}; // class template argument deduction {CTAD}
```

### Associative containers
These emphasize organizing the storage in ways that make it easy to retrieve values inserted data.

#### `set<T>`
Stores unique elements of type `T`
- <font color=#ffcba4>Reorders / sort</font> them whenever a new element is inserted.
```c++
#include <set>

std::set<int> some_set{5, 1, 2, 3, 4, 3};  // 1 2 3 4 5
```


#### `multiset<T>`
 - Set that accepts multiple occurrences of the same value.
	- <font color=#ffcba4>Reorders / sort</font> them whenever a new element is inserted.
```c++
#include <set>

std::multiset<int> some_multiset{5, 1, 2, 3, 4, 3};  // 1 2 3 3 4 5
```

#### `map<K, T>`
Stores `key-value` pairs of elements of types `K` and `T`
- Ensures that each key is unique but can have the same values.
- <font color=#ffcba4>Reorders / sort</font> the keys whenever a new element is inserted.
- uses `std::less` for comparison, which results in ascending order.

```c++
#include <map>

std::map<std::string, int> some_map{
    {"five", 5}, {"one", 1}, {"two", 2}
};

// Display map:
for (const auto& pr : some_map){
    cout << format("{} : {}\n", pr.first, pr.second);
}

// Better
for (auto [k, v] : some_map){
	cout << format("{} : {}\n", k, v);
}
```

<font color=#30D5C8><strong>Custom objects :</strong></font>
Use of `std::map` to store custom objects will result in <font color=#ff2800><strong>compiler error</strong></font> if :
- No ordering is defined in the class
- Use a type not supporting inequality operators `<` 


#### `multimap<K, T>`
`std::map` that accepts multiple pairs with the same key.
- <font color=#ffcba4>Reorders / sort</font> the keys whenever a new element is inserted.
```c++
#include <map>

std::multimap<std::string, int> some_multimap{
    {"five", 5}, {"one", 1}, {"two", 2}, {"five", -5}, {"one", -1}
};
```


#### unordered
Similar function to their counter parts:
- They <font color=#ffcba4>don't sort</font> the keys whenever a new element is inserted.
- Makes <font color=#ffcba4>searches faster</font> than their ordered counterparts.
- <font color=#ff0800>Consume more space</font> in memory
- For <font color="7cfc00">C++11 and higher</font> 

• `std::unordered_set<T>`
• `std::unordered_multiset<T>`
• `std::unordered_map<K, T>`
• `std::unordered_multimap<K, T>`


## Iterators
Objects that can iterate over elements of a sequence.
- Writing algorithms on iterators rather than writing them on containers makes algorithms more general and typically more useful.

Given an iterator `iter`, you can move to next element in sequence by:
- pre-increment operator: `++iter` (<font color=#fd6206>preferable</font>)
- post-increment operator: `iter++`

**Categories :**
### Output iterator
A single-pass tool that can be used for : 
1. Writing data to an output stream, such as with `cout`
2. Inserting data into a new container, such as at the end of a `vector`.
 
### Input iterator
A single-pass tool that can be used for :
1. Tasks like consuming data from a stream (including such fleeting things as keyboard input). 
- An iterator can often both write and read data, which would make it both an output and input iterator. 

### Forward iterator 
Similar to an` input iterator`, but :
- Allows making more than one pass over the same sequence (<font color=#fd6206>if you don’t alter the elements along the way</font>)
- Can only  go forward one element at a time.

### Bidirectional iterator
Similar to a `forward iterator`, but :
- Lets you go backward one element at a time. (<font color=#fd6206>e.g. iterators on a list - 'doubly linked list'</font>).

### Random-access iterator
Similar to a `bidirectional iterator`, but : 
- lets you go forward or backward `n` elements at a time efficiently.

### Contiguous iterator 
A `random-access iterator` with the added requirement that the <font color=#30D5C8>sequence is placed contiguously in memory</font> (<font color=#fd6206>e.g. iterators on a vector / an array, but not a deque</font>).


### Usage
<font color=#30D5C8>Example :</font>
**Defining an iterator (contiguous) on a vector**
1. Create the `vector` & `iterator` :
```c++
 // Create the iterator:
 vector<int>::iterator pos;        // "position"
 
 // A vector of integer values:
 vector<int> v = {17, 28, 12, 33, 13, 10};
```

2. Set the `iterator` to the position of the first element, 
	- Using the `begin()` member function on a vector.
	- an iterator acts syntactically like a pointer.
	- The actual element in the first position is accessed by dereferencing the iterator.
```c++
pos = v.begin();            // Sets iterator to point at 1st position
                            // of the vector v
int first_elem = *pos;      // Dereferencing pos returns 17
```

<font color=#ff2800><strong>Note : </strong></font> **`Iterators` do not validate whether you go out of bounds**

The `end()` returns the address after the last element
- Range of a container : <font color=#eab676><strong>[begin, end)</strong></font>
- Container is empty if `v.begin() == v.end()` 
```c++
pos = v.end() - 1;    // access the last element *pos = 10
```

3. To reduce **verbosity**  of step `1 & 2`
	- Both the `begin()` and `end()` on a Standard Library container will return an iterator for the sequence defined.
```c++
auto pos = v.begin();  // creates vector<int>::iterator pos;  
```

- Avoid modification of elements of a container, use `constant iterator` :
	- Using `cbegin()` & `cend()`
```c++
vector w{17, 28, 12, 33, 13, 10}; 
auto cpos = w.cbegin(); // cpos = const iterator

++cpos; 
*cpos = 19; // Compiler error!
```

4. To advance to the next use increments `iter++` / `++iter` :
	- If `bidirectional` / `random-access` to move back `--iter`
```c++
++pos;
int x = *pos;    // *pos = 28

--pos;   // *pos = 17
```

5. Reassigning the value at a position :
```c++
*pos = 19;
```

6. If container allow `random-access` move the iterator by an amount using:
```c++
pos += 3  // *pos = 13
pos -= 2  // *pos = 33
```

7. **Iterator-based** for loops
	-  Used Prior to C++11
	- range-based for loops are preferred.
```c++
std::vector<int>  v{17, 28, 12, 33, 13, 10};

for (auto pos = v.begin(); pos != v.end(); ++pos){
	std::cout << *pos << " ";
}
```

<font color=#30D5C8>Example :</font>
**Defining an iterator on associative containers**
1. Creating a new `iterator`
```c++
std::map<int, double> map = {{5, 92.6}, {2, 42.4}, {1, 10.6}, {4, 3.58}, {3, 33.3}};

auto pos = map.begin()
```

2. Displaying the values;
	-  Remember map sorts the values by `key`
```c++
// Display 1st pair:
 cout << pos->first << ":  " << pos->second;  // 1: 10.6
```

3. Modification:
	- Keys are constant but values can be modified
```c++
++pos;
pos->second = 100.0
```



## Algorithms
Resources 
- [STL algorithm header - cppreference.com](https://en.cppreference.com/w/cpp/header/algorithm)
- [Algorithms library - cppreference.com](https://en.cppreference.com/w/cpp/algorithm/)
- [105 STL Algorithms in under an Hr - CppCon”](https://www.youtube.com/watch?v=2olsGf6JIkU)
- [New Algorithms in C++23 - cpponsea](https://youtu.be/uYFRnsMD9ks?si=1XMDbQiTNl5mFAvQ)
- [STL Algorithms in Action - CppCon](https://www.youtube.com/watch?v=eidEEmGLQcU)

Add to file using:
`#include <algorithm>`

![[Algorithms-map.png]]

From <font color=#7cfc00>C++ 17</font>, `STL algorithms` can be instructed to run in #parallel by simply setting an additional parameter in the algorithm argument.

![[Cpp17-land.png]]

### Count
<font color=#30D5C8>count :</font>
Counter number of occurrences that are equal to a value
```c++
#include <algorithm>
#include <vector>

int main(){}
	std::vector v{1, 2, 3, 4, 2, 5, 2, 6, 7};

	auto count_val_2 = std::count(v.begin(), v.end(), 2); // count_val_2 = 3
}
```


<font color=#30D5C8>count_if :</font>
Counter number of occurrences that meet a condition without for/while loop:
- traverse the sequence determined by the two iterators `v.begin()` & `v.end()`
- function will be applied to each element
- If function returns `true` count increments
- datatype is `unsigned`
```c++
#include <algorithm>
#include <vector>

bool condition(x) {      // Auxiliary function / Predicate
	return (x&1) == 1;
}

main(){}
	std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9};
	
	auto c = std::count_if(v.begin(), v.end(), condition);
	printf("Odd: %d\n", c);
	
	
	/*** Lambda alternative ***/
	auto c = std::count_if(v.begin(), v.end(), 
		[](int x){ return (x % 2) != 0; }); 
    printf("Count with lambda for odd numbers: %d\n", c);

	/*** Capture variables from its surrounding scope ***/
	int threshold = 5;
	auto c_lambda_capture = std::count_if(v.begin(), v.end(), 
		[threshold](int x){ return x > threshold;} );

	// init-capture
	auto num_within_interval = std::count_if(v.begin(), v.end(),
        [low = 3, high = 6](int n) {return low <= n && n <= high;});
}

```

associative containers: [count_if map - Stack Overflow](https://stackoverflow.com/questions/27837893/how-to-count-the-number-of-a-given-value-in-a-c-map)
```C++
#include <utility>        // std::pair
 // . . .
bool is_odd_value(const std::pair<unsigned, int>& x){
	return (x.second % 2) != 0;
}

std::unordered_map<unsigned, int> map{
{1, 9}, {2, 8}, {3, 7}, {4, 6}, {5, 5}, {6, 4}, {7, 3}, {8, 2}, {9, 1}};
 
auto num = std::count_if(map.begin(), map.end(), is_odd_value);

// Lambda
auto num = std::count_if(map.begin(), map.end(), 
	[map](std::pair<unsigned, int> const &p) {
        return p.second == value;} );
```



### Heap
<font color=#ffcba4><strong>Definition</strong></font> - binary tree structure that satisfies the heap properties:
1. Highest(or lowest) priority element is at the root
2. Children must be larger than parent ( _if lowest is at root_ ) and vice versa
Algorithms to manipulate data on the #heap 

[Convert](https://en.cppreference.com/w/cpp/algorithm/make_heap.html) container to a heap : `std::make_heap(a, b)`
1) The constructed heap is with respect to `operator <` (until C++20) `std::less{}`[since C++20](https://en.cppreference.com/w/cpp/utility/functional/less.html).
2) The constructed heap is with respect to `comp`.
	- [comparison function](https://en.cppreference.com/w/cpp/named_req/Compare.html) object which returns `true` if the first argument is _less_ than the second.
```c++
#include <algorithm>
#include <vector>

int main(){ 
    std::vector<int> v{3, 2, 4, 1, 5, 9}; 
    
    std::make_heap(v.begin(), v.end()); 
    // v = 9 5 4 3 2 1
}
```

[Insert](https://en.cppreference.com/w/cpp/algorithm/push_heap.html) into a heap : `std::push_heap(a, b)`
- Bubble up into position an inserted element
```c++
#include <algorithm>
#include <vector>
 
int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9};
 
    std::make_heap(v.begin(), v.end());
    // after make_heap: 9 5 4 1 1 3
 
    v.push_back(6);
    // after make_heap: 9 5 4 1 1 3 6
 
    std::push_heap(v.begin(), v.end());
    // after push_heap: 9 5 6 1 1 3 4
}
```

[Remove](https://en.cppreference.com/w/cpp/algorithm/pop_heap.html) root element : `std:pop_heap(a, b)`
- Swaps the value in the position `first` and the value in the position `last - 1` 
- Makes the subrange `[first, last - 1)` into a heap. 
- This has the effect of removing the first element from the [heap](https://en.cppreference.com/w/cpp/algorithm.html#Heap_operations "cpp/algorithm")
```c++
#include <algorithm>
#include <vector>
 
int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9};
 
    std::make_heap(v.begin(), v.end());
    // after make_heap: 9 5 4 1 1 3
     
    std::pop_heap(v.begin(), v.end()); 
	// after pop_heap:  5 3 4 1 1 9
 
    int largest = v.back();
    // largest element: 9
 
    v.pop_back(); 
	// after pop_back:  5 3 4 1 1
}
```


### Sort
1) Elements are [sorted](https://en.cppreference.com/w/cpp/algorithm.html#Requirements "cpp/algorithm") with respect to `operator <` (until C++20) `std::less{}`[since C++20](https://en.cppreference.com/w/cpp/utility/functional/less.html).
2) Elements are sorted with respect to `comp`.

[Sorts](https://en.cppreference.com/w/cpp/algorithm/sort_heap.html) a heap : `std::sort_heap`
- Converts the [heap](https://en.cppreference.com/w/cpp/algorithm.html#Heap_operations "cpp/algorithm") `[first, last )` into a sorted range. 
- The heap property is no longer maintained.
- Similar to calling `std::pop_heap` n-time
	1) The heap is with respect to `operator <` (until C++20) `std::less{}`[since C++20](https://en.cppreference.com/w/cpp/utility/functional/less.html).
	2) The heap is with respect to `comp`.
```c++
#include <algorithm>
#include <vector>
 
int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9};
    std::make_heap(v.begin(), v.end());
    // after make_heap: 9 5 4 1 1 3
    
    std::sort_heap(v.begin(), v.end());
    // after sort_heap, v: 1 1 3 4 5 9
```


[Partially sort](https://en.cppreference.com/w/cpp/algorithm/partial_sort) : `std::partial_sort(first, middle, last)`
- Rearranges elements such that the range `[first, middle)` contains the sorted `middle − first` smallest elements.
- The order of the remaining elements in the range `[middle, last)` is random.
- Add `std::greater{}` for descending order
```c++
#include <algorithm>
#include <array>

int main()
{
    std::array<int, 10> s{5, 7, 4, 2, 8, 6, 1, 9, 0, 3};
    
    std::partial_sort(s.begin(), s.begin() + 3, s.end());
    // |0 1 2 7| 8 6 5 9 4 3
    
    std::partial_sort(s.rbegin(), s.rbegin() + 4, s.rend());
    // 4 5 6 7 8 |9 3 2 1 0|
    
    std::partial_sort(s.rbegin(), s.rbegin() + 5, s.rend(), std::greater{});
    // 4 3 2 1 |0 5 6 7 8 9|
}
```

[n-th element](https://en.cppreference.com/w/cpp/algorithm/nth_element.html) - `std::nth_element(start, start + n, end)`
- Place only the `n-th` element in position in `ascending` order.
- Elements to the left will be smaller the `n-th` and right larger (*not sorted*)
-  Add `std::greater{}` for descending order
```c++
#include <algorithm>
#include <array>

int main(){
	std::vector<int> v{5, 10, 6, 4, 3, 2, 6, 7, 9, 3};
    printVec(v);
 
    auto n = v.begin() + v.size() / 2;
    std::nth_element(v.begin(), n, v.end());
    // v[v.size() / 2] 6
	// v = {3, 2, 3, 4, 5, 6, 10, 7, 9, 6};

	std::nth_element(v.begin(), v.begin() + 1, v.end(), std::greater{});
	// v[1] = 9
	// v = {10, 9, 6, 7, 6, 3, 5, 4, 3, 2};
}
```

- [std::stable_sort](https://en.cppreference.com/w/cpp/algorithm/stable_sort.html)
- [std::sort](https://en.cppreference.com/w/cpp/algorithm/sort.html)
- [std::inplace_merge - cppreference.com](https://en.cppreference.com/w/cpp/algorithm/inplace_merge.html) - merge sort.


### Search
<font color=#fd6206><strong>Note : </strong></font>  Returns the `.end()` iterator when no element is found

<font color=#ffcba4><strong>Linear Search</strong></font>
`std::find` :
Returns an `iterator` to the first occurrence of a value in the sequence. 
- Comparisons are made with operator `==` on the elements and the value you are trying to find
- Not suitable for floating-point values.
```c++
#include <iterator>        // for std::distance(.)

vector<int> ints{747, 377, 707, 757, 727, 787, 777, 717, 247, 737, 767};
int n = 757;

// No auxiliary function:
auto ipos = std::find(ints.begin(), ints.end(), n);

if (ipos != ints.end())
    cout << format("Found value {} at index {}\n",
        n, std::distance(ints.begin(), ipos));
//  Found value 757 at index 3
```
The [std::distance(start, stop)](https://en.cppreference.com/w/cpp/iterator/distance.html), declared in `<iterator>` 
- Returns the number of positions between `start` & `stop`.

`std::find_if` :
Returns an `iterator` to the first element for which a predicate yields true in the sequence.
```c++
vector<double> reals{0.5, 1.6, -2.3, 0.85, -3.2, 2.5, 1.8, -0.72};

 // Look for the first occurrence of a negative real value x in reals.
 // An auxiliary function is employed in the case of find_if:
 auto rpos = std::find_if(reals.begin(), reals.end(),
    [](double x) {return x < 0.0;});
 
 if (rpos != reals.end())
    std::cout << std::format("First negative value is {}\n", *rpos);
//  First negative value is -2.3
```

<font color=#ffcba4><strong>Binary Search</strong></font>
`std::binary_search` :
Returns an `iterator` to the first occurrence of a value in the sorted sequence. 
- Comparisons are made with operator `==` on the elements and the value you are trying to find
- Not suitable for floating-point values.
```c++
#include <iterator>        // for std::distance(.)

vector<int> ints{747, 377, 707, 757, 727, 787, 777, 717, 247, 737, 767};
int n = 757;

std::sort(ints.begin(), ints.end());

if (std::binary_search(ints.begin(), ints.end(), n))
    cout << std::format("Found value {}\n", n);

auto ipos = std::ranges::binary_search(ints, n);

cout << format("Found value {} at index {}\n", n,std::distance(ints.begin(), ipos));
```


### Portioning


### For_each
Applies a `unary function`(<font color=#ffcba4>takes one argument</font>) to each element of a single container.
- Syntax : `std::for_each(start, end, fx)` 
-  If `UnaryFunc` is not [CopyConstructible](https://en.cppreference.com/w/cpp/named_req/CopyConstructible.html "cpp/named req/CopyConstructible"), the [behaviour is undefined](https://en.cppreference.com/w/cpp/language/ub.html "cpp/language/ub").
```c++
#include <string>
#include <algorithm>
 // . . .
using std::string;
std::deque<int> q{1, 2, 3, 4, 5, 6, 7, 8, 9};
std::vector<string> s{"Fender", "Rickenbacker", "Alembic", "Gibson"};

auto prt = [](const auto& x) {print_this(x);};

std::for_each(q.begin(), q.end(), prt);
std::for_each(s.begin(), s.end(), prt);

std::ranges::for_each(q, prn); 
std::ranges::for_each(s, prn);
```


### Transform
Provides methods to modify the target containers:
1. Apply a function across a container and replace its elements with the results. 
2. Apply a function across a container and place the results in a separate container
- Syntax : `std::transform(start, end, startForPlacing, fx)`
- Source requires `begin()` and `end()`
- Target requires `begin()`
```c++
#include <algorithm>

std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9};
std::transform(v.begin(), v.end(), v.begin(),
    [](int k) {return tmpl_square(k);});

std::ranges::transform(v, v.begin(), [](int k) {return tmpl_square(k);})
```

To insert to back of a container:
```c++
#include <iterator>        // std::back_inserter
#include <algorithm>

std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9};
std::deque dq;

std::transform(v.begin(), v.end(), 
	std::back_inserter(dq), [](int n) {return tmpl_square(n) + 0.5;});

std::ranges::transform(v, std::back_inserter(dq), 
	[](int n) {return tmpl_square(n) + 0.5;});
```

### Copy
More in #move_semantics

Use the copy constructor to copy objects of <font color=#ffcba4>same type</font>
```c++
 std::vector<int> v{1, 2, 3, 4, 5};
 auto w = v;
```

To copy `STL Containers` of <font color=#ffcba4>different type</font> but same `dataType`:
```c++
std::vector<int> v{1, 2, 3, 4, 5};

std::deque<int> queue(std::begin(v), std::end(v));
std::list<int> list(v.begin(), v.end());
```

`std::copy` & `std::ranges::copy`  exist, but sequence constructors are preferable.

### Move
More in #move_semantics 
- Must have move constructor that is `noexcept`

`std::move`
To move to a container of the <font color=#ffcba4>same type</font> making original `nullptr`
```c++
 std::vector<int> v{1, 2, 3, 4, 5};
 auto w = std::move(v);
```

To move an existing container or to `STL Containers` of <font color=#ffcba4>different type</font> :
```c++
#include <iterator>        // std::back_inserter
 // . . .

std::deque<int> dq;
std::list<int> lst;

std::move(v.begin(), v.end(), std::back_inserter(dq));
std::ranges::move(u, std::back_inserter(lst));

```

### Begin & End
As of <font color=#7cfc00>C++ 11</font> `std::begin(.)`, `std::end(.)`, `std::cbegin(.)`,  `std::cend(.)` were added.
- Provides a generalization with C-style arrays without `.begin()` etc.
- Removes need for `.begin()`
```C++
int c_array_ints[] = {10, 20, 30, 45, 50, 60};

// Copy to vector:
vector<int> v_ints(std::begin(c_array_ints), std::end(c_array_ints));

// Copy vector to list:
std::list<int> list_ints(std::begin(v_ints), std::end(v_ints));

// Apply algorithms to C-style arrays:
std::transform(std::begin(c_array_ints), std::end(c_array_ints),
    std::begin(c_array_ints), [](int k) {return tmpl_square(k);});
```


### Parallel
As of <font color=#7cfc00>C++ 17</font> parallel algorithms are available to the standard library (in `<algorithm>`) using:
- `std::execution::seq`
- `std::execution::par` 
- `std::execution::par_unseq`. 
- Header `#include <execution>`

These policies allow you to hint to the compiler and library that an algorithm (like `std::sort`, `std::for_each`, `std::transform`) can be executed in parallel.

Typically need to link against a **threading library** (like `TBB` - Threading Building Blocks for `GCC/Clang`, or the default parallel backend for `MSVC`).

- **For GCC/Clang (Linux/macOS):**
```Bash
g++ -std=c++17 -O3 -Wall -pthread -ltbb your_program_name.cpp -o your_program_name
# Or for Clang, you might need specific flags depending on your TBB setup:
# clang++ -std=c++17 -O3 -Wall -pthread -stdlib=libc++ -lc++abi your_program_name.cpp -o your_program_name
```
(Note: `pthread` is for general threading support, `-ltbb` links the TBB library, which is a common backend for parallel algorithms in GCC/Clang.)

- **For MSVC (Windows):**
```Bash
    cl /std:c++17 /O2 /MD /EHsc your_program_name.cpp
```
 (`MSVC`'s standard library implementation usually has its own parallel backend built-in, so explicit linking like TBB isn't always needed.)

## Ranges
Provides more intuitive than specifying the `begin` and `end` iterator positions every time a container is to have an `STL algorithm` applied.
- Requires <font color=#7cfc00>C++ 20</font> and higher
- Used when operating on the whole container 
- Added using `#include <ranges>`
- Declaration `std::ranges::...`
- passing a `range` into an `STL algorithm` is more concise than passing in `iterator` positions.

Resource:
- [How to make a container from a C++20 range](https://timur.audio/how-to-make-a-container-from-a-c20-range)
- [Back to Basics: (Range) Algorithms in C++ - CppCon](https://www.youtube.com/watch?v=eJCA2fynzME)
- [Effective Ranges: A Tutorial for Using C++2x Ranges - CppCon 2023](https://www.youtube.com/watch?v=QoaVRQvA6hI)

Example <font color=#30D5C8>count_if :</font>
```c++
#include <ranges>
 // . . .
 
 std::vector<int> int_vec . . .;
 std::list<int> int_list . . .;
 
 num_odd = std::ranges::count_if(int_vec, is_odd);
 num_odd = std::ranges::count_if(int_list, is_odd);
```

Example <font color=#30D5C8>for_each :</font>
```c++
#include <string>
#include <ranges>
 // . . .
using std::string;
std::deque<int> q{1, 2, 3, 4, 5, 6, 7, 8, 9};
std::vector<string> s{"Fender", "Rickenbacker", "Alembic", "Gibson"};
 
auto prn = [](const auto& x) {print_this(x);};

std::ranges::for_each(q, prn);
std::ranges::for_each(s, prn);
```

<font color=#fd6206><strong>Footnote :</strong></font>
 • Some `STL algorithms` have not yet been updated for ranges.
 • Parallel execution policies are not yet available with ranges (currently <font color=#7cfc00>C++ 23</font>).

### Range views
Summary use - <font color=#ffcba4>Data filtering.</font>
Allows certain operations to be performed by using the elements of a container without the overhead from copying data and generating temporary container objects.
- `Non-owning`, similar to a reference
- Can be `mutating` or `nonmutating`
- Include `<ranges>`

 Resource 
 - [Range-v3: User Manual](https://ericniebler.github.io/range-v3/index.html#tutorial-quick-start)
 - [View classes | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/standard-library/view-classes?view=msvc-170)

Functional behaviour allows composition of many functions, similar to [piping in Linux](https://linuxsimply.com/what-is-piping-in-linux/).
These pipelines of operations are also performed *lazily*. 
- Combine operations on views that will be deferred and performed only when needed. 
- Leads to fewer loops, less copying of data, and overall better throughput.
- From <font color=#7cfc00>C++ 20</font>

#### Range adaptor
Used to create a range view.
Examples:
 `std::views::take`  - Takes the first n elements of a view and discards the rest
 `std::views::filter` -  Filters a set of elements according to a predicate
 `std::views::transform` - Modifies elements of a view based on an auxiliary function (without modifying the original values in the source range)
 `std::views::drop` - Removes the first n elements of a view

_Given a vector, take the 1st 5 elements, filter elements than –2. Square each value of this new view and remove the first two elements of squares_
```c++
#include <ranges>
 // . . .
std::vector<double> w(10);
std::iota(w.begin(), w.end(), -5.5);  // Filling in the vector

auto take_five = std::views::take(w, 5);
for(double x : take_five) {print_this(x);}
// −5.5, − 4.5, − 3.5, − 2.5, − 1.5

 auto two_below = std::views::filter(take_five, 
	 [](double x) {return x < -2.0;})
// −5.5, − 4.5, − 3.5, − 2.5

auto squares = std::views::transform(two_below, 
	[](double x) {return x * x;});
// 30.25, 20.25, 12.25, 6.25

auto drop_two = std::views::drop(squares, 2);
// 12.25, 6.25
```
Use `auto` as the return  type is long, example : 
``` c++
//take_five 
std::ranges::take_view<std::ranges::ref_view
	<std::vector<double,std::allocator<double>>>>
```


#### Chaining views
**Composition** of `range adaptors`, with the result of one taken in as input to the next.
- This is done with the pipe operator ( `|` ),
_Same example as range adaptor:_
```c++
auto drop_two = w | std::views::take(5)
    | std::views::filter([](double x) {return x < -2.0;})
    | std::views::transform([](double x) {return x * x;})
    | std::views::drop(2);
```


#### Manipulation & storage
 As `views` are `ranges`, they can be used as range expressions in range-based for loops.
_Example : copy the results from the above view into a vector_
```c++
std::vector<double> u;
u.reserve(take_five.size());    // Note there is a size() member

for (auto x : take_five){
	u.push_back(x);
}
```

_You can start with unfiltered data and filter as you go :_
```c++
for (auto x : w | std::views::take(5)){
    print_this(x);    // -5.5 -4.5 -3.5 -2.5 -1.5
}

/*** multiple chained views ***/
for (auto x : w | std::views::take(5)
        | std::views::transform([](double x) {return x * x;})){
    print_this(x);    // 30.25 20.25 12.25 6.25 2.25
}
```


# Function Objects - Functors
#Functor - an object of a class that **overloads the function-call operator `operator()`**.
- Makes instances of that class behave like functions. 
- Can "call" them using parentheses, just like a regular function.

<font color=#30D5C8>Uses :</font>
1. <font color=#ffcba4><strong>State :</strong></font> can have member variables, which allows them to maintain "_state_" between calls.
2. <font color=#ffcba4><strong>Polymorphism :</strong></font> (though less common than with `std::function`): can be used polymorphically if dealing with abstract base classes or `std::function`.
3. <font color=#ffcba4><strong>Combinability :</strong></font> composable with STL algorithms.

```C++
class Quadratic {
	double a, b, c;
public:
    Quadratic(double x, double y, double z):
        a{x}, b{y}, c{z} {} 

	// Overloaded function-call operator
    double operator()(double x) const {
        return (a * x + b) * x + c; // quadratic calculation
    }
};
```

- **State (`a`, `b`, `c`):** The coefficients `x`, `y`, and `x` are stored as private member variables (`a_`, `b_`, `c_`). This is the "state" of the `Quadratic` object.
- **Constructor:** The constructor allows you to create instances of `Quadratic` with specific coefficients. 
- **`operator()(double x) const`:** When  "called" the `q` object (e.g., `q(5.0)`), this `operator()` is executed. It uses the `a`, `b`, and `c` values that were stored when `q` was constructed to perform the calculation .
$$
ax^2+bx+c
$$

```c++
Quadratic q{2.0, 4.0, 2.0};
std::vector<double> v{-1.4, -1.3, -1.2, -1.1, 0.0, 1.1, 1.2, 1.3, 1.4}; // Input data

std::deque<double> y; // Output container
std::ranges::transform(v, std::back_inserter(y), q); // 2. Pass the functor to std::ranges::transform
```

# Pointers
## RAII
#RAII - Resource Acquisition is initialization
A programming technique which says the <font color=#ffcba4>lifetime of a resource is guaranteed to be tied to its owner.</font> 
When an owning object goes out of scope:
- The owner is responsible for destroying its resource and returning the memory it occupied to the system.
## Clone
Refer to <font color=#ffcba4><strong>Rule of 3</strong></font>
Prior to <font color=#7cfc00>C++ 11</font> and the advent of smart pointers, a common approach to implementing RAII
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
#move_semantics  - Transfer ownership of an object, as opposed to copying it, thus avoiding a potentially nontrivial performance hit due to object copy.
- Reference - [Engineering Distinguished Speaker Series: Howard Hinnant](https://www.youtube.com/watch?v=vLinb2fgkHk)

Example, passing a vector/string as a parameter :
- Passing by reference allows for fast access but the object doesn't own it but is unsafe as it can be altered externally
- To own before C++11, required deep copying the data improving safety at cost of performance

<font color=#ffcba4><strong>Special members :</strong></font> 
```c++
default constructor  -->  X();
destructor           -->  ~X();
copy constructor     -->  X(X const&);
copy assignment      -->  X& operator=(X const&);
move constructor     -->  X(X&&);
move assignment      -->  X& operator=(X&&);
```

<font color=#ffcba4><strong>Declaration :</strong></font> what counts as user-declared
- Defaulted -> compiler generates a default implementation.
- Not-declared -> compiler doesn't provide code
- User declared -> user specified code, including keywords `default` & `delete`
```c++
struct X{
	x(){};  // User defined
	x();    // User defined
	x() = default;  // User defined to use default
	x() = delete;   // User defined to remove function
};
```

|                     | default constructor                      | Destructor                               | Copy constructor                         | Copy assigment                           | Move constructo                          | Move assigment                           |
| ------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| Nothing             | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     |
| Any constructor     | <font color=#cd76ea>Not declared</font>  | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     |
| default constructor | <font color=#7cfc00>User declared</font> | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     |
| Destructor          | <font color=#30D5C8>defaulted</font>     | <font color=#7cfc00>User declared</font> | <font color=#ff0800>defaulted</font>     | <font color=#ff0800>defaulted</font>     | <font color=#cd76ea>Not declared</font>  | <font color=#cd76ea>Not declared</font>  |
| Copy constructor    | <font color=#cd76ea>Not declared</font>  | <font color=#30D5C8>defaulted</font>     | <font color=#7cfc00>User declared</font> | <font color=#ff0800>defaulted</font>     | <font color=#cd76ea>Not declared</font>  | <font color=#cd76ea>Not declared</font>  |
| Copy assigment      | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#ff0800>defaulted</font>     | <font color=#7cfc00>User declared</font> | <font color=#cd76ea>Not declared</font>  | <font color=#cd76ea>Not declared</font>  |
| Move constructor    | <font color=#cd76ea>Not declared</font>  | <font color=#30D5C8>defaulted</font>     | <font color=#eab676>Deleted</font>       | <font color=#eab676>Deleted</font>       | <font color=#7cfc00>User declared</font> | <font color=#cd76ea>Not declared</font>  |
| Move assigment      | <font color=#30D5C8>defaulted</font>     | <font color=#30D5C8>defaulted</font>     | <font color=#eab676>Deleted</font>       | <font color=#eab676>Deleted</font>       | <font color=#cd76ea>Not declared</font>  | <font color=#7cfc00>User declared</font> |
<font color=#fd6206>context :</font> _red - check rules below_

## Rule of 3
If any one of the 3 special members is defined, include definitions for the remaining 2. 
- This can include just declaring them as `default` or `delete` where appropriate, and leaving generation of the implementations to the compiler.

Special functions:
1. copy constructor
2. copy assignment operator 
3. destructor

## Rule of 5
If any one of the five special members is defined, include definitions for the remaining four. 
- This can include just declaring them as `default` or `delete` where appropriate, and leaving generation of the implementations to the compiler.
- For <font color="7cfc00">C++11 and higher</font>.
- Reference - [The Rule of Zero, Five, or Six](https://www.modernescpp.com/index.php/c-core-guidelines-constructors-assignments-and-desctructors/)

Special functions:
1. copy constructor
2. copy assignment operator 
3. destructor
4. move constructor
5. move assignment operator 

<font color=#ffcba4><strong>Corollary :</strong></font>
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

### Compiler-Generated Move Constructor (defaulted):
The C++ compiler will implicitly generate a move constructor for a class if **all** of the following conditions are met:

1. **No user-declared copy constructor:** You haven't defined your own copy constructor.
2. **No user-declared move constructor:** You haven't defined your own move constructor.
3. **No user-declared copy assignment operator:** You haven't defined your own copy assignment operator.
4. **No user-declared move assignment operator:** You haven't defined your own move assignment operator.
5. **No user-declared destructor:** You haven't defined your own destructor.

If all these conditions are true, the compiler-generated move constructor will perform a **member-wise move** of the non-static data members of the class. This means:

- For members of built-in types (like `int`, `float`, pointers), a bitwise copy occurs.
- For members that are objects of class types, the move constructor of that member class will be called (if it exists and is accessible). If the member class doesn't have a move constructor, its copy constructor will be called instead.
```c++
class X : public Base{
	Member m;
public
	X(X&& src): Base(static_cast<Base&&>(src)),
		m(static_cast<Base&&>(src.m));
}
```

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
	SimpleClass x; 
public: 
	Enclose(const SimpleClass& sc); // Copy Constructor 
	Enclose(SimpleClass&& sc); // Move Constructor
	SimpleClass get_sc() const; // Accessor 
}; 

// Implementation: 
Enclose::Enclose(const SimpleClass& sc) :x{sc} {} // Constructor 1 
Enclose::Enclose(SimpleClass&& sc) :x{std::move(sc)} {} // Constructor 2 

SimpleClass Enclose::get_sc() const { // Accessor 
	return x; 
}
```

Disable the copy constructor functionality : 
```c++
class EncloseSingleConstructor { 
	SimpleClass x; 
public: 
	EncloseSingleConstructor(SimpleClass sc); 
	SimpleClass getX() const; 
}; 

// Constructor implementation: 
EncloseSingleConstructor::EncloseSingleConstructor(SimpleClass sc) : x{std::move(sc)}{}

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

<font color="ffcba4">Init-capture</font> - create a new variable inside the lambda's closure, and initialize it with a value
- C++ version >> <font color="7cfc00">C++14 and higher</font>
```c++
auto checkInterval = [low = 3, high = 6](int n) {
	return low <= n && n <= high;
};
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

## Math constants
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


## Numeric Algorithms
Algorithms that perform mathematical operations
- Under `Standard Library` header, `numeric`

`std::iota` :
Generates a sequence of incremented values
- Useful for testing
- Generating a fixed schedule of time increments
```c++
#include <numeric>
 //. . .
vector<int> w(6);
std::iota(w.begin(), w.end(), -2.5);
// -2.5 -1.5 -0.5 0.5 1.5 2.5
```


`std::accumulate` :
Computes the sum of elements in a container.
- Result is a scalar value.
```c++
double sum_of_reals = std::accumulate(w.begin(), w.end(), 0.0);
// 0.0 + -2.5 + -1.5 + -0.5 + 0.5 + 1.5 + 2.5 == 0.0
```
Can be generalized to perform other binary operation:
- Use header [functional](https://en.cppreference.com/w/cpp/header/functional.html)
- Multiplication, Minus, division, mod, negation …etc.
```c++
#include <functional>        // std::multiplies, etc
 //. . .
auto prod = std::accumulate(w.begin(), w.end(), 1.0, 
	std::multiplies<double>());
// 1.0 × −2.5 × −1.5×⋯×2.5 = −3.515625
```


`std::ranges::fold_left`
`<numeric>` doesn't have `ranges` versions (as of <font color=#7cfc00>C++23</font>)
Similar to `std::accumualate` but does not default to the addition operator
```c++
int sum_of_ints = std::ranges::fold_left(v, 0, std::plus<int>());  // = 621

double prod_of_reals = std::ranges::fold_left(w, 1.0,
    std::multiplies<double>());                              
// = -3.515625
```


`std::inner_product` :
Computes the dot product from two containers
- Result is a scalar value.
```C++
int dot_prod = std::inner_product(v.begin(), v.end(), u.begin(), 0);
// 0 is an initial value that is added to the dot product
```
Can be abstracted for any two operations
- sum & minus, product & sum, …etc.
```c++
vector<double> y(w.size());        // 6 elements

std::iota(y.begin(), y.end(), 1.5);
double sum_diffs = std::inner_product(w.begin(), w.end(), 
	y.begin(), 0.0, std::plus<double>(), std::minus<double>());
// −4+ (−4)⋯+ (−4) = −24
```


`std::adjacent_difference` :
Computes the difference of each element and its predecessor
- Result is the original with elements replaced with modified values
- Alternatively a separate container where the values are inserted.
Computes:
$$
y_i = x_i - x_{i-1}
$$
For each `i = 1, ...,n` and place the result  in a new container / in the original by replacing the elements :
$$
\{x_0, y_1, y_2, ..., y_{n-1}, y_n \}
$$
Consider using a `deque` as the target container in order to pop `x0` off the front of the container and store only the differenced values.
```c++
 std::vector<int> u{100, 101, 103, 106, 110, 115, 121};
 std::deque<int> adj_diffs;
 
std::adjacent_difference(u.begin(), u.end(), std::back_inserter(adj_diffs));        
// adj_diffs = 100 1 2 3 4 5 6

adj_diffs.pop_front();  // adj_diffs now = 1 2 3 4 5 6
```
Can specify an operation other than the default:
```c++
std::deque<double> adj_sums{-2.5, -1.5, -0.5, 0.5, 1.5, 2.5};
std::adjacent_difference(adj_sums.begin(), adj_sums.end(),
    adj_sums.begin(), std::plus<double>());

adj_sums.pop_front();        // adj_sums now = -4 -2 0 2 4
```


`std::partial_sum` :
Computes the cumulative sums at each element in a container (like _CDF_ in stats)
- Result is the original with elements replaced with modified values
- Alternatively a separate container where the values are inserted.
```c++
vector<int> z{10, 11, 13, 16, 20, 25, 31};
vector<int> part_sums;

part_sums.reserve(z.size());
std::partial_sum(z.begin(), z.end(), std::back_inserter(part_sums));
// Result: part_sums  = 10 21 34 50 70 95 126
```
Can specify an operation other than the default:
```c++
vector<int> part_prods(z.size());

std::partial_sum(z.begin(), z.end(), part_prods.begin(), std::multiplies<int>());
 // Result: part_prods = 10 110 1430 22880 457600 11440000 354640000
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


## Random Number generation


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
