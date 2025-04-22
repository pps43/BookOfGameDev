# Summary

* [你好](README.md)

## Math
* Number
    * Float IEEE754对确定性的影响
    * Pairing Function及其用途
* Vector and Matrix
    * TRS相关推导
* Quatenion
    * FromToRotation实现细节
* Lerp and Curve
    * Slerp球形插值
    * Bezier Curve为什么重要
    * Interpolation和Extrapolation实现细节

## Programming
* C#
  * [学习资料](DotNet/Readme.md)
  * [C# struct灵魂拷问](DotNet/dotNetStructQuestions.md)
  * [CIL的世界：call和callvirt](https://github.com/stakx/ecma-335/blob/master/docs/i.12.1.6.2.4-calling-methods.md)
  * [.NET装箱拆箱机制](DotNet/dotNetBoxing.md)
  * [深入.NET垃圾回收(GC)机制](DotNet/dotNetGC.md)

* Go
    * interface与正确的判空
    * interface与模拟多态
    * 如何集成c++库

* [Lua](Lua/LuaNotes.md)

## Platform
* WebGL

## General Game Development

* Game Engine
    * [学习资料](GameEngine/GameEngineLearningMaterial.md)
    * [关于游戏引擎的认知](GameEngine/AboutGameEngine.md)

* [Networking](GamePlay/Network/README.md)
    * [帧同步](GamePlay/Network/FrameLockStepSync.md)
    * 状态同步
    * [物理同步](GamePlay/Network/NetworkedPhysics/IntroOfNetworkedPhysics.md)

* Physics
    * PhysX基本概念
    * PhysX增加Scale支持
    * [PhysX场景查询](GameEngine/Physics/PhysXSceneQuery.md)
    * PhysX碰撞检测
    * PhysX刚体动力学
    * PhysX角色控制器
    * PhysX接入项目工程
    * 物理同步
    * 物理破坏

* Design Pattern
    * [常用设计模式](GamePlay/Pattern/CommonPatternsCollection.md)
    * [MVP 架构模式](GamePlay/Pattern/MVP.md)
    * [ECS 架构模式](GamePlay/Pattern/ECS.md)

## Unity
* Runtime
    * [Unity拥抱CoreCLR](GamePlay/Unity/Runtime/MonoOrCLR.md)
    * [浅析Mono内存管理](GamePlay/Unity/Runtime/DiveIntoMonoMemory.md)

* UGUI
    * [浅析UGUI渲染机制](GamePlay/Unity/UGUI/UGUIRenderSystem.md)
    * [浅析UGUI文本优化](GamePlay/Unity/UGUI/UGUIOptimization_TextFont.md)
    * [介绍若干UGUI实用技巧](GamePlay/Unity/UGUI/UGUITipsOnHowTo.md)

* Resource Management
    * [浅析Unity堆内存的分类和管理方式](GamePlay/Unity/Asset/README.md)
    * [深入Unity资源](GamePlay/Unity/Asset/DiveIntoUnityAsset.md)
    * [深入Unity序列化](GamePlay/Unity/Asset/DiveIntoUnitySerialization.md)
    * [深入Assetbundle机制](GamePlay/Unity/Asset/DiveIntoAssetBundle.md)

* Async
    * [深入Unity协程](GamePlay/Unity/Coroutine/DiveIntoUnityCoroutine.md)
    * [介绍若干Unity协程实用技巧](GamePlay/Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
    * [异步动作队列](GamePlay/Unity/Coroutine/CreateUsefulActionSequence.md)

* Hot Reload
    * [Unity+Xlua](Lua/Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Lua/Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Lua/Xlua/XluaExampleNotes02.md)

* [Editor Extension](GamePlay/Unity/EditorExtension/README.md)

* [Performance](GamePlay/Unity/PerformanceOptimizition/README.md)
    * [浅析Unity Profiler](GamePlay/Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)
    * [介绍一个Overdraw分析工具](GamePlay/Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)

## Real-world Project

* Souce Engine
* DOOM3 BFG