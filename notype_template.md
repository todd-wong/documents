## 非类型模板
模板参数支持使用非类型参数，比如整型或enum变量等。

### 类模板
在类模板中，我们实现了Stack类，现在如果已知Stack容量的上限，我们也可以用
另一种方式实现：

basics/stack8.hpp
``` c++
#include <array>
#include <cassert>

template <typename T, std::size_t MaxSize>
class Stack {
  public:
   void push(T const& elem);
   ...
  private:
   std::array<T, MaxSize> elems;
   std::size_t numElems = 0;
};

template <typename T, std::size_t MaxSize>
void Stack<T, MaxSize>::push(T const& elem) {
    assert(numElems < MaxSize);
    elems[numElems++] = elem;
}
```

### 函数模板
函数模板也支持非类型参数，比如：

basics/addvalue.hpp
``` c++
template <int Val, typename T>
T addValue(T x) {
    return x + Val;
}
```

这样我们可以迭代给每个元素加5。

``` c++
std::transform(source.begin(), source.end(), dest.begin(),
    addValue<5, int>);
```

### 限制
非类型参数只可以是常整数（包括enum）/指针（指向函数、对象、成员函数）/ std::nullptr_t。