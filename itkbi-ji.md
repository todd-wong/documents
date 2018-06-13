## 介绍

API参考地址：https://itk.org/Doxygen/index.html
## ITK目录结构
The ITK repository contains the following subdirectories:

* ITK/Modules — the heart of the software; the location of the majority of the source code.
* ITK/Documentation — migration guides and Doxygen infrastructure.
* ITK/Examples — a suite of simple, well-documented examples used by this guide, illustrating important ITK concepts.
* ITK/Testing — a collection of the MD5 files, which are used to link with the ITK data servers to download test data. This test data is used by tests in ITK/Modules to produce the ITK Quality Dashboard using CDash. (see Section ?? on page ??.)
* Insight/Utilities — the scripts that support source code development. For example, CTest and Doxygen support.
* Insight/Wrapping — the wrapping code to build interfaces between the C++ library and various interpreted languages (currently Python is supported).

The source code directory structure—found in ITK/Modules—is the most important to understand.

* ITK/Modules/Core — core classes, macro definitions, type aliases, and other software constructs central to ITK. The classes in Core are the only ones always compiled as part of ITK.
* ITK/Modules/ThirdParty — various third-party libraries that are used to implement image file I/O and mathematical algorithms. (Note: ITK’s mathematical library is based on the VXL/VNL software package9 .)
* ITK/Modules/Filtering — image processing filters.
* ITK/Modules/IO — classes that support the reading and writing of images, transforms, and geometry.
* ITK/Modules/Bridge — classes used to connect with the other analysis libraries or visualization libraries, such as OpenCV10 and VTK11 .
* ITK/Modules/Registration — classes for registration of images or other data structures to each other.
* ITK/Modules/Segmentation — classes for segmentation of images or other data structures.
* ITK/Modules/Video — classes for input, output and processing of static and real-time data with temporal components.
* ITK/Modules/Compatibility — collects together classes for backwards compatibility with ITK Version 3, and classes that are deprecated – i.e. scheduled for removal from future versions of ITK.
* ITK/Modules/Remote — a group of modules distributed outside of the main ITK source repository (most of them are hosted on github.com) whose source code can be downloaded via CMake when configuring ITK.
* ITK/Modules/External — a directory to place in development or non-publicized modules.
* ITK/Modules/Numerics — a collection of numeric modules, including FEM, Optimization, Statistics, Neural Networks, etc.