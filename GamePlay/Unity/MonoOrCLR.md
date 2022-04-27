# Unity的未来，固守Mono，还是拥抱CoreCLR？

更新日期 `2022-04-24`
初版日期 `2021-07-04`

> TLDR; Unity ❤ CoreCLR

### Mono vs CoreCLR

对于一个C#的初学者，首先要了解的便是.NET和C#的关系。所以这里不再赘述。对于一个Unity的初学者，在使用C#编码的过程中，一定会遇到一些C#新特性不能在项目中使用的情况，这是因为微软官方提供的.NET运行时环境（最新版为 .NET 6 的 `CoreCLR`）远比Unity集成的`Mono`强大。由于历史原因，Unity一直未能使用最新的.NET运行时。本文就来细说一下其中的历史，以及Unity未来的发展。

首先，`Mono`是.NET的开源实现，由Xamarin牵头维护 [mono/mono](https://github.com/mono/mono) 这个repo。2016年Xamarin被微软收购，将其license从GPL改成了MIT，同时微软也参与到Mono的开发中。在微软的 [dotnet/runtime](https://github.com/dotnet/runtime/tree/main/src/mono) 这个repo中，可以发现mono，但这并不是mono/mono的替代品，而是微软为了方便其他组件开发，将部分代码拷贝过来进行魔改，在需要的时候同步回mono/mono。(出处： [Announcement: Consolidating .NET GitHub repos](https://github.com/dotnet/runtime/issues/13257))

然后说一下 `CoreCLR`。微软改名部绝非浪得虚名，这些年的一系列改名操作把好端端的.NET技术搅成一滩浑水。非常简要的说，过去.NET运行时只能用于Windows平台，名为 .NET Framework，运行时叫CLR，大名鼎鼎的《CLR via C#》就是基于该运行时。后来微软决定将.NET技术开源，并彻底地跨平台（Windows，Linux，MacOS等），顺便大幅提高运行效率并抛弃一些旧组件，重写了一版 .NET Core，运行时叫CoreCLR。双线开发并不是个好主意，因此再后来微软决定将前者废弃，以.NET Core相关技术为核心，将.NET技术大一统（桌面开发、移动开发、游戏开发、IoT开发、云开发等等），史称 [`.NET 5`](https://docs.microsoft.com/en-us/dotnet/core/dotnet-five)。因此，往后.NET的官方运行时就叫`CoreCLR`（暂定🙂）。repo在这里：[dotnet/runtime](https://github.com/dotnet/runtime)。

大饼是画出来了，但当下移动平台的.NET开发的主流还是`Mono`。然而Mono对.NET的特性支持和性能一直落后于微软官方的`CLR`，性能上也大幅落后于`CoreCLR`（补充：2018年Unity的[官方数据](https://youtu.be/QkM6zEGFhDY?t=203)是CLR比Mono快30%~3倍。Mono这些年的进步有目共睹，但奈何不了CoreCLR的神通。经过4年的发展，多处民间测试资料显示CoreCLR比Mono快10~20倍，比IL2CPP快8~10倍）。这里结合[Wiki](https://en.wikipedia.org/wiki/Mono_(software))，对Mono的重要历史加以整理。

- 2010年9月，Mono2.8发布，带来了新的分代式GC：SGen；支持 C#4；支持 .NET 4.0。
- 2013年7月，Mono3.2发布，SGen取代Boehm成为默认的GC。
- 2015年4月，Mono4.0发布，开始集成 .NET Core；支持 C#6。
- 2017年5月，Mono5.0发布，开启了concurrent SGen；使用Roslyn编译器；支持 C#7。
- 2019年9月，Mono6.4发布，支持.NET Standard 2.1；支持 .NET 4.8。

### 过去，Unity 选择 Mono

上文理清了`Mono` 和`CoreCLR`的关系，下面说说Unity在这二者间的选择。

早在2008年，Unity就宣布和Mono合作，但后续Mono新版本使用SGen GC取代Boehm GC时，Unity不想再次付许可证费用。直到当下（2021年7月），Unity依然依赖 Boehm GC。这是一种没有分代的、Mark-Sweep的、保守的GC，而且并不能精确地识别垃圾。

> Unity has been and is still relying on the [Boehm GC](https://en.wikipedia.org/wiki/Boehm_garbage_collector), which is a `conservative (stack-root) GC`. The link above doesn't go into some details like how managed objects on the stack are collected by the GC, but basically: a conservative GC will scan the entire stack of all managed threads to "pin" memory referenced by it. Because of this blind scan, it can bring false pin, because it can interpret an integer value, as a pointer to a region of the heap memory, while it was really an int in the first place. By doing so, a conservative GC can start to block some objects from being collected (or worse for a moving-generational GC, to relocate objects). Otoh, a precise GC is able to scan precisely stack-roots and report only pointers that actually point to heap memory. In order for a precise GC to work, the (JIT) codegen needs to be GC aware, which is the case for CoreCLR.

对于Boehm GC造成的性能问题，Unity有一些折中方案。

- 先尽量分配好所有对象的内存，然后关闭GC，等到合适的时机（如关卡结束），再开启GC；
- 默认开启 incremental 模式分帧处理，注意如果在期间有大量引用关系的改写，分帧处理反而会有大量额外性能损耗（主要来自写屏障）

### 未来，Unity 会选择 CoreCLR 吗

自2016年微软将Mono的许可证由GPL改为MIT以来，Unity也加入了 .NET Foundation，开始将最新的Mono集成到自己的引擎中。但随着微软构筑开源的大一统解决方案 .NET 5，Unity似乎改变了原先的想法。从官方论坛中可以总结出他们的规划：

- 首先集成最新的Mono，因为其支持 .NET Core 的BCL；
- 然后将自家的[IL2CPP](https://docs.unity3d.com/2021.2/Documentation/Manual/IL2CPP.html)也更新（其依赖Mono编译出IL）；
- [Unity 2021.2开始完全支持.NET Standard 2.1，C#8和部分C#9](https://forum.unity.com/threads/unity-future-net-development-status.1092205/page-10#post-7658806)（**Span**，Range，default interface methods），其中`Span`的影响非常深远，目前和BurstCompiler的NativeArray还不能无缝转换，[最大难点](https://forum.unity.com/threads/unity-future-net-development-status.1092205/page-10#post-7665529)是Span并没有自己的memory但后者有。
- 支持 C#9/10，基于前面的工作，这一步并不难；
- 支持 .NET 6（跳过 .NET 5）。但有两大难题：
  - 一是所有dll必须重新编译（预计Unity2022.1完成）；
  - 二是要修改UnityEditor中大量使用AppDomain进行hot reload的部分（AppDomain在新版.NET中被废弃，目前的替代方案是Assembly Load Context，但隔离性并不像AppDomain那么强。Unity正在和微软协商中间方案。）

    > The issue with Assembly Load Context is that is doesn't provide all the functionality of appdomain reloads. In general Assembly Load Context is cooperative, and any remaining references (static fields, GC Handles, running threads, etc) will prevent the code from being unloaded. We are investigating how to augment the current behavior.

- **可能**用CoreCLR替代Mono（GC也相应升级为CoreCLR GC），在此之前，GC并不会升级为Mono的SGen。这项工作会持续比较久，目前还没有ETA。Unity大部分代码是C++，C#只有薄薄的一层（但是越来越多的代码在切换到 C#）。在切换到CoreCLR后，其访问Managed Object的方式需要彻底改变，因此改动会很大。参考[Write a custom .NET Core runtime host - .NET | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/core/tutorials/netcore-hosting)。总体顺序是：先将Player替换为CoreCLR，然后将Editor也替换掉。
- **不太可能**用微软最新发布的`NativeAOT`替代`IL2CPP`，但会和微软合作、学习和交流。[出处](https://forum.unity.com/threads/unity-future-net-development-status.1092205/page-9#post-7632391)

长远来看，Unity该团队已经意识到自己当年的一些轮子（Coroutine, Customized Boehm GC, IL2CPP, asmdef等）在近几年来.NET运行时、工具链和整套生态的飞速发展面前显得有些陈旧，正在致力于向开发者提供原汁原味的.NET开发体验（[出处](https://forum.unity.com/threads/unity-future-net-development-status.1092205/page-13#post-7923634)），同时尽量不颠覆原有的使用习惯（例如出于[这些原因](https://forum.unity.com/threads/unity-future-net-development-status.1092205/page-16#post-8022236)，`UPM`不会和`NuGet`双向互通）。

总之，希望Unity越来越好。GDC2022上Unity团队将以上规划也已经传达出来，[视频链接](https://youtu.be/6UiKtnDQeEg)。最新消息可关注官方论坛的讨论： [Unity Future .NET Development Status - Unity Forum](https://forum.unity.com/threads/unity-future-net-development-status.1092205/)
