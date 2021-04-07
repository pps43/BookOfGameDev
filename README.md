# 缘之所起



---

游戏开发是一个能串联起很多有趣经历的一个领域。是艺术、科学、工程的完美结合。这本小书主要记载一些游戏开发之路上的知识、经验和感悟，希望对更多的人有所帮助。



丙申年春月

---
> 下面是推荐阅读的几个章节：
> - [深入Unity序列化](Unity/Asset/DiveIntoUnitySerialization.md)
> - [深入Unity资源](Unity/Asset/DiveIntoUnityAsset.md)
> - [深入剖析Unity协程](Unity/Coroutine/DiveIntoUnityCoroutine.md)
> - [深入Mono的C\#资源回收](Unity/Asset/DiveIntoMonoCsharpGC.md)
> - [UGUI渲染机制](Unity/UGUI/UGUIRenderSystem.md)
> - [一个好用的 overdraw 分析工具](Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)
> - [一个好用的异步动作队列](Unity/Coroutine/CreateUsefulActionSequence.md)

---
# 从事游戏引擎开发前最好知道的事

兜兜转转，还是下决心回到游戏行业。不同的是，这一次的切入点是游戏引擎。在这里整理了一些近期的材料，和对游戏引擎未来的设想。

## 背景

2020年，全球游戏市场规模约`2000亿美元`，用户规模约`27亿`。中国游戏市场销售收入约`500亿美元`，用户规模约`6亿`。

> 作为对比，同时期全球云计算市场规模约`2200亿美元`，中国约`200亿美元`。

游戏市场规模扩大的同时，产业链也日趋成熟：

- 研运（研发、发行、运营）
- 分发渠道（三方商店如应用宝、硬件厂商如huawei、运营商、垂直渠道如taptap）
- 用户服务（媒体、语音、加速器、交易）
- 产业服务（游戏引擎、云服务、支付、信息流广告、数据分析、ASO、IP供应）

对于游戏硬件平台的收入分布，移动设备约`50%`，主机约`30%`，PC约`20%`。

此外，国内游戏产业值得注意的趋势是：

- 游戏出海。头部厂商例如`FunPlus`，`米哈游`，`莉莉丝`，`腾讯`，`网易`。
- 内容为王。渠道和流量红利减弱，开始拼研发水平，一些具有产品实力的中小团队在steam上崭露头角。
- 上海崛起。2020年，`33%`的游戏求职者迁入上海（深圳为11%，成都为6%）。

越是大型的游戏，越离不开游戏引擎。其大致分为商业引擎（如`Unity`，`Unreal`）、免费引擎（如`Godot`，`Ogre3D`）和自研引擎。游戏引擎的不可或缺指出在于其提供了内容制作（强大的DCC工具链）、跨平台能力。同时由于商业引擎大都可提供实时渲染、物理模拟等功能，也开始应用在游戏以外的领域。



## 商业游戏引擎近况

### Unity

近年来，Unity由`Entity-Component-System (ECS)`架构切入，主推CPU友好的`DOTS`解决方案。其`Scriptable Render Pipeline (SRP)`以先进的设计思想，理论上可以定制出比Unreal更高效的渲染效率，受到开发者高度评价（[链接](https://www.zhihu.com/question/379857981/answer/1179288746)），甚至技术美术都可以自己改渲染管线。


Unity公司除了引擎本身，还提供云托管、云测试、广告系统。Unity的2020年总收入约7.7亿美元。这是因为引擎本身占游戏收入的分成很有限，并且头部厂商具有自研引擎的能力。所以Unity正大力发展游戏以外的业务。截至2020年底的793个年缴费超过10万美金的客户中，有60个来自游戏以外的行业。例如船厂车间和船坞设施和空间布局规划。

### Unreal
Unreal的底层架构还是`OO`而没有`ECS`方案，其主攻方向是以`GPU Driven`，以优化pipeline层面的性能。工程难度很大，例如一个复杂的粒子系统可能就没法完全 GPU Driven 而需要重新设计。

Unreal 5 的新特性：`Nanite` (虚拟几何体，运行时自动LOD)，`Lumen`（动态全局光照）。此前游戏3D物体的制作流程是：基本形--(雕刻)-->高模--(拓扑)-->低模-->游戏引擎；
高模--(烘焙)--> 法线贴图等贴图 --> 游戏引擎。游戏引擎运行时根据算法进行LOD（普通LOD会导致远处巨大物体细节丧失，快速移动时远处物体突然出现）。现在有了 `Nanite`，高模可以直接导入游戏引擎且不需要LOD算法。而且高模可以直接来自3D扫描器（例如Quixel Bridge）。大大节省了美术制作流程，提高了细节表现。关于光照，此前全局光效果多采用烘焙，只适用于场景中静态的物体（也不能支持昼夜交替），因此要和局部的动态光相互配合；另一种解决方案光线追踪需要专门的硬件支持。`Lumen`则不依赖光追硬件，来实现近似光追达到的全局光效果。

### 基础技术
引擎技术的革新往往源于基础技术的革新，例如 `GPU Direct Storage` ([Nvdia RTX IO](https://www.nvidia.com/en-us/geforce/news/rtx-io-gpu-accelerated-storage-technology/)，读取带宽从顶级SSD的7GB/s，提升到14GB/s，同时CPU占用变为原来的1/4)。

## 游戏引擎子方向选择
Rendering, Animation, Simulation, Tooling

[TODO]


## 游戏引擎开发 vs 技术美术

通用基础
- C++, 尤其内存管理，对象的生命周期管理
- 多线程，缓存
- GC，
- 渲染API，OpenGL/DX/Vulkan

Unity专用
- VM, IL2CPP, GC优化, Burst Compiler, ... DOTS
- 资源管理：Addressable, Scriptable Building Pipeline
- 渲染：SRP，ShaderLab

[TODO]

## 游戏之外，以及未来


就中国游戏行业自身来说，伴随着经济的迅速发展和工业化完成，按照美国和日本的历史规律，接下来文化产业也会迎来爆发式的发展。中国游戏行业也会在未来十年内迅速提升在全球的影响力。

就大局来说，未来一定是属于虚拟世界的，而目前来看，唯有游戏引擎有潜力成为创建这些虚拟世界的工具。
从这个角度上来说，从事游戏引擎开发便是参与人类最浪漫的梦想之一——创世。