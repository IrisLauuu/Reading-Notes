# Chapter 3. Moving to Modern C++
## Item 7: Distinguish between () and {} when creating objects.
* Distinguish copy constructor from copy operator(=)  (i.e if its an assignment)
```C++
Widget w1;
Widget w2 = w1; // not an assignment; calls copy ctor
w1 = w2; // an assignment; calls copy operator=
```
### Uniform initialization be used in the widest variety of contexts
* Initialization values may be specified with parentheses, an equals sign, or braces:
```C++
int x(0); // initializer is in parentheses
int y = 0; // initializer follows "="
int z{ 0 }; // initializer is in braces
```
* In C++11, ways to specify default initialization values for non-static data members in a class:
```C++
class Widget {
…
private:
 int x(0); // error! no parentheses!
 int y = 0; // fine
 int z{ 0 }; // also fine
};
```
* Uncopyable objects (e.g., **std::atomics**) may be initialized using braces or parentheses, but not using “=”:
```C++
std::atomic<int> x{ 0 }; // fine
std::atomic<int> y(0); // fine
std::atomic<int> z = 0; // error!
```
### Advantages of uniform initialization
*  Prohibits implicit **narrowing conversions** among built-in types
```C++
double x, y, z;
int sum1{ x + y + z }; // error! sum of doubles cannot be expressible as int
int sum2(x + y + z); //ok
int sum3 = x + y + z;  //ok
```
* It’s immune to C++’s **most vexing parse**.  
  **most vexing parse**: when default constructor is used to create an obj, it ends up declaring a function by accident.
```C++
  Widget w1(10); // fine, call constructor with 10
  Widget w2(); // most vexing parse! declares a function named w2 that returns a Widget!
  Widget w3{}; // calls Widget constructor with no args
               // fine, because functions cannot be declared using braces
```

### Disadvantages of uniform initialization

* During constructor overload resolution, braced initializers are matched to **std::initializer_list** parameters if at all possible, even if other constructors offer seemingly better matches.  
If one or more constructors declare a parameter of type **std::initializer_list**, calls using the braced initialization syntax strongly prefer the overloads taking **std::initializer_list**.
```C++
class Widget {
public:
 Widget(int i, bool b);
 Widget(int i, double d);
 Widget(std::initializer_list<long double> il);
};
//create objects
Widget w1(10, true); // as normal, calls first constructor
Widget w2{10, true}; // calls the third one instead of the first
                       // (10 and true convert to long double)
Widget w3(10, 5.0); // as normal, calls second constructor
Widget w4{10, 5.0}; // calls the third one (10 and 5.0 convert to long double)
```
Only if there’s no way to convert the types of the arguments in a braced initializer to the type in a **std::initializer_list** do compilers fall back on normal overload resolution.
```C++
 class Widget {
 public:
  Widget(int i, bool b);
  Widget(int i, double d);
  Widget(std::initializer_list<std::string> il);
 };
 //create objects
 Widget w1(10, true); // calls first ctor
 Widget w2{10, true}; // calls first ctor
                    // because there is no way to convert ints and bools to std::strings:
```
### Tremendous difference between parentheses and braces when creating a std::vector< numeric type > with two arguments.
```C++
std::vector<int> v1(10, 20); // use non-std::initializer_list ctor: 10-element
                              //  all elements have value of 20
std::vector<int> v2{10, 20}; // use std::initializer_list ctor: 2-element
                             // element values are 10 and 20
```
## Item 8: Prefer nullptr to 0 and NULL.
* **0** could be an integer or a null pointer. **NULL** is the same.
* **nullptr**'s actual type is **std::nullptr_t**.
 **std::nullptr_t** implicitly converts to all raw pointer types, and that’s what makes **nullptr** act as if it were a pointer of all types.

 ```C++
 auto result = findRecord( /* arguments */ );
if (result == 0) {
 … //ambiguous, not sure what type result is
}
```
```C++
auto result = findRecord( /* arguments */ );
if (result == nullptr) {
 … //good, result must be a pointer.
}
```
## Item 9: Prefer alias declarations to typedefs.
* The purpose of using alias and typedefs: give long types another name which is easy to understand.
```C++
//same expression, simplyfy a long type
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> anotherName;
using anotherName = std::unique_ptr<std::unordered_map<std::string, std::string>>;
`
//give function pointer another name
typedef void (*FP)(int, const std::string&);
using FP = void (*)(int, const std::string&);
```
* **alias declarations** can be used in templates (called alias templates) while typedefs cannot.
```C++
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;
MyAllocList<Widget> lw;
```
To use typedefs, they should be nested in templatized structs.
```C++
template<typename T>
struct MyAllocList {
 typedef std::list<T, MyAlloc<T>> type;
};
MyAllocList<Widget>::type lw;
```
* C++11 provides tools to perform type transformations in the form of type traits, an assortment of templates inside the header **type_traits**, but they use typedefs. In C++14, alias templates are used in corresponding transformations.
```C++
std::remove_const<T>::type // C++11: const T → T
std::remove_const_t<T> // C++14 equivalent
`
std::remove_reference<T>::type // C++11: T&/T&& → T
std::remove_reference_t<T> // C++14 equivalent
`
std::add_lvalue_reference<T>::type // C++11: T → T&
std::add_lvalue_reference_t<T> // C++14 equivalent
```

## Item 10: Prefer scoped enums to unscoped enums.
* **unscoped enums**: enumerator names leak into the scope containing their enum definition
```C++
enum Color { black, white, red };
auto white = false; // error! white already declared in this scope
```
 **scoped enums** (enum classes) dont leak names
```C++
enum class Color { black, white, red };
auto white = false; // fine, no other "white" in scope
Color c = white; // error! no enumerator named "white" is in this scope
Color c = Color::white; // fine
```
* Enumerators for **unscoped enums** implicitly convert to integral types.
Enumerators of **scoped enums** convert to other types only with a cast.

* Both **scoped** and **unscoped enums** support specification of the underlying type.
 The default underlying type for **scoped enums** is int.
 **Unscoped enums** have no default underlying type.

## Item 11: Prefer deleted functions to private undefined ones.
* Prevent a particular function from being called.
  C++98: use **private** to restrain accessibility
```C++
template <class charT, class traits = char_traits<charT>>
class basic_ios : public ios_base {
private:
    basic_ios(const basic_ios& ); // not defined
    basic_ios& operator=(const basic_ios&); // not defined
};
```
  C++11: use **“= delete”** to mark functions as deleted functions. And public!
```C++
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
      basic_ios(const basic_ios& ) = delete;
      basic_ios& operator=(const basic_ios&) = delete;
};
```
* **Any** function may be deleted, while only member functions may be private.
  Overloads with **delete** make the right parameter is passed in.
```C++
bool isLucky(int number); // non-member function
bool isLucky(char) = delete; // reject chars
bool isLucky(bool) = delete; // reject bools
bool isLucky(double) = delete; // reject doubles and floats
```
* Template instantiations can also be **delete**, while private member functions can’t.
```C++
template<typename T>
void processPointer(T* ptr); // a template with pointer
```
 void* pointers and char* pointers should be rejected by our template:
 ```C++
 template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
