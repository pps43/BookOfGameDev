# 资源管理

## 引言
在开始正文前，要明确Unity游戏的内存可以分成哪几块，以及本节讨论的是哪一块。

游戏占用的内存大体分为`代码 + 资源`。

对于Unity游戏，确切来说，可以分为`Native堆`（Unity资源）和`Mono堆`（c#资源）。后者通过对应版本的Mono自带的垃圾采集器（GC，Garbage Collector）进行内存管理，前者提供了一些API以及很弱的自动内存管理。

本章首先讨论Native堆上的资源管理，为此，需要先认识Unity的资源。
接着，会着重讨论Unity中的AssetBundle、Resources的使用，以及Unity的一些特殊的资源文件夹。
然后讨论Mono堆上的资源管理，主要是为了减少GC工作而做的一些优化。



## 资源分析小工具
- unity内存分类剖析
https://bitbucket.org/Unity-Technologies/memoryprofiler/overview

- 查看IL代码工具：ILSpy
http://www.fishlee.net/soft/ilspy_chs/