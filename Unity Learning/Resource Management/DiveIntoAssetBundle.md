# 浅析Assetbundle机制

---
- [浅析Assetbundle机制](#assetbundle)
  - [一、概要](#)
    - [定义](#)
    - [优点](#)
    - [缺点](#)
    - [内部结构](#)
    - [关于 LZMA 和 LZ4 算法](#lzma--lz4)
  - [二、生成ab包](#ab)
    - [老方法](#)
    - [新方法（unity 2017后使用）](#unity-2017)
    - [实验中的方法（预计unity 2018.3后使用）](#unity-20183)
  - [三、ab包的内部结构](#ab)
    - [AssetBundles](#assetbundles)
    - [AssetBundles.manifest](#assetbundlesmanifest)
    - [myprefabs](#myprefabs)
    - [myprefabs.manifest](#myprefabsmanifest)
  - [四、加载/管理/卸载ab包](#ab)
    - [加载ab包](#ab)
    - [加载时处理ab包依赖](#ab)
    - [AssetBundle管理器](#assetbundle)
    - [ab包小工具](#ab)
  - [五、加载ab包中的Object](#abobject)
  - [例子](#)

1. [官方深度详解 best practice](https://unity3d.com/learn/tutorials/topics/best-practices/assetbundle-fundamentals?playlist=30089)
2. [官方指南系列 manual](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)

## 一、概要

### 定义

`AssetBundle System`提供了一套资源文件打包、并可以被unity方便地索引的方法。打出的包下文我们简称为`ab包`。`AssetBundle`除了抽象的内涵，这个字符串本身也是unity暴露给c#的wrapper类名。

### 优点

* 资源分离，缩减安装包大小
* 资源差异化，不同设备选择不同资源加载
* 资源更新，不过不包含代码逻辑更新
* 更细粒度的运行时内存管理
* 缩短构建时间，加快项目迭代

### 缺点
摘取自[官方总结](https://www.youtube.com/watch?v=OSx4GouFFCE)
* 学习成本大
  * 构建、加载、卸载都需自己编写代码
  * API要么太底层要么太高层
  * variants系统形同虚设
* 和现有打包流程结合不紧密，导致错漏
  * 代码裁剪、shader variants
* 缺少可视化的资源管理


### 内部结构

* header
  * assetbundle的id
  * 是否压缩
  * manifest。所包含Object的查找表（基于平衡二叉树或红黑树）
* data
  * 如果开启压缩，则默认采用`LZMA`压缩算法将**整体压缩**成一个ab包。
  * 如果选择`LZ4`算法，则可以选择为每个 Object **单独压缩**。这样的好处是加载一个Object可以不用解压整个ab包到缓存中，加载的速度甚至和不压缩差不多。注意，Unity5.3以后才引进了LZ4这个选项。

### 关于 LZMA 和 LZ4 算法

LZMA比LZ4压缩/解压速度更慢，但压缩率更高（压缩后的文件更小）。

详见[这里](https://catchchallenger.first-world.info/wiki/Quick_Benchmark:_Gzip_vs_Bzip2_vs_LZMA_vs_XZ_vs_LZ4_vs_LZO)。

## 二、生成ab包
### 老方法
* 资源归属到ab包（在Unity编辑器inspector窗口下方完成）
* 调用打包脚本构建ab包（这里必须自己写，并且只能放到 `Editor` 目录下）

```csharp
using UnityEditor;
using System.IO;

public class CreateAssetBundles
{
    [MenuItem("Assets/Build AssetBundles")]
    static void BuildAllAssetBundles()
    {
        string assetBundleDirectory = "Assets/AssetBundles";
        if(!Directory.Exists(assetBundleDirectory))
        {
            Directory.CreateDirectory(assetBundleDirectory);
        }


        /* opt:
         * None                     =   LZMA 压缩
         * UncompressedAssetBundle  =   不压缩
         * ChunkBasedCompression    =   LZ4 压缩
         * 
         * 参见 https://docs.unity3d.com/Manual/AssetBundles-Building.html
         */
        BuildAssetBundleOptions opt = BuildAssetBundleOptions.None;

        BuildPipeline.BuildAssetBundles(assetBundleDirectory, opt, BuildTarget.StandaloneWindows64);
    }
}
```

### 新方法（unity 2017后使用）
- 下载插件 `Assetbundle Browser` 完成可视化管理当前有哪些ab包，是否有重复打包现象。并且可以一键生成ab包。
- 下载插件 `Asset Graph Tool` 完成复杂打包流程的可视化？


### 实验中的方法（预计unity 2018.3后使用）
Addressable Asset，和之前的不太兼容了。

## 三、ab包的内部结构

按照上面的打ab包脚本，从菜单中执行后，在AssetBundles文件夹中有`2*(n+1)`个文件（n是自定义的ab包的数量，ab包必须全部小写）。例如我们有myprefabs和mysprites两个ab包，那么会生成：

> AssetBundles  
> AssetBundles.manifest  
> myprefabs  
> myprefabs.manifest  
> mysprites  
> mysprites.manifest

下面逐个分析每个文件。

### AssetBundles

是一个序列化的 `AssetBundleManifest 对象`。提供了一些API，诸如：

> * `GetAllAssetBundles()` which returns all the AssetBundle names in this build.
> * `GetDirectDependencies()` which returns the direct dependent AssetBundle names.
> * `GetAllDependencies()` which returns all the dependent AssetBundle names.
> * `GetAssetBundleHash(string)` which returns the hash for the specified AssetBundle.
> * `GetAllAssetBundlesWithVariant()` which returns all the AssetBundles with variant.

### AssetBundles.manifest

记载了CRC，以及各个AssetBundle之间的**依赖关系**。

例如下面有两个AssetBundle：myprefabs和mysprits，其中myprefabs里有一个Object用到了mysprits里的一张图，所以产生了依赖关系。

```
ManifestFileVersion: 0
CRC: 320579488
AssetBundleManifest:
  AssetBundleInfos:
    Info_0:
      Name: myprefabs
      Dependencies:
        Dependency_0: mysprites
    Info_1:
      Name: mysprites
      Dependencies: {}
```

### myprefabs

ab包。注意，使用共同资源的prefab最好打到同一个ab包中，否则会大量浪费空间。

### myprefabs.manifest

对于每个AssetBundle都生成了对应的manifest，记载了包含哪些Assets以及依赖何种资源。这个文件仅供打差异包使用，运行时不可用。

> [This manifest file is only used for incremental build, not necessary for runtime.](https://docs.unity3d.com/500/Documentation/Manual/BuildingAssetBundles5x.html)

```
ManifestFileVersion: 0
CRC: 129934035
Hashes:
  AssetFileHash:
    serializedVersion: 2
    Hash: eb750738b2f2979dc0f9a95b36fa0de9
  TypeTreeHash:
    serializedVersion: 2
    Hash: 80992b54a8950244f5c8a5c98a859660
HashAppended: 0
ClassTypes:
- Class: 1
  Script: {instanceID: 0}
- Class: 4
  Script: {instanceID: 0}
- Class: 21
  Script: {instanceID: 0}
  Script: {fileID: 1392445389, guid: f5f67c52d1564df4a8936ccd202a3bd8, type: 3}
- Class: 114
  Script: {fileID: -765806418, guid: f5f67c52d1564df4a8936ccd202a3bd8, type: 3}
- Class: 114
  Script: {fileID: 708705254, guid: f5f67c52d1564df4a8936ccd202a3bd8, type: 3}
- Class: 115
  Script: {instanceID: 0}
Assets:
- Assets/Prefabs/Button.prefab
- Assets/Prefabs/cardSprite.prefab
Dependencies:
- F:/TestProjects/TestAssetBundle/TestAssetBundle/Assets/AssetBundles/mysprites
```

## 四、加载/管理/卸载ab包

### 加载ab包

加载ab包主要有4个API，但具体表现会根据压缩方法、平台的不同而不同。

1. ~~AssetBundle.LoadFromMemoryAsync / CreateFromMemory~~  \(DO NOT USE IT ANYMORE
   \)
2. AssetBundle.LoadFromFile
3. WWW.LoadFromCacheOrDownload
4. UnityWebRequest's DownloadHandlerAssetBundle \(on Unity 5.3 or newer\)

下面对4个API分别来做简要说明。由于这几个API在移动端和编辑器模式下行为不同，具体参考 [https://docs.unity3d.com/Manual/AssetBundles-Native.html](https://docs.unity3d.com/Manual/AssetBundles-Native.html)。下文说的都是在移动端上的表现。

#### _AssetBundle.LoadFromFile_

* 适用于未压缩或LZ4压缩模式下的本地文件。
* 对于LZMA压缩不使用。
* 对于安卓平台下存放在`Streaming Assets`底下的文件不适用（因为路径是以jar:开头的，识别不了）。
* 调用该API仅载入ab包的header部分，不会直接将所有都放到内存中。调用ab.Load之类的方法再寻找并载入具体的object。
* 该API是首选的，因为快速、省内存。

#### _WWW.LoadFromCacheOrDownload_

为了弄清楚该函数的流程，首先定义：

* 步骤1：服务器下载到本地
* 步骤2：本地载入到Unity专门的Cache（Cache中一定是解压的）
* 步骤3：Cache到可用的内存对象


对于某个ab包：

* 若已经存在Cache中，则直接执行步骤3，相当于调用 `AssetBundle.LoadFromFile`；
* 否则，则会从url中读取，读取完毕后会开启一个新的后台线程尝试去解压，解压完放到Cache中，之后同上。

使用该API需要注意的是：

* 不要连续调用下载ab包，这样会导致开启太多解压线程，卡死
* 由于www.bytes的需要，**www对象会额外保存一份ab包的数据放在Native堆上**，若未能及时、正确的释放www这个wrapper对应的Native资源，则会造成浪费


> **Unity Cache 是什么**
和CPU的Cache所指代的含义不同，Unity的Cache指的是存在本地的。一般占用空间的上限是4G。如果要下载的ab包超出了剩余Cache的大小，Unity会删除最近最少使用的ab包以及被标记为`Caching.MarkAsUsed`的ab包。另外还有一个属性`Caching.expirationDelay`，超过期限的ab包会被自动删除。再有一个就是清空所有Cache的API`Caching.CleanCache`。除此之外，难以从更细的粒度对Cache进行管理。





#### _UnityRequest + AssetBundleDownLoadHandler_

* 这两个API在Unity 5.3以上版本中才能使用
* 可以理解为是`WWW.LoadFromCacheOrDownload`的替代，解决了其两个痛点。多个线程解压不再会卡死了，因为由一个内置的jobsystem进行分配。多余的数据也没了，但记得这个Handler也是个wrapper要主动释放哦。
* 可以配置要不要放到Cache中。要存到Cache，则类似`WWW.LoadFromCacheOrDownload`；不要Cache，则`AssetBundle.LoadFromFile`。



### 加载时处理ab包依赖

首先要明确的一点是：**unity不会自动处理ab包的依赖加载**。

好消息是，对于有依赖关系的ab包，加载的先后顺序不重要，如果ab包1依赖于ab包2，那么，只要自己保证在加载ab包1的某个Object的时候，其依赖的ab包2已经被加载就行。这一点可以通过使用上文介绍过的`AssetBundleManifest`对象完成。

假设我们需要从名为`myprefab`的ab包中加载某个object，而这个ab包又依赖别的ab包，那么加载逻辑是：

```csharp
AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
string[] dependencies = manifest.GetAllDependencies("myprefab"); //Pass the name of the bundle you want the dependencies for.
foreach(string dependency in dependencies)
{
    AssetBundle.LoadFromFile(Path.Combine(assetBundlePath, dependency));
}
```

### AssetBundle管理器

首先，Unity不会自动卸载刚刚从当前场景被移除的Object，而是在特定时机触发。为了更好的表现，通常需要人工管理。对于AssetBundle来说，管理的不好，通常会造成内存浪费（重复创建Obejct）或丢失材质。

AssetBundle的管理核心在于何时调用`AssetBundle.Unload(bool)`，以及选true还是false。这个API主要是卸载内存中的ab包header数据，若选true，则还会卸载从这个ab包中加载的所有Object。若选false，则不会，而是断开ab包和这些Obejct之间的关联。这也可能造成一个问题：选false后，再次加载这个ab包，然后又走一遍加载这些Object的逻辑，则会造成比较隐蔽的内存浪费——之前的Object还没清除（也不是没办法清除，方法接下来说），和现在的Object是完全重复的！所以，官方建议，在掌握何时调用该API的前提下，尽量使用true参数。

一定要使用false参数的话，通常要在调用该API追加如下逻辑：将该ab包加载出来的Object的引用（注意包含两部分：c\#代码和场景）都消除，然后调用`Resources.UnloadUnusedAssets`。

### ab包小工具

Unity官方制作的AssetBundle管理器，既包含了下载和管理的逻辑，还包含了编辑器插件用于模拟ab包下载。

  
插件安装： [https://www.assetstore.unity3d.com/en/?&\_ga=2.79209684.1521571944.1497786482-670410889.1464698583\#!/content/45836](https://www.assetstore.unity3d.com/en/?&_ga=2.79209684.1521571944.1497786482-670410889.1464698583#!/content/45836)

插件使用：[https://docs.unity3d.com/Manual/AssetBundles-Manager.html](https://docs.unity3d.com/Manual/AssetBundles-Manager.html)  
插件源码：[https://bitbucket.org/Unity-Technologies/assetbundledemo](https://bitbucket.org/Unity-Technologies/assetbundledemo)



另外，为了使工作流更可视化，Unity正尝试推出新的ab包插件，详见： [https://blogs.unity3d.com/2016/10/25/new-assetbundle-graph-tool-prototype/](https://blogs.unity3d.com/2016/10/25/new-assetbundle-graph-tool-prototype/)

## 五、加载ab包中的Object

总的来说有同步和异步两种思路，每种思路下各有3个API：加载全部，加载部分，加载一个。

同步理论上比异步快1帧以上，实际体验更快。Unity 5.1之前，每帧最多能异步加载一个object，所以就更更慢了（这是一个bug，5.2修复了）。另外，异步加载在每一帧占用的时间片长度是可以通过`Application.backgroundLoadingPriority`进行优先级调节的。默认的优先级对应每帧最多占用10ms。

Unity 5.3之后，多个object可以并行地由多个后台线程加载了。

一次性加载全部比每次加载一个所用的总时间短。

**加载**（Load）完某个Object后，通常下一步就是以此为母板进行**实例化**（Instantiate）了。注意区分这两个概念。

## 例子

[https://unity3d.com/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager](https://unity3d.com/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager)

