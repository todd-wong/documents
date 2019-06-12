## 可变模板
从C++11开始，模板开始支持可变数量的模板参数，这进一步扩展了C对可变数量参数的支持，
下面实例是用可变模板实现的print和printElems。

basics/varprint.hpp
``` c++
#include <iostream>

void print() {
}

template <typename T, typename... Types>
void print(T firstArg, Types... args) {
    std::cout << firstArg << "\n";
    // std::cout << sizeof...(args) << std::endl;
    print(args...);
}

template <typename C, typename... Idx>
void printElems(C const& coll, Idx... idx) {
    print(coll[idx]...);
}
```

在std标准库中也能找到大量使用。

``` c++
namespace std {

template <typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... args);

class Thread {
  public:
    template <typename F, typename... Args>
    explicit thread(F&& f, Args&&... args);
    ...
};

template <typename T, typename Alllocator = allocator<T>>
class vector {
  public:
    template<typename... Args> reference emplace_back(Args&&... args);
};
}
```