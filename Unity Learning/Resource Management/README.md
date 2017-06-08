# 资源管理

## 引言
对于Unity游戏，堆内存占用大致分为三块：`Native堆`（Unity资源）、`Native Dll`（第三方DLL库） 和`Mono堆/栈`（c#资源）。具体来说：
> 
- Native (internal)
  - AssetData: _Textures,AudioClips,Meshes_
  - Game Objects & Components: _Transforms, etc_
  - Engine Internals: _Managers,Rendering,Physics,etc_
- Mono (managed)
  - scripts' object (reference type)
  - `Unity Wrappers objects`
- Native Dll
  - 3rd library Dlls

堆的内存管理方式为：

- Mono堆通过对应版本的 Mono GC 进行内存管理；
- Native堆由Unity提供了一些API以及某种自动内存管理。

关于mono和GC的讨论见下一章节，本节讨论的是Native堆上的内存管理。

---

**注**：
上文中的`Unity Wrappers objects`就是unity暴露给c#的一些类，大都继承自Object类，比如GameObject、AudioClip、Transform等。

可以这样理解：unity object数据是比较庞大的，**像一座冰山**，大部分保存在native堆中，而露出到c#的部分可以通过轻量级的wrapper对象来访问。

这些wrapper对象本身通过mono堆的GC管理：**`wrapperObj = null`之后不久会被GC，但其对应的Native资源不会释放，除非调用Resources.UnloadUnusedAssets()，或场景切换后自动跳用这个API**。

另外，unity提供了API或实现了 `IDisposable` 接口来通过这些wrapper对象来主动释放Native资源。
- API的例子:Destroy(wrapperObj), DestroyImmediate(wrapperObj)。
- IDisposable的例子具体见后文关于c#资源的讨论。

---


## 资源分析小工具
- unity内存分类剖析
https://bitbucket.org/Unity-Technologies/memoryprofiler/overview

- 查看IL代码工具：ILSpy
http://www.fishlee.net/soft/ilspy_chs/