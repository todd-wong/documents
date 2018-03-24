v8解析过程：
![](/assets/v8_pipeline_wasm.png)
![](/assets/EmscriptenToolchain.png)
## 应用场景
* cpu密集型任务，例如：数值计算、像素操作等。
* gpu渲染，典型场景包括游戏、VR、流媒体等。

## 代码保密性
wasm不具有代码保密性，仅仅是增加难度。虽然其为二进制格式，但可以通过[binaryen](https://github.com/WebAssembly/binaryen)转为wast文本格式，或者是通过devtools直接查看S-表达式。

## 主流的toolchain
* Emscripten+Wabt。
* Clang+Binaryen。

## 代码限制
1、 多线程且利用共享变量。
2、 应用结果依赖大端架构或者是x86内存对齐行为。

* wasm比native code只慢20%，且现在还在快速发展中。
* wasm相比asm.js是二进制格式，体积更小，下载更快，解析更快，大概比asm.js提升了5%的速率。
