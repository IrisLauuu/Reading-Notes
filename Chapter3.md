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
