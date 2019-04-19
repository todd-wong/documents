## 一处定义原则(The One-Define Rule)
* 翻译单元(Translation Units)

## 名称的定义
* 受限名称(Qualified name): 名称前有作用域符(::)和操作符(./->)。比如：S::x, this->f, (*this).f等。
* 非受限名称(Unqualified name)：受限名称的反义词。
* 依赖名称(Dependent name): 依赖于模板参数的名称。比如在模板类的成员函数this->f。
* 非依赖参数(Nondependent name): 依赖名称的反义词。

## typename的使用
typename可以用来定义后面跟的模板参数，也可以指定随后待定的类型，被指定的类型名称需要满足以下条件：

* 必须受限名称。
* 不可以使用在基类特化或者是构造函数的初始化列表中。
* 依赖于模板参数，作为未知的特化类型。

```c++
template <typename T>     // #1(required)
struct S : typename X<T>::Base {   // #2(disallowed)
  S() : typename X<T>::Base(typename X<T>::Base(0)) {  // #3(dissallowed), #4(required)
  }
  
  typename X<T> f()  {  // #5(disallowed)
    typename X<T>::C* p;  // #6(required)
    X<T>::D * q;  // #7(multiplication)
  }
  
  typename X<int>::C* s;  // #8(Optional)
  
  using Type = T;
  using OtherType = typename S<T>::Type;  // #9(required)
};
```

## 参数依赖查找(Argument-Dependent Lookup)
```c++
namespace math {
  class Complex {
    ...
  };
  
  bool operator < (Complex const&, Complex const&);
}

bool f(math::Complex const& a, math::Complex const& b) {
  return a < b;
}
```

```c++
#include <iostream>

namespace X {
template<typename T> void f(T);
}

namespace N {
using namespace X;
enum E {e1};
void f(E) {
  std::cout << "N::f(N::E) called\n";
}
}

void f(int) {
  std::cout << "::f(int) called\n";
}

int main()
{
  ::f(N::e1);  // qualified function name: no ADL
  f(N::e1); // use ADL
}
```
考虑上述代码，operator <声明在namespace math，但在全局函数f中能调用到，中间调用就使用了ADL。

```c++
template <typename T>
class C {
  ...
  friend void f();
  friend void f(C<T> const&);
  ...
};

void g(C<int>* p) {
  f();  // ERROR
  f(*p);
}
```
## 实例化
* 按需实例化(On-Demand)

```c++
template <typename T>
class C;

C<int>* p = 0;

template <typename T>
class C {
 public:
  void f();
};

void g(C<int>& c) {
  c.f();
}

template <typename T>
void C<T>::f() {}
```

* 延迟实例化(Lazy Instantiation)

```c++
template <typename T> T f(T p) {
  return 2 * p;
}

decltype(f(2)) x = 2;  // just partial instantion for declare, not include body

template <typename T> class Q {
  using Type = typename T::Type;
};

Q<int>* p = 0;  // partial instantion
```

## 二阶段查找(Two-Phase Lookup)
1.  During the first phase, while parsing a template, nondependent names are looked up using both the ordinary lookup rules and, if applicable, the rules for ADL. Unqualified dependent names are looked up using the ordinary lookup rules, but the result of the lookup is not cnsidered complete util an additional lookup is performed in the second phase(when the template is instantiated).
2. During the second phase, while instantiating a template at POI, dependent qualified names are looked up, and an additional ADL is performed for the unqualified dependent names that were looked up using ordinary lookup in the first phase.

```c++
template <typename T>
class Base {
 public:
  void bar();
};

template <typename T>
class Derived : Base<T> {
 public:
  void foo() {
    bar();  // ERROR
    this->bar();
    Base<T>::bar();
  }
};
```

## 实例化点(Point Of Instantiation)
```c++
class MyInt {
 public:
  MyInt(int i);
};

MyInt operator-(MyInt const&);

bool operator > (MyInt const&, MyInt& const);

using Int = MyInt;  // if using Int = int ?

template <typename T>
void f(T i) {
  if （i > 0）
    g(-i);
}

// #1
void g(Int) {
  // #2
  f<Int>(42);
  // #3
}
// #4 POI
```
函数模板的POI在调用后最近的namespace。

```c++
template <typename T>
void f1(T x) {
  g1(x);
}

void g1(int) {}

int main() {
  f1(7);
}
```

```c++
template <typename T>
class S {
 public:
  T m;
};

// #1 POI
unsigned long h() {
  // #2
  return (unsigned long)sizeof(S<int>);
  // #3
}
// #4
```

类模板的POI在调用前最近的namespace。

## 实例化实现方案
* 贪婪实例化(Greedy Instatiation)
* 询问实例化(Queried Instatiation)
* 迭代实例化(Iterated Instatiation)

## SFINAE(Substitution Failure Is Not An Error)

```c++
template <typename T, unsigned N>
T* begin(T (&array)[N]) {
  return array;
}

template <typename Container>
typename Container::iterator begin(Container& c) {
  return c.begin();
}

int main()
{
  std::vector<int> v;
  int a[0];
  
  ::begin(v);
  ::begin(a);
}
```
