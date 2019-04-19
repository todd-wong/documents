原文地址：https://blogs.msdn.microsoft.com/the1/2004/05/07/how-would-you-get-the-count-of-an-array-in-c-2/

这个问题可简单描述为：
给定一个c++ 数组（如：int x[10]），如何获取它的元素个数？

一种显而易见的方法是使用宏（定义1）：

```
#define countof( array ) (sizeof( array )/sizeof( array[0] ))
```

我不能说这种方法不正确，因为在给定一个数组，他确实能获取其元素个数。然而，当提供的对象不是数组时，同样的表达式会给你一些不真实的结果。例如，如果你提供

```
int* p;
```

这时当运行在int指针和int拥有相同大小（如win32平台）机器上，countof( p )总是返回1。

这种宏表示也可以错误接受任何有operator[]方法的类对象。例如，假设你写的代码如下：

```
class IntArray {
 private:
  int* p;
  size_t size;

 public:
  int &operator[](size_t i);
} x;
```
此时，sizeof(x)将返回x对象的大小，而不是x.p指向buffer的大小。因此，通过countof(x)，你将不能获取正确值。

所以我们可以断言定义1不是一种好的方法，因为编译器不能阻止你错用它。它不能强制只传递数组。

有没有更好的选择呢？

是的，如果我们希望编译器保证coutof的参数总是数组，我们必须找到一个只允许数组的上下文，且同时必须拒绝所有非数组的表达式。

一些初学者可能尝试这样（定义2）：

```
template <typename T, size_t N>

size_t countof(T array[N])
{
  return N;
}
```
他们认为，这个模板函数将接受大小为N的数组，并返回N。

不幸的是，这将编译不过，因为C++对待数组形参等同于指针参数，也就是上述的定义等同于：

```
template <typename T, size_t N>
size_t countof(T* array)
{
  return N;
}
```
现在显然函数主体没方法知道N是多少。

然而，如果传入的参数是数组引用，此时编译器保证实参的大小匹配形参。这意味我们小改定义2，使其工作（定义3）：

```
template <typename T, size_t N>
size_t countof(T (&array)[N])
{
  return N;
}
```
这个countof能工作很好，且你不能传递指针参数。但是，它是个函数，而不是宏。这意味着你不能用它作为编译常数。尤其，你不能写代码如下：

```
int x[10];

int y[2*countof(x)];
```

我们还有其他方法吗？

有人（我不知道具体是谁）提出更好的主意：
从函数主体移出N至返回类型，此时我们能获取N，而不需要实际调用函数。


确切地说，我们必须让函数返回一个数组引用，因为C ++不允许直接返回数组。

具体实现：

```
template <typename T, size_t N>

char (&_ArraySizeHelper(T(&array)[N]))[N];

#define countof(array) (sizeof(_ArraySizeHelper(array)))
```

不可否认，语法看起来很糟糕。事实上，有必要做一些解释。

首先，顶层的表述

```
char (&_ArraySizeHelper(...))[N];
```
表示“_ArraySizeHelper是返回大小为N的char数组引用的函数”。

而后，函数参数是

```
T (&array)[N]
```
表示引用元素个数为N的T数组。

最后，coutof被定义为函数_ArraySizeHelper返回值大小。注意我们没必要定义_ArraySizeHelper()，一声明足矣。

利用新的定义，

```
int x[10];
int y[2*countof(x)]; //twice as big as x
```
正如我们希望的，变为有效定义。
