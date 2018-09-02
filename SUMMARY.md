# Summary

* [为什么会有此书](README.md)


## 模式与架构

* [导言](Game Code/README.md)

* [游戏编程模式](Game Code/Game Programming Pattern/README.md)
  * \[单例\]
  * \[组合还是继承\]
  * 命令模式
    * 如何实现解耦的输入系统
  * \[状态模式\]
    * 自己实现的一个较完善的状态机
  * \[消息机制\]
    * 自己实现的一个较完善的消息系统
  * [MVP 架构模式（适合UI）](Game Code/Game Programming Pattern/MVP_ArchitecturePattern.md)
  * [ECS 架构模式（不适合UI）](Game Code/Game Programming Pattern/ECS_ArchitecturePattern.md)


* [网络游戏架构](Game Code/Network Related/README.md)
  * [帧同步](Game Code/Network Related/FrameLockStepSync.md)



## Unity

* [导言](Unity Learning/README.md)
* [编辑器扩展](Unity Learning/IDE/Editor Extension/README.md)
  * \[可视化与反射机制\]  --知乎上和km上有文章


* \[分析工具\]
  * \[性能分析与真机调试\] --包含第三方（高通）分析工具trepn
  * 反编译 ILSpy（yield有用到）


* [协程]
  * [深入剖析Unity协程：从Yield到StartCoroutine](Unity Learning/Coroutine/DiveIntoUnityCoroutine.md)
  * [Unity协程实用技巧](Unity Learning/Coroutine/CodeHappilyWithUnityCoroutine.md)



* [碰撞和物理]
  * todo：onenote待整理



* [资源管理](Unity Learning/Resource Management/README.md)
  * [浅析Unity资源](Unity Learning/Resource Management/DiveIntoUnityAsset.md)
  * [浅析Assetbundle机制](Unity Learning/Resource Management/DiveIntoAssetBundle.md)
  * [浅析基于Mono的C\#资源回收](Unity Learning/Resource Management/DiveIntoMonoCsharpGC.md)



* [UGUI](Unity Learning/UGUI/README.md)
  * [布局系统](Unity Learning/UGUI/UGUILayoutSystem.md)
  * [消息系统](Unity Learning/UGUI/UGUIEventSystem.md)
  * [浅析UGUI的渲染机制](Unity Learning/UGUI/UGUIRenderSystem.md)
  * [浅析字体和文本的优化](Unity Learning/UGUI/UGUIOptimization_TextFont.md)
  * [UGUI优化高级篇：纹理压缩](Unity Learning/UGUI/UGUIOptimization_TextureCompression.md)
  * [实用技巧](Unity Learning/UGUI/UGUITipsOnHowTo.md)



* [性能优化](Unity Learning/PerformanceOptimizition/README.md)
  * [到底是优化什么](Unity Learning/PerformanceOptimizition/WhatToOptimize.md)
  * [Profiler的正确解读方法](Unity Learning/PerformanceOptimizition/HowToUseProfilerCorrectly.md)



* [前沿功能]
  * [TileMap和Brush]
  * [Cinemachine]
  * [Timeline]
  * [Addressable Asset]
  * [IK]


## Csharp


## Lua

* [导言](Lua/README.md)
* [Lua介绍](Lua/Lua Language/First Notes About Lua.md)
* 在Unity里使用Xlua
  * [愉快地在Unity里与Xlua玩耍](Lua/Unity with Xlua/CodeHappierWithXlua.md)
  * [Xlua Examples学习（一）](Lua/Unity with Xlua/XluaExampleNotes.md)
  * [Xlua Examples学习（二）](Lua/Unity with Xlua/XluaExampleNotes02.md)
  * [更多有关Xlua](Lua/Unity with Xlua/XluaMoreInfo.md)
* 热更新

## 渲染

* [入门常识](Rendering/Intro/README.md)
* [Unity Shader](Rendering/Unity Shader/README.md)