```
And they can be deleted outside the class:
```C++
class Widget {
  public:
    template<typename T>
    void processPointer(T* ptr);
};
template<>
void Widget::processPointer<void>(void*) = delete;
```

## Item 12: Declare overriding functions override.
* Virtual function implementations in derived classes override the implementations of their base class counterparts. For overriding to occur, several requirements must be met:
 + The base class function must be virtual.
 + The base and derived function names must be identical (except in the case of destructors).
 + The parameter types of the base and derived functions must be identical.
 + The constness of the base and derived functions must be identical.
 + The return types and exception specifications of the base and derived functions must be compatible.
 + (C++11) The functions’ reference qualifiers must be identical. (&lvalue and &&rvalue)
* Since overriding functions in derived class is easy to get wrong, declare overriding functions override explicitly helps with that. Compilers will catch all the overriding-related problems.

## Item 13: Prefer const_iterators to iterators.
*  In C++11, **const_iterators** are both easy to get and easy to use.
The container member functions cbegin and cend produce **const_iterators**, even for non-const containers, and STL member functions that use iterators to identify positions (e.g., insert and erase) actually use **const_iterators**.
```C++
std::vector<int> values;
auto it = std::find(values.cbegin(),values.cend(), 1983);
values.insert(it, 1998);
```

## Item 14: Declare functions noexcept if they won’t emit exceptions.
* **noexcept** is part of a function’s interface, and that means that callers may
depend on it (important to clients).
* **noexcept** functions are more optimizable than non-noexcept functions (for compilers).
* **noexcept** is particularly valuable for the move operations on containers, and swap in STL.
  However, the interface specifications for **move** in the Standard Library lack noexcept. In practice, implementers often declare them noexcept, even though the Standard does not require them to do so.
* Most functions are **exception-neutral**. They throw no exceptions themselves, but functions they call might emit one. They allow the emitted exception to pass through to a handler further up the call chain. (They never **noexcept**.)

## Item 15: Use constexpr whenever possible.
* **constexpr objects** are const and are initialized with values known during compilation.
Values known during compilation are privileged. They may be placed in read-only memory of embedded systems. Besides, they can be used in specification of array sizes, integral template arguments (including lengths of std::array objects), enumerator values, alignment specifiers, and more.
Note that **const** doesn’t offer the same guarantee as **constexpr**, because const objects need not be initialized with values known during compilation.
* **constexpr functions** can produce compile-time results when called with arguments whose values are known during compilation.
When a **constexpr function** is called with one or more values that are not known during compilation, it acts like a normal function, computing its result at runtime.
* **constexpr functions** are limited to taking and returning literal types, which essentially means types that can have values determined during compilation. In C++11, all built-in types except void qualify, but user-defined types may be literal, too, because constructors and other member functions may be constexpr.

## Item 16: Make const member functions thread safe.
...to be continued...

## Item 17: Understand special member function generation.
* The special member functions are those compilers may generate on their own: default constructor, destructor, copy operations(copy constructor & copy assignment operator), and move operations(move constructor & move assignment operator).
* Move operations are generated only for classes lacking explicitly declared move operations, copy operations, and a destructor. (two opereations are dependent, both of them or never)
* The copy constructor and the copy assignment operator are independent. They are respectively generated only for classes lacking an explicitly declared one. If a move operation is declared, they both will not be generated.
* Rule of Three:  if you declare any of a copy constructor, copy assignment operator, or destructor, you should declare all three. If automatic generated one is satisfied, use **=default** explicitly declare it.
```C++
class Widget {
public:
  ~Widget(define how to delete it yourself);
  ...
  Widget(Widget&) = default;  // default copy ctor
  Widget& operator=(Widget&) = default; // default copy assign
  ...
  Widget(Widget&&) = default; // default move ctor
  Widget& operator=(Widget&&) = default; // default move assignment operator
```
