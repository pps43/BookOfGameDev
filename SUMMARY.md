# Summary

* [为什么会有此书](README.md)


## 模式与架构

* [导言](Game Code/README.md)

* [游戏编程模式]
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
  * [MVP 架构模式](Game Code/Game Programming Pattern/MVP_ArchitecturePattern.md)
  * [ECS 架构模式](Game Code/Game Programming Pattern/ECS_ArchitecturePattern.md)


* [网络游戏架构(TODO)](Game Code/Network Related/README.md)
  * 前后端网络模块方案
  * 状态同步
  * [帧同步](Game Code/Network Related/FrameLockStepSync.md)


## 基于 Unity 引擎开发

* [导言](Unity Learning/README.md)

* 坐标系统(TODO)
  * 几个坐标系的作用和转化：屏幕坐标、相机坐标、世界坐标、本地坐标
  * 3D旋转，万向节死锁


* [UI系统](Unity Learning/UGUI/README.md)
  * [UGUI布局系统(TODO)](Unity Learning/UGUI/UGUILayoutSystem.md)
  * [UGUI消息系统(TDOO)](Unity Learning/UGUI/UGUIEventSystem.md)
  * [UGUI渲染机制](Unity Learning/UGUI/UGUIRenderSystem.md)
  * [文本优化](Unity Learning/UGUI/UGUIOptimization_TextFont.md)
  * [图像优化：纹理压缩(TODO)](Unity Learning/UGUI/UGUIOptimization_TextureCompression.md)
  * [UGUI代码小技巧(TODO)](Unity Learning/UGUI/UGUITipsOnHowTo.md)


* 动画系统(TODO)
  * 人型动画高级功能:IK, TargetMatch, BT等


* 碰撞系统(TODO)
  * 碰撞连续性和顺序性在离散系统中如何保证
  * 碰撞矩阵最优实践

* [资源管理](Unity Learning/Resource Management/README.md)
  * [深入Unity资源](Unity Learning/Resource Management/DiveIntoUnityAsset.md)
  * [深入Assetbundle机制](Unity Learning/Resource Management/DiveIntoAssetBundle.md)
  * [深入Mono的C\#资源回收](Unity Learning/Resource Management/DiveIntoMonoCsharpGC.md)
  * [深入Unity序列化](Unity Learning/Resource Management/DiveIntoUnitySerialization.md)


* 异步编程
  * [深入剖析Unity协程](Unity Learning/Coroutine/DiveIntoUnityCoroutine.md)
  * [Unity协程实用技巧](Unity Learning/Coroutine/CodeHappilyWithUnityCoroutine.md)
  * [一个好用的异步动作队列](Unity Learning/Coroutine/CreateUsefulActionSequence.md)


* Unity 热更新(TODO)
  * [Lua简介](Lua/Lua Language/First Notes About Lua.md)
  * 在Unity里使用Xlua
    * [Unity 里使用 Xlua](Lua/Unity with Xlua/CodeHappierWithXlua.md)
    * [Xlua Examples学习（一）](Lua/Unity with Xlua/XluaExampleNotes.md)
    * [Xlua Examples学习（二）](Lua/Unity with Xlua/XluaExampleNotes02.md)
    * [更多有关Xlua](Lua/Unity with Xlua/XluaMoreInfo.md)
  * 其他热更新方案


* [编辑器扩展(TODO)](Unity Learning/IDE/Editor Extension/README.md)
  * 可视化与反射机制
  * 用Flutter、SwiftUI的语法写Unity编辑器插件

* 分析工具(TODO)
  * 性能分析与真机调试 --包含第三方（高通）分析工具trepn
  * 反编译 ILSpy/dnSpy
  * 一个好用的 overdraw 分析工具（Computer shader）

* [性能优化](Unity Learning/PerformanceOptimizition/README.md)
  * [到底优化什么](Unity Learning/PerformanceOptimizition/WhatToOptimize.md)
  * [移动设备的硬件架构和瓶颈(TODO)]
  * [Unity Profiler正确解读方法](Unity Learning/PerformanceOptimizition/HowToUseProfilerCorrectly.md)

* 项目规范和工作流(TODO)
  * 美术标准如何确定
  * 游戏场景规范：相机、层级、碰撞矩阵
  * 目录命名和组织
  * Unity中如何进行TDD测试驱动开发
  * Git还是Svn
  * 思考：第三方插件和编辑器工具





## 游戏中的算法(TODO)

## 渲染(TODO)
