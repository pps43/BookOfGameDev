# Profiler的正确解读方法

Unity自带的Profiler工具十分强大，尤其是可以用于真机测试，是性能优化必备的工具。其使用方法这里不再介绍，可参考《性能分析与真机调试》一文。这里主要解决以下痛点：对于Profiler中给出的条目名称，并不太了解对应程序哪里的问题，对应该做何种优化总是缺乏十足的把握。

以下基于Unity5.3.5自带的Profiler版本进行阐述。

## CPU耗时分析
###  加载相关
- Loading.ReadObject
 - Loading.ReadObjectThreaded
 - Loading.AwakeFromLoad
> [原文链接](https://zhuanlan.zhihu.com/p/23733044) Loading.ReadObject是Unity引擎的资源加载函数，一般出现在切换场景和加载API调用时，这其中包括纹理、网格、Material、Shader、AnimationClip等资源。[纹理加载](https://blog.uwa4d.com/archives/LoadingPerformance_Texture.html).[动画加载](https://blog.uwa4d.com/archives/Loading_AnimationClip.html)（文中提出，可以写一个资源监测的工具，防范大于救灾）

- Instantiate
 - Instantiate.Copy/Awake/Produce

- Loading.LoadFileHeaders
 - Loading.ArchiveFileSystemSize
> 实例化引起的序列化操作（[原文链接](http://uwa-download.oss-cn-beijing.aliyuncs.com/uwa4d.com/%E5%A4%A7%E8%A7%84%E6%A8%A1%E5%9C%BA%E6%99%AF%E7%9A%84%E8%B5%84%E6%BA%90%E6%8B%86%E5%88%86%E5%92%8C%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD-%E4%B8%8A%E6%B5%B720161126.pdf)）。。。避免层级复杂，组件awake过多；尝试拆分prefab，流式Instantiate（？？？）；预加载粒子系统



- Loading.UpdatePreloading
 - Application.LoadLevelAsyncOperationToComplete
  - Application.LoadLevelAsync Intergrate
   - Loading.AwakeFromLoad
    - SkeletonGraphic.Awake
   - UnloadScene
> Loading.UpdatePreloading是Unity引擎最主要的加载函数，主要负责卸载当前场景的资源，并且加载下一场景中的相关资源和序列化信息。


- Font.CacheFontForText
 - Font.InvokeTextureRebuild_Internal

- GameObject.Activate
 - 各个组件的OnEnable

- GarbageCollectAssetsProfile
引擎在执行UnloadUnusedAssets操作（该操作是比较耗时的,建议在切场景的时候进行）[参考](http://www.gad.qq.com/article/detail/11394)

- SpriteMeshGenerator.TraceShape/Decompose/Simplify
[参考](https://blog.uwa4d.com/archives/TechSharing_66.html)。通常出现在加载或者创建（Sprite.Create）SpriteMeshType为Tight类型的Sprite时，Tight类型的Sprite在加载或创建时，需要检测图片的alpha区域从而生成多边形。计算量较大，建议将其改为FullRect模式。

### 渲染相关
- Canvas.SendWillRenderCanvases
- PutGeometryJobFence
> [原文链接](https://blog.uwa4d.com/archives/1809.html) PutGeometryJobFence是Unity项目的主线程在等子线程的一些网格计算操作完成，其在不同的模块中均有出现（UGUI模块、渲染模块、Mesh.Skin操作、Animator动画模块等等），出现的地方不同，其本身含义也并不相同，不能一概而论。如果项目中使用了UGUI，那么很大几率（>80%）是由于UI模块的不合理操作所致。并且该函数与WaitingForJob经常成对出现，特别是在Android平台上。。。最终结论是UI模块的重建开销很高，优化后该项耗时可接近0ms。

- Graphics.PresentAndSync / Gfx.WaitForPresent
 - Device.Present
  - Present.BlitToCurrentFB
  - Present.SecondarySurfaces
> [原文链接](http://blog.sina.com.cn/s/blog_155a1f2470102wa88.html)若开启了多线程渲染，则出现的是Gfx.WaitForPresent，其实二者表示的含义都是一样的：等待GPU以进行下一次渲染。出现这种情况有以下原因：（1）开了垂直同步（ios无法关闭垂直同步，只能设置帧率），则可能是CPU开销远小于GPU开销，导致等待VSync；也可能是CPU开销很高，错过了VSync，只有等待下一次VSync（`常见于加载大AB包，Resource.Load加载大纹理`）（2）GPU开销很高，CPU要等待GPU上一帧渲染完成。。。最终结论是该项耗时无法直接优化，根本原因在别处。

- Graphics.Blit
> 通常与屏幕后处理的shader相关。

- Selectable.OnCanvasGroupChanged (BaseSceneEaseInOut.Start里面)

- UIBehaviour.OnCanvasGroupChanged

- Canvas.BuildBatch
 - JobAlloc.Overflow

- WaitForTargetFPS
> 设定目标帧率后，用于填补多余时间的。优化的时候可以忽略这个。顺便补充一点：ios上帧率只能设置为30或60，其他帧率无效，默认是60。

## GPU分析
好像暂不支持。


## 内存分析

## 音频分析
音频的消耗不能明确反映在内存分析里，之前有一个创建过多AudioSource导致卡顿的BUG，很久都没能发现，就是因为Profiler中的CPU和内存表现正常。