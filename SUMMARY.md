# Summary

* [写在前面的话](README.md)


## 游戏逻辑开发和优化
### 模式与架构

* 游戏编程模式
  * 单例
  * 抽象工厂
  * 对象池
  * 命令模式
    * 一个简易的输入系统
  * 状态模式
    * 一个简易的状态机
  * 观察者模式
    * 一个简易的消息分发系统
  * 组合vs继承
  * [MVP 架构模式](GameLogic/Pattern/MVP.md)
  * [ECS 架构模式](GameLogic/Pattern/ECS.md)


* [网络游戏架构](GameLogic/Network/README.md)
  * [帧同步⚠](GameLogic/Network/FrameLockStepSync.md)


### 使用Unity开发

* [导言](GameLogic/Unity/README.md)


* [UI系统](GameLogic/Unity/UGUI/README.md)
  * [UGUI布局系统⚠](GameLogic/Unity/UGUI/UGUILayoutSystem.md)
  * [UGUI消息系统⚠](GameLogic/Unity/UGUI/UGUIEventSystem.md)
  * [UGUI渲染机制](GameLogic/Unity/UGUI/UGUIRenderSystem.md)
  * [文本优化](GameLogic/Unity/UGUI/UGUIOptimization_TextFont.md)
  * [纹理压缩⚠](GameLogic/Unity/UGUI/UGUIOptimization_TextureCompression.md)
  * [UGUI代码小技巧⚠](GameLogic/Unity/UGUI/UGUITipsOnHowTo.md)


* 动画系统
  * ⚠人型动画高级功能:IK, TargetMatch, BT等


* 碰撞系统
  * ⚠碰撞连续性和顺序性在离散系统中如何保证
  * ⚠碰撞矩阵最优实践

* [资源管理机制](GameLogic/Unity/Asset/README.md)
  * [深入Unity资源](GameLogic/Unity/Asset/DiveIntoUnityAsset.md)
  * [深入Assetbundle机制](GameLogic/Unity/Asset/DiveIntoAssetBundle.md)
  * [深入Mono的C\#资源回收](GameLogic/Unity/Asset/DiveIntoMonoCsharpGC.md)
  * [深入Unity序列化](GameLogic/Unity/Asset/DiveIntoUnitySerialization.md)


* 异步编程
  * [深入剖析Unity协程](GameLogic/Unity/Coroutine/DiveIntoUnityCoroutine.md)
  * [Unity协程实用技巧](GameLogic/Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
  * [一个简易的异步动作队列](GameLogic/Unity/Coroutine/CreateUsefulActionSequence.md)


* 热更新
  * 在Unity里使用Xlua
    * [Unity 里使用 Xlua](Lua/Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Lua/Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Lua/Xlua/XluaExampleNotes02.md)
    * [更多有关Xlua](Lua/Xlua/XluaMoreInfo.md)


* [编辑器扩展](GameLogic/Unity/EditorExtension/README.md)
  * ⚠可视化与反射机制
  * ⚠用Flutter、SwiftUI的语法写Unity编辑器插件

* 分析工具
  * ⚠性能分析与真机调试 profiler/trepn
  * ⚠反编译 ILSpy/dnSpy
  * [一个简易的 overdraw 分析工具](GameLogic/Unity/PerformanceOptimizition/CreateUsefulOverdrawIndicator.md)

* [性能优化](GameLogic/Unity/PerformanceOptimizition/README.md)
  * [到底优化什么](GameLogic/Unity/PerformanceOptimizition/WhatToOptimize.md)
  * [⚠移动设备的硬件架构和瓶颈]
  * [Unity Profiler正确解读方法](GameLogic/Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)
  * [Unity 性能优化实战经验]

* [项目规范和工作流](Specs/README.md)
  * ⚠美术标准如何确定
  * ⚠游戏场景规范：相机、层级、碰撞矩阵
  * ⚠目录命名和组织
  * ⚠Unity中如何进行TDD测试驱动开发
  * ⚠Git还是Svn
  * ⚠对第三方插件的思考




## 游戏引擎开发
* [关于游戏引擎开发这件事](GameEngine/AboutGameEngine.md)

## 游戏编程语言和工具集

### .NET 和 C#

### C++

### Lua
* [Lua简介](Lua/Lang/LuaNotes.md)