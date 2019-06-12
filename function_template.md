模板是C++有用的特性，支持泛型编程(generic programs)，使算法和数据类型分离，从而更大程度上使代码灵活复用。以下我将分为四部分讲解模板的使用：

* 函数模板

* 类模板

* 非类型模板参数

* 可变参数模板

## 函数模板
函数模板代表一类函数，内部实现逻辑都一致，只是数据类型不同，像我们常见max/min/bubbleSort等用函数模板实现可以避免为支持多种数据类型组合而使代码臃肿。

一般格式为：
``` c++
template <typename T1, typename T2, ...>
void someFunc(T1 arg1, T2 arg2, ...) {
  ...
}
```

### 基本使用
以下我们以max作为范例，求两个数的最大值，一种实现为：

basics/max1.hpp
``` c++
template <typename T>
T max(T a, T b) {
    return a < b ? b : a;
}
```

对应地，我们使用它做测试：

basics/max1.cpp
``` c++
#include "max1.hpp"
#include <iostream>
#include <string>

int main()
{
    int i = 3, j = 7;
    std::cout << "max(i, j) = " << ::max(i, j) << std::endl;

    // std::cout << "max(i, 2.2) = " << ::max(i, 2.2) << std::endl;  (1)

    double f1 = 3.14, f2 = 2.0;
    std::cout << "max(f1, f2) = " << ::max(f1, f2) << std::endl;
    // std::cout << "max(i, f2) = " << ::max(i, f2) <<std::endl;   (2)

    std::string s1 = "hello";
    std::string s2 = "world";
    std::cout << "max(s1, s2) = " << ::max(s1, s2) << std::endl;

    return 0;
}
```

上面的运行结果完全符合预期，但若打开(1)和(2)时，会发现编译不过，至此，我们发现模板
函数与一般函数的区别：**模板函数在类型推导，不会类型转换**。

知道这个问题，我们该对应如何调整，回到我们的模板函数的定义，我们发现，其中`a`和`b`
类型必须一致，所以我们可以调整为：
``` c++
std::cout << "max(i, 2.2) = " << ::max<float>(i, 2.2) << std::endl;
std::cout << "max(i, f2) = " << ::max((float)i, f2) << std::endl;
```

上面不管是手动显示实例化或强制转换，都会是代码阅读性和维护都造成困难，尤其在多参数时。
回到函数模板的定义形式，我们可实现为：

basics/max2.hpp
``` c++
template <typename T1, typename T2>
T2 max(T1 a, T2 b) {
    return a < b ? b : a;
}
```
以此定义，我们完全能cover住上面的问题，但我们发现在调用`max(4, 7.7)`会返回`4`,这不符合预期，
我们再次对它做调整：

basics/max3.hpp
``` c++
template <typename RT, typename T1, typename T2>
RT max(T1 a, T2 b) {
    return a < b ? b : a;
}
```
配合max1.cpp的使用方式，我们会发现编译出错，提示出错原因是不能推导出RT的类型，这里我们可以得出另一个
结论：**实例化时，在不依赖其他实参，返回值不参与模板参数推导**。解决上面问题，我们必须显式指定：

basics/max3.cpp
``` c++
::max<float>(2, 3.3);
::max<double>(3, 4.2);
```

### 缺省模板参数
我们也可以规避显式制定返回值类型：
basics/max4.hpp
``` c++
template <typename RT = double, typename T1, typename T2>
RT max(T1 a, T2 b) {
    return a < b ? b : a;
}
```
相比一般函数，我们发现：**缺省模板参数不一定在最后**。

### ReturnType的推导
回归到max4.hpp实现的max版本，它的局限性并没有解除，因为一旦T1和T2的类型是char*或string时，缺省默认型参并不能
起作用，从理论上，我们的设想RT的类型应该是根据T1和T2推导出来的，取其最大类型。借用C++11的尾置返回类型(Trailing teturn type syntax):

basics/max5.hpp
``` c++11
auto max(T1 a, T2 b) -> decltype(a < b ? b : a) {
    return a < b ? b : a;
}
```

或者

basics/max6.hpp
``` c++14
auto max(T1 a, T2 b) {
    return a < b ? b : a;
}
```

上面的`decltype(a < b ? b : a)`可换为`decltype(false ? b : a)`。

如果T1或T2中有引用类型，有什么隐患？(查阅std::common_type和std::decay)

### 重载
万事总有特例，模板函数也不例外，所以模板函数也是支持重载的，同时也可以重载一般函数。
具体到上面的max版本，我们可实现为:
basics/max7.cpp
``` c++
#include <cstring>
#include <string>

template <typename T>
T max(T a, T b) {
    return a < b ? b : a;
}

template <typename T>
T* max(T* a, T* b) {
    return *a < *b ? *b : *a;
}

inline char const* max(char const* a, char const* b) {
    return std::strcmp(b, a) < 0 ? a : b;
}
```

basics/max7.cpp
``` c++
#include "basics/max7.hpp"

int main ()
{
    int a = 3;
    int b = 4;

    auto m1 = ::max(a, b);  // max<int>

    std::string s1 = "hello";
    std::string s2 = "world";
    auto m2 = ::max(s1, s2);  // max<std::string>

    int* p1 = &a;
    int* p2 = &b;
    auto m3 = ::max(a, b); // max<int>

    char const* x = "hello";
    char const* y = "world";
    auto m4 = ::max(x, y);  // ...
}
```

在函数调用选择适配的重载函数，我们一般的原则是**对于同等适配的(模板)函数，编译器倾向选择可表示类型越窄的函数，，若仍不能选中，则报错(多个适配)**。

### 思考
1. 我们描述的始终都是针对二个元素，大家可以考虑实现支持多个参数或任意个的版本？
2. 回归到我们自己的项目中，你觉得哪些地方使用模板能够带来收益？