# Summary

* [前言](README.md)

## AI
* Coding AI
    * [Codex 最佳实践](AI/CodingAI/CodexBestPractice.md)

## Math
* Numbers
* Functions
    * [Pairing Function 及其用途](Math/PairingFunction.md)
* Vectors & Matrices
    * [LossyScale 深入分析](Math/LossyScale.md)

## Programming Languages
* C#
    * [学习资料](Programming/CSharp/Readme.md)
    * [C# struct 灵魂拷问](Programming/CSharp/dotNetStructQuestions.md)
    * [CIL 的世界：call 和 callvirt](https://github.com/stakx/ecma-335/blob/master/docs/i.12.1.6.2.4-calling-methods.md)
    * [.NET 装箱拆箱机制](Programming/CSharp/dotNetBoxing.md)
    * [.NET 垃圾回收机制](Programming/CSharp/dotNetGC.md)
* Go
    * [Go 基础特性](Programming/Golang/Golang.md)
    * [如何正确判空 interface](Programming/Golang/GolangCheckNil.md)
    * [如何用 interface 模拟多态](Programming/Golang/GolangMockPolymorphysm.md)
    * [如何定制 JSON 序列化](Programming/Golang/GolangJson.md)
    * [如何安全在循环中删除元素](Programming/Golang/GolangRemoveItemInLoop.md)
* Lua
    * [Lua 基础特性](Programming/Lua/Lua.md)

## Game Development Fundamentals
* Game Engine
    * [学习资料](GameDev/GameEngine/GameEngineLearningMaterial.md)
    * [关于游戏引擎的认知](GameDev/GameEngine/AboutGameEngine.md)
* Networking
    * [网络同步概览](GameDev/Network/README.md)
    * [帧同步](GameDev/Network/FrameLockStepSync.md)
    * [网络物理同步](GameDev/Network/NetworkedPhysics/IntroOfNetworkedPhysics.md)
* Physics
    * [PhysX 基本概念](GameDev/Physics/PhysXBasics.md)
    * [PhysX 场景查询](GameDev/Physics/PhysXSceneQuery.md)
* Animation
    * [角色动画的核心：从 IK 重定向到骨骼的本质](GameDev/Animation/IKRetargetingAndBoneSystem.md)
* Design Patterns
    * [常用设计模式](GameDev/DesignPattern/CommonPatternsCollection.md)
    * [MVP 架构模式](GameDev/DesignPattern/MVP.md)
    * [ECS 架构模式](GameDev/DesignPattern/ECS.md)

## Using Unity Engine
* Runtime
    * [Unity 拥抱 CoreCLR](GameDev/Unity/Runtime/MonoOrCLR.md)
    * [浅析 Mono 内存管理](GameDev/Unity/Runtime/DiveIntoMonoMemory.md)
* UGUI
    * [浅析 UGUI 渲染机制](GameDev/Unity/UGUI/UGUIRenderSystem.md)
    * [浅析 UGUI 文本优化](GameDev/Unity/UGUI/UGUIOptimization_TextFont.md)
    * [UGUI 实用技巧](GameDev/Unity/UGUI/UGUITipsOnHowTo.md)
* Resource Management
    * [Unity 堆内存分类与管理](GameDev/Unity/Asset/README.md)
    * [深入 Unity 资源](GameDev/Unity/Asset/DiveIntoUnityAsset.md)
    * [深入 Unity 序列化](GameDev/Unity/Asset/DiveIntoUnitySerialization.md)
    * [深入 AssetBundle 机制](GameDev/Unity/Asset/DiveIntoAssetBundle.md)
* Asynchronous Programming
    * [深入 Unity 协程](GameDev/Unity/Coroutine/DiveIntoUnityCoroutine.md)
    * [Unity 协程实用技巧](GameDev/Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
    * [异步动作队列](GameDev/Unity/Coroutine/CreateUsefulActionSequence.md)
* Scripting & Hot Reload (xLua)
    * [Unity + xLua](Programming/Lua/Xlua/CodeHappierWithXlua.md)
    * [xLua 示例学习（一）](Programming/Lua/Xlua/XluaExampleNotes.md)
    * [xLua 示例学习（二）](Programming/Lua/Xlua/XluaExampleNotes02.md)
* Editor Extension
    * [编辑器扩展概览](GameDev/Unity/EditorExtension/README.md)
* Performance Optimization
    * [性能优化概览](GameDev/Unity/PerformanceOptimizition/README.md)
    * [浅析 Unity Profiler](GameDev/Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)
    * [Overdraw 分析工具介绍](GameDev/Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)

## Roadmap
* Math
    * Float IEEE754 对确定性的影响（TBD）
    * TRS 基础概念（TBD）
    * FromToRotation 实现细节（TBD）
    * Slerp 球形插值（TBD）
    * Bezier Curve 为什么重要（TBD）
    * Interpolation 和 Extrapolation 实现细节（TBD）
* Go
    * 如何安全关闭 channel（TBD）
    * 如何集成 C++ 库（cgo + swig）（TBD）
    * 如何做性能测试（benchmark, pprof）（TBD）
* Networking
    * 状态同步（TBD）
* Physics
    * PhysX 增加 Scale 支持（TBD）
    * PhysX 碰撞检测（TBD）
    * PhysX 刚体动力学（TBD）
    * PhysX 角色控制器（TBD）
    * PhysX 接入项目工程（TBD）
    * 物理同步（TBD）
    * 物理破坏（TBD）
* Platform
    * WebGL（TBD）
* Case Studies
    * Source Engine（TBD）
    * DOOM3 BFG（TBD）
