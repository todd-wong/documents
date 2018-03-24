# WebAssembly原理和应用介绍

## 背景

&emsp;&emsp;在介绍WebAssembly之前，先回顾下当前web遇到的问题，随着html5的普及，支持的功能也越来越多，像WebGL、WebVR、Video、Media等，应用也越来越复杂，相应地对性能要求也迫在眉睫，因此各大浏览器厂商都在不断尝试优化JS Engine(V8/SpiderMonkey/Chakra/JavaScriptCore)，以V8 pipeline为例，
![](/assets/v8_pipeline.png)

&emsp;&emsp;在WebAssembly诞生以前，虽有google的[Google Native Client](https://en.wikipedia.org/wiki/Google_Native_Client)(NaCl),和Mozilla的[asm.js](https://en.wikipedia.org/wiki/Asm.js)，二者对web端性能，相对native code都有长足的长进，但二者皆有其局限性，前者仅限chrome/chromium支持，后者虽然兼容其他浏览器，但和native code仍有所的差距，详细的数据可参考<https://github.com/kripken/emscripten/blob/incoming/docs/paper.pdf>。基于以上的考量，正式引出我们将介绍的WebAssembly，一下是摘自wikipedia的定义：



&emsp;&emsp;WebAssembly起初实现是基于asm.js的feature，

## 定义
&emsp;&emsp;“WebAssembly或称wasm是一个实验性的低级编程语言，应用于浏览器内的客户端。WebAssembly是便携式的抽象语法树，被设计来提供比JavaScript更快速的编译及运行。WebAssembly将让开发者能运用自己熟悉的编程语言（最初以C/C++作为实现目标）编译，再藉虚拟机引擎在浏览器内运行。WebAssembly的开发团队分别来自Mozilla、Google、Microsoft、Apple，代表着四大网络浏览器Firefox、Chrome、Microsoft Edge、Safari。2017年11月，所有以上四个浏览器都开始实验性的支持WebAssembly。“[^1]

## 原理
![](/assets/v8_pipeline_wasm.png)

## 特点
* ####快速高效####
    wasm被编码为二进制格式，占用空间小，下载快，同时解析快。比native code慢20%，比asm.js快5%以上。

* ####安全####
    1、 wasm运行在沙箱环境中，强制检测同源和安全策略。
    2、 对比C/C++，指针可以访问整个进程用户态的堆栈，而wasm execution stack则完全和wasm程序隔离，防止其修改本地变量。
![](/assets/wasm_memory_mode.png)

* ####可调式####
    wasm可通过[Wabt工具](https://github.com/WebAssembly/wabt)转为[wat](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format)的可读文本格式，也支持在devtools动态调试。

## 使用

## 应用

## 展望
WebAssembly现在仍处于快速发展阶段，像多线程、尾递归优化
[^1]: https://zh.wikipedia.org/wiki/WebAssembly