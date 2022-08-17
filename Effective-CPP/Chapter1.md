# Chapter 1. Deducing Types
## Item 1: Template Type Deduction
```C++
template<typename T>
void f(ParamType param);

f(expr);
```
**T** and **ParamType** are deduced by **expr**.
There are 3 cases according to different  **ParamType**:
### 1.ParamType is Ref or Ptr
decuction rules:
* ignore param's reference part
* pattern-match

```C++
template<typename T>
void f(T& param);  //ParamType is a reference
//pseudocode
f(int);// T:int    ParamType:int&
f(const int);// T:const int     ParamType:const int&
f(const int&);// T:const int     ParamType:const int&

template<typename T>
void f(T* param);  //ParamType is a pointer
f(int);// T:int    ParamType:int*
f(const int);// T:const int     ParamType:const int*
f(const int&);// T:const int     ParamType:const int*
```

### 2.ParamType is Universal Ref
decuction rules:
* if argument is lvalue, both are deduced to be lvalue-reference
* if argument is rvalue, just as normal

```C++
template<typename T>
void f(T&& param);  //ParamType is a universal reference
f(int);// T and ParamType: int&
f(const int);// T and ParamType: const int&
f(const int&);// T and ParamType: const int&
f(27);// T:int    ParamType:int&&
```
### 3.Pass-by-value
Pass-by-value means a copy is passed in, whether the argument can be modified has nothing to do with the copied one.

decuction rules:
* ignore referenceness
* ignore const and volatile

```C++
template<typename T>
void f(T param);  //ParamType is passed by value
f(int); // T and ParamType: int
f(const int); // T and ParamType: int
f(const int&); // T and ParamType: int
```
### Array Argument
Array decays into a pointer to its first element.
```C++
const char myArray[] = "itsmyarray";

template<typename T>
void f(T param);
f(myArray); // T and ParamType: const char*

template<typename T>
void f(T& param);
f(myArray); // T: const char[sizeofarray]   ParamType: const char(&)[10]
```
### Function Argument
Function decays to a pointer to itself as well.
```C++
void myfunction(int, double);

template<typename T>
void f(T param);
f(myfunction); // T and ParamType: void(*)(int, double) which is a ptr-to-func

template<typename T>
void f(T& param);
f(myfunction); // ParamType: void(&)(int, double) which is a ref-to-func
```


## Item 2: Auto Type Deduction
* Auto type deduction is the same as template type deduction with only one exception.
```C++
auto x = 27; // int
const auto cx = x; // const int
const auto& rx = x; // const int&
auto&& uref1 = x; // int&
auto&& uref2 = cx; // const int&
auto&& uref3 = 27; // int&&
```

* Exception with initializer_list:
```C++
int x1 = { 27 }; //an int with value 27
auto x2 = { 27 }; // type is initializer_list<int>, value is { 27 }
```
* **auto** assumes that a braced initializer represents a **std::initializer_list**, but template type deduction doesn't.
```C++
template<typename T>
void f(T param);
f({ 1, 2, 3 }); // error! T cannot be deduced
```
```C++
template<typename T>
void f(std::initializer_list<T> param);
f({ 1, 2, 3 }); // passed! T is int and param is std::initializer_list<int>
```
* (C++14) **auto** in a function return type or a lambda parameter implies template type deduction.
No **std::initializer_list** allowed.
```C++
auto createInitList(){
    return { 1, 2, 3 }; //error! cannot deduce type for { 1, 2, 3 }
}
```
## Item 3: Decltype
* **decltype** almost always yields the type of a variable or expression without any modifications

* In C++11 and 14, **decltype** is used to declare function template's return type, always with **auto**. Parameters are not determined to be lvalues or rvalues, thus we use universal reference (i.e T&&) to accept both.
```C++
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i)
{
 authenticateUser();
 return std::forward<Container>(c)[i];
}
```
* For lvalue expressions of type T other than names, decltype always reports a type of T&
```C++
decltype(auto) f1(){
    int x = 0; // x is the name of a variable
    return x; // decltype(x) is int
}
decltype(auto) f2(){
    int x = 0;  // (x) is a lvalue expression
    return (x); // decltype((x)) is int&
}
```

## Item 4: Know how to view deduced types
* Deduced types can often be seen using **IDE editors**, compiler **error messages**, and the **Boost TypeIndex library**.
* IDE compilers are not always reliable. The type been passed to a template function is passed by-value. Thus, referenceness and constness are ignored. So an understanding of C++’s type deduction rules remains essential.
