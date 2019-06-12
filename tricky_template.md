## 杂项
这一节我偏向于具体写模板时遇到的细节问题。

### 零初始化
考虑写一个对vector求和的模板，实现如下：

basics/initialize1.h
``` c++
template <typename T>
T sum(const std::vector<T>& coll) {
    T result;     // (1)
    for (const auto& v : coll)
      result += v;
    return result;
}
```

现假设实例化T = int时，这时我们发现(1)中result未初始化，c++编译器对
这个问题的解决方案是：

basics/initialize2.h
``` c++
template <typename T>
T sum(const std::vector<T>& coll) {
    T result{};  // 或者T result = T();
    ...
}
```
个人推荐使用`{}`，这会将基本类型转为0/false/nullptr。

### typename使用
typename除了在template后指定模板参数，也指定依赖模板参数的类型。比如：

basics/typename.h
``` c++
template <typename T>
void print(const T& coll) {
    typename T::const_iterator pos;          // (1)
    typename T::const_iterator end(coll.end());       // (2)
    for (pos = coll.begin(); pos != end; ++pos)
      std::cout << *pos << ' ';

    std::cout << '\n';
}
```