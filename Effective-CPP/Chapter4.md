# Chapter 4. Smart Pointers
## Item 18: Use std::unique_ptr for exclusive-ownership resource management.
**std::unique_ptr** embodies exclusive ownership semantics.  
**std::unique_ptr** are the same size as raw pointers.  
A non-null **std::unique_ptr** always owns what it points to.  
Copying a **std::unique_ptr** isn’t allowed, it is a move-only type.

By default, resource destruction takes place via **delete**, but custom deleters can be specified. Stateful deleters and function pointers as deleters increase the size of **std::unique_ptr** objects.
* The **std::unique_ptr** automatically deletes what it points to when the **std::unique_ptr** is destroyed.  
* **Custom deleters**: arbitrary functions (or function objects, including those arising from lambda expressions) to be invoked when it’s time for their resources to be destroyed.
```C++
auto delInvmt = [](Investment* pInvestment)
 {
    makeLogEntry(pInvestment);    // (a lambda expression custom deleter)
    delete pInvestment;               
 };
```
When a **custom deleter** is to be used, its type must be specified as the second type argument to **std::unique_ptr**.
``` C++
std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);
```
* Stateless function objects (e.g., from lambda expressions with no captures) has the same size of raw pointer.  
For deleters that are function objects, the change in size depends on how much state is stored in the function object.
```C++
void delInvmt2(Investment* pInvestment)
{
   makeLogEntry(pInvestment);
   delete pInvestment;
}
template<typename... Ts> // size of Investment* plus at least size of function pointer
std::unique_ptr<Investment, void (*)(Investment*)>
makeInvestment(Ts&&... params);
```

Do not use **std::unique_ptr<T[ ]>**. There are better data structures such as **std::array**, **std::vector**, and **std::string**.
Converting a **std::unique_ptr** to a **std::shared_ptr** is easy. Flexible for callers.
