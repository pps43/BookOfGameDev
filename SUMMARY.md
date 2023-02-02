# Summary

* [你好](README.md)

## 游戏逻辑开发
* 模式与架构
  * [必备设计模式](GamePlay/Pattern/CommonPatternsCollection.md)
  * [MVP 架构模式](GamePlay/Pattern/MVP.md)
  * [ECS 架构模式](GamePlay/Pattern/ECS.md)
  * [网络游戏架构](GamePlay/Network/README.md)
    * [🔴帧同步](GamePlay/Network/FrameLockStepSync.md)

* 基于Unity开发游戏
  * Runtime
    * [Unity的未来，是固守Mono, 还是拥抱CoreCLR](GamePlay/Unity/MonoOrCLR.md)
  * UGUI
    * [UGUI渲染机制](GamePlay/Unity/UGUI/UGUIRenderSystem.md)
    * [UGUI文本优化](GamePlay/Unity/UGUI/UGUIOptimization_TextFont.md)
    * [UGUI代码小技巧](GamePlay/Unity/UGUI/UGUITipsOnHowTo.md)
  * 资源管理
    * [Unity堆内存的分类和管理方式](GamePlay/Unity/Asset/README.md)
    * [深入Unity资源](GamePlay/Unity/Asset/DiveIntoUnityAsset.md)
    * [深入Assetbundle机制](GamePlay/Unity/Asset/DiveIntoAssetBundle.md)
    * [深入Mono的C\#资源回收](GamePlay/Unity/Asset/DiveIntoMonoCsharpGC.md)
    * [深入Unity序列化](GamePlay/Unity/Asset/DiveIntoUnitySerialization.md)
  * 异步
    * [深入剖析Unity协程](GamePlay/Unity/Coroutine/DiveIntoUnityCoroutine.md)
    * [Unity协程实用技巧](GamePlay/Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
    * [一个简易的异步动作队列](GamePlay/Unity/Coroutine/CreateUsefulActionSequence.md)
  * 热更新
    * [Unity 里使用 Xlua](Lua/Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Lua/Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Lua/Xlua/XluaExampleNotes02.md)
  * [编辑器扩展](GamePlay/Unity/EditorExtension/README.md)
  * [性能优化](GamePlay/Unity/PerformanceOptimizition/README.md)
    * [到底优化什么](GamePlay/Unity/PerformanceOptimizition/WhatToOptimize.md)
    * [Unity Profiler正确解读方法](GamePlay/Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)
    * [一个简易的 overdraw 分析工具](GamePlay/Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)


## 游戏引擎开发
* [关于游戏引擎](GameEngine/AboutGameEngine.md)
* [学习资料](GameEngine/GameEngineLearningMaterial.md)

## 编程

* C# 和 .NET
  * [学习资料](DotNet/Readme.md)
  * [struct灵魂拷问](DotNet/dotNetStructQuestions.md)
  * 🔴[CIL的世界：call和callvirt]
  * [.NET装箱拆箱(box/unbox)机制](DotNet/dotNetBoxing.md)
  * [.NET垃圾回收(GC)机制](DotNet/dotNetGC.md)
* [Lua](Lua/LuaNotes.md)