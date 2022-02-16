# Chapter 2. auto
## Item 5: Prefer auto to explicit type declarations.
* Compared with explicit specified types, **auto** variables save typing, ease the process of refactoring. **auto** variables must be initialized cause thier type deduced from initializer.
```C++
int x1;         // potentially uninitialized
auto x2;        // error!
auto x3 = 0;    // yes!
```
* **auto** variable holding a closure behaves better than **std::function** holding a closure. In lambda expression for example:
```C++
auto derefLess =
 [](const auto& p1, const auto& p2) { return *p1 < *p2; };
```
* Use **auto** variables to avoid type mismatches.
```C++
std::vector<int> v;
//v.size() returns std::vector<int>::size_type
unsigned sz = v.size(); // unsigned is 32 bits, error in 64-bits Windows
auto sz = v.size(); // perfectly to be deduced as std::vector<int>::size_type
```
```C++
std::unordered_map<std::string, int> m;
for (const std::pair<std::string, int>& p : m){
    …   // key of the hash table is const
        // it should be
        // const std::pair<const std::string, int>& p : m
}
for(const auto& p :m){
    …   // use auto is much more better
}
```

## Item 6: Use the explicitly typed initializer idiom when auto deduces undesired types.
* Invisible proxy types can cause **auto** to deduce the “wrong” type for an initializing expression. Typically used in types returned from functions works with **operator[ ]**.
```C++
// function returns std::vector<bool>::reference instead of std::vector<bool>
// auto deduction is not bool as we expect
auto result = function(argument)[int];
```
* Use **explicitly typed initializer idiom** to force **auto** do what you want. Plus explicit type cast.
```C++
auto result = static_cast<bool>(function(argument)[int]);
```
```C++
double d = 1.0;
int i = d;  // No!
auto i = static_cast<int>(d); // Yes!
```
