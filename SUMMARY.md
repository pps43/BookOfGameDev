# Summary

* [为什么会有此书](README.md)


## 模式与架构

* [导言](CodePattern/README.md)

* 游戏编程模式
  * 单例
  * 抽象工厂
  * 对象池
  * 命令模式
    * 一个好用的输入系统
  * 状态模式
    * 一个好用的状态机
  * 观察者模式
    * 一个好用的消息分发系统
  * 组合vs继承
  * [MVP 架构模式](CodePattern/MVP_ArchitecturePattern.md)
  * [ECS 架构模式](CodePattern/ECS_ArchitecturePattern.md)


* [网络游戏架构](Network/README.md)
  * (TODO)前后端网络模块方案
  * (TODO)状态同步
  * [(TODO)帧同步](Network/FrameLockStepSync.md)


## 基于 Unity 引擎开发

* [导言](Unity/README.md)

* 坐标系统
  * (TODO)几个坐标系的作用和转化：屏幕坐标、相机坐标、世界坐标、本地坐标
  * (TODO)3D旋转，万向节死锁


* [UI系统](Unity/UGUI/README.md)
  * [UGUI布局系统(TODO)](Unity/UGUI/UGUILayoutSystem.md)
  * [UGUI消息系统(TDOO)](Unity/UGUI/UGUIEventSystem.md)
  * [UGUI渲染机制](Unity/UGUI/UGUIRenderSystem.md)
  * [文本优化](Unity/UGUI/UGUIOptimization_TextFont.md)
  * [图像优化：纹理压缩(TODO)](Unity/UGUI/UGUIOptimization_TextureCompression.md)
  * [UGUI代码小技巧(TODO)](Unity/UGUI/UGUITipsOnHowTo.md)


* 动画系统
  * (TODO)人型动画高级功能:IK, TargetMatch, BT等


* 碰撞系统
  * (TODO)碰撞连续性和顺序性在离散系统中如何保证
  * (TODO)碰撞矩阵最优实践

* [资源管理机制](Unity/Asset/README.md)
  * [深入Unity资源](Unity/Asset/DiveIntoUnityAsset.md)
  * [深入Assetbundle机制](Unity/Asset/DiveIntoAssetBundle.md)
  * [深入Mono的C\#资源回收](Unity/Asset/DiveIntoMonoCsharpGC.md)
  * [深入Unity序列化](Unity/Asset/DiveIntoUnitySerialization.md)


* 异步编程
  * [深入剖析Unity协程](Unity/Coroutine/DiveIntoUnityCoroutine.md)
  * [Unity协程实用技巧](Unity/Coroutine/CodeHappilyWithUnityCoroutine.md)
  * [一个好用的异步动作队列](Unity/Coroutine/CreateUsefulActionSequence.md)


* Unity 热更新
  * [Lua简介](Lua/Lang/LuaNotes.md)
  * 在Unity里使用Xlua
    * [Unity 里使用 Xlua](Lua/Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Lua/Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Lua/Xlua/XluaExampleNotes02.md)
    * [更多有关Xlua](Lua/Xlua/XluaMoreInfo.md)
  * (TODO)其他热更新方案


* [编辑器扩展](Unity/EditorExtension/README.md)
  * (TODO)可视化与反射机制
  * (TODO)用Flutter、SwiftUI的语法写Unity编辑器插件

* 分析工具
  * (TODO) 性能分析与真机调试 profiler/trepn
  * (TODO) 反编译 ILSpy/dnSpy
  * [一个好用的 overdraw 分析工具]()

* [性能优化](Unity/PerformanceOptimizition/README.md)
  * [到底优化什么](Unity/PerformanceOptimizition/WhatToOptimize.md)
  * [(TODO)移动设备的硬件架构和瓶颈]
  * [Unity Profiler正确解读方法](Unity/PerformanceOptimizition/HowToUseProfilerCorrectly.md)

* 项目规范和工作流(TODO)
  * 美术标准如何确定
  * 游戏场景规范：相机、层级、碰撞矩阵
  * 目录命名和组织
  * Unity中如何进行TDD测试驱动开发
  * Git还是Svn
  * 对第三方插件的思考





## 游戏中的算法(TODO)

## 渲染(TODO)
