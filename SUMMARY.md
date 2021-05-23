# Summary

* [自序](README.md)

## 游戏逻辑开发和优化
* 模式与架构
  * [居家旅行必备的六大设计模式](GameLogic/Pattern/CommonPatternsCollection.md)
  * 🟡组合还是继承，这是个问题
  * [MVP 架构模式](GameLogic/Pattern/MVP.md)
  * [ECS 架构模式](GameLogic/Pattern/ECS.md)
  * [网络游戏架构](GameLogic/Network/README.md)
    * [🔴帧同步](GameLogic/Network/FrameLockStepSync.md)

* 基于Unity开发游戏
  * [导言](GameLogic/Unity/README.md)
  * UGUI
    * [UGUI渲染机制](GameLogic/Unity/UGUI/UGUIRenderSystem.md)
    * [UGUI文本优化](GameLogic/Unity/UGUI/UGUIOptimization_TextFont.md)
    * [UGUI代码小技巧](GameLogic/Unity/UGUI/UGUITipsOnHowTo.md)
    * [🟡UI纹理压缩](GameLogic/Unity/UGUI/UGUIOptimization_TextureCompression.md)
  * 资源管理
    * [Unity堆内存的分类和管理方式](GameLogic/Unity/Asset/README.md)
    * [深入Unity资源](GameLogic/Unity/Asset/DiveIntoUnityAsset.md)
    * [深入Assetbundle机制](GameLogic/Unity/Asset/DiveIntoAssetBundle.md)
    * [深入Mono的C\#资源回收](GameLogic/Unity/Asset/DiveIntoMonoCsharpGC.md)
    * [深入Unity序列化](GameLogic/Unity/Asset/DiveIntoUnitySerialization.md)
  * 异步编程
    * [深入剖析Unity协程](GameLogic/Unity/Coroutine/DiveIntoUnityCoroutine.md)
    * [Unity协程实用技巧](GameLogic/Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
    * [一个简易的异步动作队列](GameLogic/Unity/Coroutine/CreateUsefulActionSequence.md)
  * 热更新
    * [Unity 里使用 Xlua](Lua/Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Lua/Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Lua/Xlua/XluaExampleNotes02.md)
    * [更多有关Xlua](Lua/Xlua/XluaMoreInfo.md)
  * [编辑器扩展](GameLogic/Unity/EditorExtension/README.md)
    * 🔴可视化与反射机制
    * 🔴用Flutter、SwiftUI的语法写Unity编辑器插件
  * [性能优化](GameLogic/Unity/PerformanceOptimizition/README.md)
    * [到底优化什么](GameLogic/Unity/PerformanceOptimizition/WhatToOptimize.md)
    * [🟡移动设备的硬件架构和瓶颈]
    * [Unity Profiler正确解读方法](GameLogic/Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)
    * [Unity 性能优化实战经验]
    * [一个简易的 overdraw 分析工具](GameLogic/Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)
  * 项目规范和工作流
    * 🔴美术标准如何确定
    * 🔴游戏场景规范：相机、层级、碰撞矩阵
    * 🔴目录命名和组织
    * 🔴Unity中如何进行TDD测试驱动开发
    * 🔴Git还是Svn
  * 其他  
    * 🔴人型动画高级功能
    * 🔴碰撞连续性和顺序性
    * 🔴碰撞矩阵


## 游戏引擎开发
* [关于游戏引擎](GameEngine/AboutGameEngine.md)
* [学习资料](GameEngine/GameEngineLearningMaterial.md)
* 基于Unreal学习引擎
  * 学引擎，到底学什么
  * UE4 的构建系统 UBT
  * UE4 的基础工具 UHT
  * UE4 的 UObject 漫谈，意义和代价
  * UE4 的类型系统
  * UE4 的反射机制
  * UE4 的垃圾回收机制
  * UE4 的蓝图虚拟机
  * C++和蓝图的互操作，以及带来的复杂性

## 语言 | IDE | Devops

* C# 及 dotNET
  * [🟢dotNET 垃圾回收机制](DotNet/dotNetGC.md)
* C++
* Lua
  * [Lua简介](Lua/Lang/LuaNotes.md)
* Tools
  * Visual Studio高级调试技巧
  * Renderdoc
* Devops
  * 游戏开发与单元测试
