## 类模板
类似于函数模板，类也支持模板，主要用于容器，适于管理各种类型，其基本结构为：

``` c++
template <typename T1, typename T2, ...>
class SomeClass {
  public:
  ...
  private:

   T m;
};
```

### 基本用法
下面我将实现Stack范例来描述类模板相关知识。其实现如下：

basics/stack1.hpp
``` c++
#include <vector>
#include <cassert>

template <typename T>
class Stack {
 public:
  Stack() = default;

  Stack(Stack const&);

  void push(T const& elem);

  void pop();

  T const& top() const;

  bool empty() const {
      return elems.empty();
  }
 private:
  std::vector<T> elems;
};

template <typename T>
Stack<T>::Stack(Stack<T> const& other)
    : elems(other.elems) {}

template <typename T>
void Stack<T>::push(T const& elem) {
    elems.push_back(elem);
}

template <typename T>
void Stack<T>::pop() {
    assert(!elems.empty());
    elemes.pop_back();
}

template <typename T>
T const& Stack<T>::top() const {
    assert(!elems.empty());
    return elems.back();
}
```

从上面可发现类模板的写法和函数模板是相似的，需要注意的是在类内部和外部定义方法时
有所区别，在类外定义需要加上template关键字，同时上面定义的方法均不是模板方法。

相应的使用如下：
basics/stack1.cpp
``` c++
#include "basics/stack1.hpp"
#include <iostream>
#include <string>

int main() {
    Stack<int> intStack;
    Stack<std::string> stringStack;

    intStack.push(7);
    std::cout << intStack.top() << std::endl;

    stringStack.push("hello");
    std::cout << stringStack.top() << std::endl;

    return 0;
}
```

对比函数模板，我们必须显式实例化类模板，调用方法和普通类没有区别。

### 类方法按需实例化

为了减少模板实例化代码膨胀，编译器内部会优化掉所有未用到的模板方法，
具体到stack1.cpp中，我们只使用了push()和top()方法，所以编译器不会
生成pop()代码。除此之外，这还会带来另一个好处，比如我们修改Stack实现
如下：
basics/stack2.hpp
``` c++
template <typename T>
class Stack {
    ...
  public:
    void printOn(std::ostream& strm) const {
        for (T const& elem : elems)
          strm << elem << ' ';
    }
    ...
};
```
调用如下：
``` c++
Stack<std::pair<int, int>> ps;
ps.push({4, 5});
// std::cout << ps; (error)
```
我们知道std::pair<int, int>不支持operator<<,但当我们实例化Stack<std::pair<int, int>>
并没有报错，所以这样会更大程度使用模板的实现。

除了成员方法，友元函数也可以优化，见下节，虚函数会优化吗？？？

### 友元函数
为了完善Stack的答应功能，我们为Stack添加打印友元函数。
basics/stack3.hpp
``` c++
template <typename T>
class Stack {
    ...
    friend std::ostream& operator<<(std::ostream& strm, Stack<T> const& s) {
        for (auto const& elem : s.elems)
          strm << elem << ' ';
    }
    ...
};
```
一切都能完美运行，现在假设我们的operator<<很复杂，为了代码的整洁性，我们希望将其移出类外，
这时我们会发现问题会变得很复杂，因为友元函数的模板参数是和类依赖的，只有当模板类实例化后，才
有可能生成友函数。实际上，我们有两种方式解决这个问题：
1. 让友元函数的形参不依赖模板类的形参T。

``` c++
template <typename T>
class Stack {
    ...
    template <typename U>
    friend std::ostream& operator<<(std::ostream&, Stack<U> const&);
};

template <typename U>
std::ostream& operator<<(std::ostream& strm, Stack<U> const& s) {
    ...
}
```

2. 前置声明operator<<。

``` c++
template <typename T>
class Stack;

template <typename T>
std::ostream& operator<<(std::ostream&, Stack<T> const&);

template <typename T>
class Stack {
    ...
    friend std::ostream& operator<< <T>(std::ostream&, Stack<T> const&);
};
...
```

### 特化
若我们需要对当T = std::string时做特殊处理，也就是特化，我们可以这样实现：
basics/stack4.hpp
``` c++
#include "basics/stack1.hpp"

#include <deque>
#include <string>
#include <cassert>

template <>
class Stack<std::string> {
  public:
    void push(std::string const&);
    void pop();
    std::string const& top() const;
    bool empty() const {
        return elems.empty();
    }
  private:
    std::deque<std::string> elems;
};

void Stack<std::string>::push(std::string const& elem) {
    elems.push_back(elem);
}

void Stack<std::string>::pop() {
    assert(!elems.empty());
    elems.pop_back();
}

std::string const& Stack<std::string>::top() const {
    assert(!elems.empty());
    return elems.back();
}
```

注意对比特化后和之前的区别。

### 偏特化
我们也可以对类模板做偏特化处理，比如我们可以对Stack做关于指针的偏特化，具体见：
basics/stack5.hpp
``` c++
#include "basics/stack1.hpp"

template <typename T>
class Stack<T*> {
  public:
    void push(T*);
    T* pop();
    T* top() const;
    bool empty() const {
        return elems.empty();
    }
  private:
    std::vector<T*> elems;
};

template <typename T>
void Stack<T*>::push(T* elem) {
    elems.push_back(elem);
}

template <typename T>
T* Stack<T*>::pop() {
    assert(!elems.empty());

    T* p = elems.back();
    elems.pop_back();
    return p;
}

template <typename T>
T* Stack<T*>::top() const {
    assert(!elems.empty());
    return elems.back();
}
```

当然，偏特化也可以用在多模板参数的情形，如：

``` c++
template <typename T1, typename T2>
struct IsSame {
    static const bool value = false;
};

template <typename T>
struct IsSame<T, T> {
    static const bool value = true;
};
```

### 缺省模板参数

为了支持使用者定制不同的容器，我们可以实现为：

basics/stack6.hpp
``` c++
template <typename T, typename Cont = std::vector<T>>
class Stack {
  public:
    void push(T const& elem);
    ...
  private:
    Cont elems;
};

template <typename T, typename Cont>
void Stack<T, Cont>::push(T const& elem) {
    elems.push_back(elem);
}
...
```

basics/stack6.cpp
```
#include "basics/stack6.hpp"

#include <deque>

int main()
{
    Stack<int> intStack;
    Stack<double, std::deque<double>> dblStack;

    intStack.push(3);

    dlbStack.push(2.7);
    return 0;
}
```

### 模板成员函数

因为int和double可以相互转换，所以我们希望Stack<int>和Stack<double>可以相互赋值，
但我们现在的实现暂时还不支持，这里我们可以引入模板成员函数解决。

basics/stack7.hpp
``` c++
#include <vector>
#include <cassert>

template <typename T>
class Stack {
  public:
    ...
    template <typename U>
    Stack(Stack<U> const& other) {
        static_assert(std::is_convertible(U, T), "U must can convert to T");
        for (auto elem : other.elems)
            elems.push_back(elem);
    }
    ...
};
```

现在我们就可以像如下调用：

``` c++
Stack<Int> intStack;

intStack.push(3);

Stack<double> dblStack(intStack);
```

注意虚函数不可以实现为模板成员函数。