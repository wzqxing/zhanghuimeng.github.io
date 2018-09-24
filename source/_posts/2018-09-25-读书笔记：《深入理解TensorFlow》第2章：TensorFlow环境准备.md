---
title: 读书笔记：《深入理解TensorFlow》第2章：TensorFlow环境准备
urlname: reading-report-tensorflow-in-depth-ch-2-prepare-for-tensorflow-summary
toc: true
date: 2018-09-25 02:06:08
updated: 2018-09-25 02:06:08
tags: [Reading Report, TensorFlow]
---

书：[深入理解TensorFlow架构设计与实现原理](https://github.com/DjangoPeng/tensorflow-in-depth)

---

本章主要介绍了TensorFlow的安装方法和基本结构。基本结构里大部分内容我也不是很明白……

## TensorFlow的五种安装方法

1. 用包管理工具Anacoda安装
2. 用原生pip安装
3. 用virtualenv安装
4. 自己下载源码进行编译并得到whl包
5. 用docker运行含有TensorFlow二进制包的容器

如果需要使用GPU版本的TensorFlow，则需要安装CUDA、cuDNN和libcupti-dev开发库。（因为这些是TensorFlow的外部依赖项。）

我是用virtualenv安装的。本来打算用原生pip，但PyCharm自动提供virtualenv环境，那就直接这么装好了。

## TensorFlow的几种重要依赖项

1. Bazel：TensorFlow软件构建
2. Protocol Buffers数据结构序列化工具和gRPC通信库：TensorFlow的进程间通信机制，以及Serving等周边组件的交互机制所依赖的框架
3. Eigen线性代数计算库：主要用于在CPU和OpenCL GPU设备上实现TensorFlow的计算类操作
4. CUDA统一计算设备架构：用于并行计算（是不受Bazel管理的外部依赖项）

## 源代码结构

### 根目录

根目录是一个Bazel项目的工作空间，包含了TensorFlow的所有源代码、Bazel构建规则文件，以及一些辅助脚本。

![2018.9.22的根目录截图](root.png)

* `tensorflow/`：TensorFlow项目自身的源代码
* `third_party/`：部分第三方源代码以及针对第三方项目的Bazel构建规则文件
* `tools/`：Bazel构建过程所需的环境配置脚本
* `utils/`：现已删除

### tensorflow目录

这一目录下的源文件几乎实现了TensorFlow的全部功能，同时体现了TensorFlow的整体模块布局。

![2018.9.22的tensorflow目录截图](tensorflow.png)

* `c/`：C语言应用层API，亦作为C、C++以外的其他语言应用层API的实现基础
* `cc/`：C++语言应用层API
* `compiler/`：XLA（Accelerated Linear Algebra）编译优化组件的源代码。
* `contrib/`：社区托管的第三方贡献组件
* `core/`：TensorFlow核心运行时库的源代码，主要使用C++语言实现
* `doc_src/`：TensorFlow软件文档（即TensorFlow官方网站文档）的Markdown源代码
* `examples/`：TensorFlow应用开发示例代码
* `g3doc/`：旧的文档目录，已弃用
* `go/`：Go语言应用层API
* `java/`：Java语言应用层API
* `python/`：Python语言应用层API
* `security/`：<del>显然写书的时候还没有这个文件夹</del>
* `stream_executor/`：StreamExecutor库的源代码，主要使用C++语言实现，用于管理CUDA GPU上的计算。
* `tensorboard/`：TensorBoard组件的源代码，主要使用Python语言实现，用于深度学习过程可视化；现已移入单独仓库[tensorboard](https://github.com/tensorflow/tensorboard)中
* `tools/`：TensorFlow构建和运行时使用的工具程序或脚本
* `BUILD`：Bazel构建规则文件，用于构建TensorFlow核心运行时库等组件
* `tensorflow.bzl`：Bazel构建过程所需的辅助脚本，主要用于定义TensorFlow特有的构建规则
* `workspace.bzl`：Bazel构建过程所需的辅助脚本，主要用于定义外部依赖项的下载规则

### tensorflow/core目录

![2018.9.22的tensorflow/core目录截图](tensorflow_core.png)

* `api_def/`：<del>显然写书的时候还没有这个文件夹</del>
* `common_runtime/`：核心库的公共运行时源代码，实现了TensorFlow数据流图计算的主要逻辑
* `debug/`：用于核心库调试的组件
* `distributed_runtime/`：核心库的分布式运行时源代码，实现了TensorFlow分布式运行模式的主要逻辑
* `example/`：使用Protocol Buffers创建自定义数据结构并访问序列化文件的示例代码
* `framework/`：核心库的框架性组件，包含TensorFlow编程框架中主要抽象的C++或Protocol Buffers定义
* `graph/`：数据流图相关抽象和工具类的源代码
* `grappler/`：Grappler优化器（一种基于硬件使用成本分析的数据流图优化器）的源代码
* `kernels/`：数据流图操作（Op）针对各类计算设备实现的核函数源代码
* `lib/`：公共基础库，涉及通用数据结构、常用算法的实现，以及多种图形、音频格式的访问接口类
* `ops/`：数据流图操作的接口定义源代码
* `platform/`：用于访问特定操作系统或云服务接口的平台相关代码
* `protobuf/`：数据流图基本抽象以外的序列化数据结构的Protocol Buffers源代码，例如gRPC接口定义
* `public/`：对应用层可见的公开接口的头文件
* `user_ops/`：用于存放用户自行开发的数据流图操作，包含一组示例代码
* `util/`：核心库内部使用的多种实用工具类或函数的集合，例如用于解析命令行参数和访问环境变量的工具

### tensorflow/python目录

![2018.9.22的tensorflow/python目录截图](tensorflow_python.png)

* `autograph/`：？
* `client/`：TensorFlow主-从模型中的客户端组件，主要包括会话抽象，用于维护数据流图计算的生命周期
* `compat/`：？
* `data/`：？
* `debug/`：用于Python应用程序调试的组件
* `distribute/`：？
* `eager/`：？
* `estimator/`：各类模型评价器（estimator）
* `feature_column/`：特征列（feature column）组件
* `framework/`：Python API的框架型组件，包含TensorFlow编程框架中主要抽象的Python语言定义。
* `grappler/`：Grappler优化器的Python语言接口。
* `keras/`：？
* `kernel_tests/`：数据流图操作的单元测试代码，有助于用户学习各种操作的使用方法。
* `layers/`：预置的神经网络模型层（layer）组件
* `lib/`：公共基础库，涉及专用数据结构访问和文件系统I/O等。
* `ops/`：数据流图操作的Python语言接口
* `platform/`：用于访问特定操作系统或云服务接口的平台相关代码
* `profiler/`：？
* `saved_model/`：用于访问TensorFlow通用模型序列化格式（SavedModel）的组件
* `summary/`：用于生成TensorFlow事件汇总文件（summary）的组件，以便在TensorFlow中可视化计算过程。
* `tools/`：若干可独立运行的Python脚本工具，涉及访问和优化模型文件等功能。
* `training/`：与模型训练相关的组件，例如各类优化器（optimizer）、模型保存器（saver）等。
* `user_ops/`：用于存放用户自行开发的数据流图操作的Python语言接口。
* `util/`：用于存放用户自行开发的数据流图操作的Python语言接口
* `build_defs.bzl`：Bazel构建过程所需的辅助脚本，主要用于定义Python API特有的构建规则。
* `pywrap_tensorflow.py`：间接封装核心库通过C API导出的函数，以便在Python API内部调用核心库的功能。
* `tensorflow.i`：对接C API的SWIG接口描述文件，用于在软件构建时为Python API生成核心层的动态链接库。

### TensorFlow的安装目录

使用二进制包安装TensorFlow时，pip会将python文件、动态链接库和必要的依赖项复制到当前Python环境的`site-packages`或`dist-packages`目录中。
