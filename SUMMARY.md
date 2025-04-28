# Summary

* [你好](README.md)

## Math
* Number
    * Float IEEE754对确定性的影响
    * [Pairing Function及其用途](Math/PairingFunction.md)
* Vector and Matrix
    * TRS基础概念
    * [LossyScale深入分析](Math/LossyScale.md)
* Quatenion
    * FromToRotation实现细节
* Lerp and Curve
    * Slerp球形插值
    * Bezier Curve为什么重要
    * Interpolation和Extrapolation实现细节

## Programming
* C#
  * [学习资料](Programming/CSharp/Readme.md)
  * [C# struct灵魂拷问](Programming/CSharp/dotNetStructQuestions.md)
  * [CIL的世界：call和callvirt](https://github.com/stakx/ecma-335/blob/master/docs/i.12.1.6.2.4-calling-methods.md)
  * [.NET装箱拆箱机制](Programming/CSharp/dotNetBoxing.md)
  * [.NET垃圾回收机制](Programming/CSharp/dotNetGC.md)

* Go
    * [基础特性](Programming/Golang/Golang.md)
    * 如何正确的判空interface
    * 如何用interface模拟多态
    * 如何定制json序列化
    * 如何安全在循环中删除元素
    * 如何安全关闭channel
    * 如何集成c++库(cgo+swig)
    * 如何性能测试(benchmark, pprof)

* Lua
    * [基础特性](Programming/Lua/Lua.md)


## General Game Development

* Game Engine
    * [学习资料](GameDev/GameEngine/GameEngineLearningMaterial.md)
    * [关于游戏引擎的认知](GameDev/GameEngine/AboutGameEngine.md)

* [Networking](GameDev/Network/README.md)
    * [帧同步](GameDev/Network/FrameLockStepSync.md)
    * 状态同步
    * [物理同步](GameDev/Network/NetworkedPhysics/IntroOfNetworkedPhysics.md)

* Physics
    * [PhysX基本概念](GameDev/Physics/PhysXBasics.md)
    * PhysX增加Scale支持
    * [PhysX场景查询](GameDev/Physics/PhysXSceneQuery.md)
    * PhysX碰撞检测
    * PhysX刚体动力学
    * PhysX角色控制器
    * PhysX接入项目工程
    * 物理同步
    * 物理破坏

* Design Pattern
    * [常用设计模式](GameDev/DesignPattern/CommonPatternsCollection.md)
    * [MVP 架构模式](GameDev/DesignPattern/MVP.md)
    * [ECS 架构模式](GameDev/DesignPattern/ECS.md)

## Unity
* Runtime
    * [Unity拥抱CoreCLR](GameDev/Unity/Runtime/MonoOrCLR.md)
    * [浅析Mono内存管理](GameDev/Unity/Runtime/DiveIntoMonoMemory.md)

* UGUI
    * [浅析UGUI渲染机制](GameDev/Unity/UGUI/UGUIRenderSystem.md)
    * [浅析UGUI文本优化](GameDev/Unity/UGUI/UGUIOptimization_TextFont.md)
    * [介绍若干UGUI实用技巧](GameDev/Unity/UGUI/UGUITipsOnHowTo.md)

* Resource Management
    * [浅析Unity堆内存的分类和管理方式](GameDev/Unity/Asset/README.md)
    * [深入Unity资源](GameDev/Unity/Asset/DiveIntoUnityAsset.md)
    * [深入Unity序列化](GameDev/Unity/Asset/DiveIntoUnitySerialization.md)
    * [深入Assetbundle机制](GameDev/Unity/Asset/DiveIntoAssetBundle.md)

* Async
    * [深入Unity协程](GameDev/Unity/Coroutine/DiveIntoUnityCoroutine.md)
    * [介绍若干Unity协程实用技巧](GameDev/Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
    * [异步动作队列](GameDev/Unity/Coroutine/CreateUsefulActionSequence.md)

* Hot Reload
    * [Unity+Xlua](Programming/Lua/Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Programming/Lua/Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Programming/Lua/Xlua/XluaExampleNotes02.md)

* [Editor Extension](GameDev/Unity/EditorExtension/README.md)

* [Performance](GameDev/Unity/PerformanceOptimizition/README.md)
    * [浅析Unity Profiler](GameDev/Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)
    * [介绍一个Overdraw分析工具](GameDev/Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)

## Platform
* WebGL

## Real-world Project

* Souce Engine
* DOOM3 BFG