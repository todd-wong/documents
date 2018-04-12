
test.cc:

``` cplusplus

#include <iostream>
#include <string>

#include <emscripten/bind.h>

int add(int i, int j) {
  return i + j;
}

int add(int i, int j, int k) {
  return i + j + k;
}

uintptr_t getPtr() {
  static int s = 7;
  return (uintptr_t)&s;
}

//适用于int*,double*...
void printPtr(uintptr_t ptr) {
  std::cout << "printPtr:" << *(int*)ptr << std::endl;
}

//适用于char*
void printStr(const char* str) {
  std::cout << "str:" << str << std::endl;
}

struct Point {
  int x;
  int y;
};

struct Location {
  int x;
  int y;
};

Location convert(Point p) {
  return {
           p.x + 10,
           p.y + 10
         };
}

void multiAndCallback(double x, double y, emscripten::val callback) {
  if (callback.typeof().as<std::string>() != "function")
    return;

  callback(x * y);
}

std::string test(std::string str) {
  return str + " through test";
}

std::function<std::string(std::string)> getFunc() {
  return test;
}

std::string callFunctor(const std::string& str, std::function<std::string(std::string)> func) {
  return func(str);
}

class Inner {
 public:
  Inner(const std::string& name)
      : name_(name) {
    std::cout << "Inner constructor.\n";
  }

  virtual ~Inner() {
    std::cout << "Inner destructor.\n";
  }

  virtual std::string Name() const {
    return name_;
  }

  virtual void Pure() = 0;

  private:

   std::string name_;
};

class Middle : public Inner {
 public:
  static Middle* Create() {
    return new Middle("Middle", 50);
  }

  explicit Middle(const std::string& name, int account)
      : Inner(name)
      , account_(account) {
    std::cout << "Middle constructor.\n";
  }

  ~Middle() override {
    std::cout << "Middle destructor.\n";
  }

  void Pure() override {
    std::cout << "Middle::PureMethod called.\n";
  }

  int Add(int value) {
    return account_ += value;
  }

  int Diff(Middle* rhs) {
    return account_ - rhs->account_;
  }
  
#if 0
  int Diff(const Middle& rhs) {
    return account_ - rhs.account_;
  }
#else
  int Diff(Middle rhs) {
    return account_ - rhs.account_;
  }
#endif

 private:
  int account_;
};

class Outer : public Middle {
 public:
  Outer(const std::string& name)
      : Middle(name, 100) {
    std::cout << "Outer constructor.\n";
  }

  ~Outer() {
    std::cout << "Outer destructor.\n";
  }

  static std::shared_ptr<Outer> Create(const std::string& name) {
    return std::make_shared<Outer>(name);
  }

  Outer* Get() {
    return this;
  }

  void Pure() override {
    std::cout << "Outer Pure method called.\n";
  }
};

EMSCRIPTEN_BINDINGS(my_module) {
using namespace emscripten;

value_array<Point>("Point")
    .element(&Point::x)
    .element(&Point::y)
    ;

value_object<Location>("Location")
    .field("x", &Location::x)
    .field("y", &Location::y)
    ;

function("add", select_overload<int(int,int)>(&add));
function("add_3", select_overload<int(int,int,int)>(&add));
function("getPtr", &getPtr);
function("printPtr", &printPtr);
function("printStr", optional_override([](const std::string& str){
    printStr(str.c_str());
}));
function("convert", &convert);
function("multiAndCallback", &multiAndCallback);

class_<std::function<std::string(std::string)>>("StringFunctorString")
    .constructor<>()
    .function("opcall", &std::function<std::string(std::string)>::operator())
    ;
function("getFunc", &getFunc);
function("callFunctor", &callFunctor);

class_<Inner>("Inner")
    .function("Name", &Inner::Name)
    .function("Pure", &Inner::Pure, pure_virtual())
    ;

class_<Middle, base<Inner>>("Middle")
    .constructor<const std::string&, int>()
    .function("Diff_p", select_overload<int(Middle*)>(&Middle::Diff), allow_raw_pointers())
#if 0
    .function("Diff_c", select_overload<int(const Middle&)>(&Middle::Diff))
#else
    .function("Diff", select_overload<int(Middle)>(&Middle::Diff))
#endif
    .function("Add", &Middle::Add)
    .class_function("Create", &Middle::Create, allow_raw_pointers())
    ;

class_<Outer, base<Middle>>("Outer")
    .smart_ptr_constructor("std::shared_ptr<Outer>", &Outer::Create)
 // .smart_ptr<std::shared_ptr<Outer>>("std::shared_ptr<Outer>")
 // .constructor(&Outer::Create)
    .function("Get", &Outer::Get, allow_raw_pointers())
    .property("name", &Outer::Name)
    ;
}

```

编译命令为：
``` shell

$>em++ test.cc -std=c++11 --bind -Os -s ASSERTIONS=1 -s WASM=1 -s ALLOW_MEMORY_GROWTH=1 -o index.html

```

### 普通函数(function)
原型为：
``` cplusplus

template<typename ReturnType, typename... Args, typename... Policies>
void function(const char* name, ReturnType (*fn)(Args...), Policies...);

```

1、由于JavaScript没有重载的概念，需要用select_overload选择特定的Signature，类方法类似。
2、如果只是需要单纯override被binding方法的Signature，可以使用optional_override附带无捕获状态的lamda函数。
3、基本类型的裸指针（void\*/int\*/double\*/long\*等），binding编译会报错，可像上述利用uintptr_t方式规避，或者单纯的char*，可以用printStr的方式。

### 类(class_)
原型为：
``` cplusplus

template<typename ClassType, typename BaseSpecifier = internal::NoBaseClass>
class class_ {
 public:
  EMSCRIPTEN_ALWAYS_INLINE explicit class_(const char* name)...
};

```
1、构造函数使用constructor或者smart_ptr_constructor。
2、类方法使用function。
3、静态方法使用class_function。
4、类继承可通过指定BaseSpecifier基类。
5、若方法为纯虚函数，附带pure_virtual()。
6、利用select_overload选择特定的重载方法。

### 值类型(value_array和value_object)
原型为：
``` cplusplus

template<typename ClassType>
class value_array : public internal::noncopyable {
 public:
  value_array(const char* name)...
};

template<typename ClassType>
class value_object : public internal::noncopyable {
 public:
  value_object(const char* name)...
};

```
1、适用于值传递，注意不可binding到指针类型。

### 指针(allow_raw_pointers)
1、若函数或方法中带有指针类型的参数火返回值，需带allow_raw_pointers()。

### callback
1、js callback参考multiAndCallback。
2、c/c++ callback参考callFunctor。

### 智能指针(smart_ptr)
1、常规用法参考上例中的Outer::Create。
2、自定义智能指针需特化smart_ptr_trait。

### 重载函数(select_overload)、覆写binding函数(optional_override)