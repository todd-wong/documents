## 介绍
&emsp;&emsp;“WebAssembly或称wasm是一个实验性的低级编程语言，应用于浏览器内的客户端。WebAssembly是便携式的抽象语法树，被设计来提供比JavaScript更快速的编译及运行。WebAssembly将让开发者能运用自己熟悉的编程语言（最初以C/C++作为实现目标）编译，再藉虚拟机引擎在浏览器内运行。WebAssembly的开发团队分别来自Mozilla、Google、Microsoft、Apple，代表着四大网络浏览器Firefox、Chrome、Microsoft Edge、Safari。2017年11月，所有以上四个浏览器都开始实验性的支持WebAssembly。“[^1]（摘自wiki）

&emsp;&emsp;WebAssembly是延续Mozilla的[Asm.js](https://en.wikipedia.org/wiki/Asm.js)，更深一步的优化。支持语言包括C/C++/Rust。

## 历程
* 2015.4 - WebAssembly Community Group成立
* 2015.6 - WebAssembly被声明
* 2017.3 - MVP(Minimum viable product)被主流浏览器支持

## 特点
* 快速高效([stack machine](http://webassembly.org/docs/semantics/))，二进制格式，解析快，比asm.js快5%以上，比native code慢20%。
* 安全(沙箱、Memory Model)。
* 可调试([wat文本格式](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format))。
* 纳入W3C标准，得到主流浏览器支持，包括(Firefox、Chrome、Safri、Edge)。

![](/assets/v8_pipeline_wasm.png)
![](/assets/wasm_memory_mode.png)

## 范例
我们先尝试最原始的做法，直接写WebAssembly，具体如下：
``` scheme
(module
  (import "console" "log" (func $log (param i32)))
  (func $add (param $lhs i32) (param $rhs i32) (result i32)
    get_local $lhs
    get_local $rhs
    i32.add)
  (func $innerlog (param $lhs i32)
    get_local $lhs
    call $log
  )
  (export "add" (func $add))
  (export "log" (func $innerlog))
)
```
上述代码保存为add.wat，利用wat2wasm转为wasm，
> $> wat2wasm add.wat -o add.wasm

在当前目录创建相应的index.html为：
``` html
<!doctype html>

<html>

  <head>
    <meta charset="utf-8">
    <title>WASM test</title>
  </head>

  <body>
    <script>
      var importObject = {
        console: {
          log: function(arg) {
            console.log(arg);
          }
        }
      };

      fetch('add.wasm').then(response =>
        response.arrayBuffer()
      ).then(bytes =>
        WebAssembly.instantiate(bytes, importObject)
      ).then(result => {
          var add = result.instance.exports.add;
          var log = result.instance.exports.log;
          log(add(3, 4));
        }
      );
    </script>
  </body>

</html>
```
启动server，
> $> emrun --no_browser --port 8888 index.html 或 python -m SimpleHTTPServer 8888

打开chrome浏览器，访问<http://localhost:8888/>，并打开Devtools，会显示：
> \> 7
![](/assets/loading_webassembly.jpg)

&emsp;&emsp;从例子中，我们发现即使实现简单的整数相加，写出add.wat都非常麻烦，尤其对于大型项目，有没有更好的方法呢？。。。

注：上述实例需要用到工具[wabt](https://github.com/WebAssembly/wabt)，具体安装参考链接。至于WebAssembly API的使用可以参看<https://developer.mozilla.org/zh-CN/docs/WebAssembly>，WebAssembly文本表示（S-expressions）参考<https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format>

## [开发工具链](https://github.com/WebAssembly/binaryen)
对于C/C++代码移植，现在有两种选择：
* C/C++ => asm2wasm =>WebAssembly(with emcc)
* C/C++ => WebAssembly LLVM backend => s2wasm => WebAssembly(without emcc)

基于asm.js项目比较成熟，且是官方推荐，主流选择第一种方式——[Emscripten](http://kripken.github.io/emscripten-site/docs/introducing_emscripten/index.html)(具体安装见
<https://github.com/kripken/emscripten>)。

![](/assets/EmscriptenToolchain.png)

## [Emscripten](http://kripken.github.io/emscripten-site/docs/introducing_emscripten/index.html)
Emscripten是从LLVM到JavaScript的编译器开源项目，功能包括：
* 翻译C/C++代码，转为JavaScript输出。
* 翻译任何能够转为LLVM bitcode的其他代码为JavaScript。
* 翻译任何其他语言的runtimes(C/C++实现)为JavaScript，进而能够运行其他语言，包括Python和Lua。

Emscripten[支持的库](http://kripken.github.io/emscripten-site/docs/introducing_emscripten/about_emscripten.html)包括c标准库、c++标准库、c++异常、SDL、OpenGL ES2.0。

## [移植限制](http://kripken.github.io/emscripten-site/docs/porting/guidelines/portability_guidelines.html)
* 代码可移植性限制，以下情况不支持：
    * 使用多线程且共享状态。
    * 依赖大端处理器。
    * 依赖x86内存对齐行为。
    * 使用本地环境底层features。
    * 需要获取寄存器或堆栈内容。
    * 内联本地汇编代码。
* [API限制](http://kripken.github.io/emscripten-site/docs/porting/guidelines/api_limitations.html)
    * 网络接口必须异步。
    * 支持文件系统接口。
    * [应用程序主循环](http://kripken.github.io/emscripten-site/docs/porting/emscripten-runtime-environment.html#browser-main-loop)。
![](http://kripken.github.io/emscripten-site/_images/FileSystemArchitecture.png)

## C/C++和JavaScript代码交互
* ###js调用c函数
    * [使用ccall或cwrap](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#calling-compiled-c-functions-from-javascript-using-ccall-cwrap)。
    * [直接调用编译的C/C++代码](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#call-compiled-c-c-code-directly-from-javascript)。

下面以官方例子为例：
``` c
#include <math.h>

extern "C" {

int int_sqrt(int x) {
  return sqrt(x);
}

}
```
保存为hello_function.cpp，
> $>emcc hello_function.cpp -o function.html -s EXPORTED_FUNCTIONS='["_int_sqrt"]' -s EXTRA_EXPORTED_RUNTIME_METHODS='["ccall", "cwrap"]' -s WASM=1

运行
> $>emrun --no_browser --port 8888 function.html

打开Devtools调试，可得如下结果：
![](/assets/result.png)


* ###[通过Embind调用C++类](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/embind.html)


``` cplusplus
#include <emscripten/bind.h>
  
class MyClass {
public:
  MyClass(int x, std::string y)
    : x(x)
    , y(y)
  {}

  void incrementX() {
    ++x;
  }

  int getX() const { return x; }
  void setX(int x_) { x = x_; }

  static std::string getStringFromInstance(const MyClass& instance) {
    return instance.y;
  }

private:
  int x;
  std::string y;
};

// Binding code
EMSCRIPTEN_BINDINGS(my_class_example) {
  emscripten::class_<MyClass>("MyClass")
    .constructor<int, std::string>()
    .function("incrementX", &MyClass::incrementX)
    .property("x", &MyClass::getX, &MyClass::setX)
    .class_function("getStringFromInstance", &MyClass::getStringFromInstance)
    ;
}

```
保存为myclass.cpp，运行
> $>emcc myclass.cpp --bind -o myclass.html -s WASM=1
> $>emrun --no_browser --port 8888 myclass.html

打开Devtools，结果如下：
![](/assets/embind_result.png)

* ###[通过WebIDL Binder调用C++类](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/WebIDL-Binder.html)

``` IDL
interface Foo {
    void Foo(long v);
    long getVal();
    void setVal(long v);
    void print();
};
```
保存为myclass.idl，运行
> $>python ~/emsdk/emscripten/1.37.36/tools/webidl_binder.py myclass.idl glue

``` cplusplus

#ifndef __MYCLASS__
#define __MYCLASS__

class Foo {
 public:
  Foo(int val);
  int getVal();
  void setVal(int val);
  void print();

 private:
  int val_;
};

#endif

```
保存为myclass.h，
``` cplusplus

#include "myclass.h"
  
#include <iostream>

Foo::Foo(int val) : val_(val) {}

int Foo::getVal() {
  return val_;
}

void Foo::setVal(int val) {
  val_ = val;
}

void Foo::print() {
  std::cout << "Hello WebIDL.\n";
}

```
保存为myclass.cpp，
``` cplusplus

#include "myclass.h"
#include "glue.cpp"

```
保存为myclass_wrapper.cpp，运行
> $>emcc myclass.cpp myclass_wrapper.cpp --post-js glue.js -o output.html -s WASM=1
> $>emrun --no_browser --port 8888 output.html

最终结果如下：
![](/assets/idl_result.png)

* ###C/C++调用JavaScript
    * ####[emscripten_run_script](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#interacting-with-code-call-javascript-from-native)
    * ####[EM_ASM](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#interacting-with-code-call-javascript-from-native)
    * ####[JavaScript实现C函数](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#implement-a-c-api-in-javascript)

``` cplusplus

extern void my_js(void);
  
int main()
{
  my_js();
  return 1;
}

```
保存为main.c，
``` javascript

mergeInto(LibraryManager.library, {
  my_js: function() {
    console.log('this is my js run.');
  }
});

```
保存为my_lib.js，运行：
> $>emcc main.c -o main.html -s WASM=1 --js-library my_lib.js

最终结果如下：
![](/assets/cfun_result.png)

## [与页面元素交互](http://kripken.github.io/emscripten-site/docs/api_reference/index.html)
* ####html5(参考html5.h)
* ####其他可查询(emscripten.h/val.h/bind.h)

实例参考：<http://www.rossis.red/wasm.html>

## 适用领域
* CPU密集型任务（数值计算、游戏、图片处理等）

参考资料：
<https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79>
<https://github.com/mbasso/awesome-wasm>

[^1]: https://zh.wikipedia.org/wiki/WebAssembly