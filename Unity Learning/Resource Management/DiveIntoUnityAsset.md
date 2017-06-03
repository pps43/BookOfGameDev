Unity资源统称为`Asset`。


## （一）Asset 和 Unity.Object
[官方详解](https://unity3d.com/learn/tutorials/temas/best-practices/assets-objects-and-serialization)

### Asset在Unity中的内涵

- **泛指Unity可识别的资源文件**，据可依具体分成 Native Asset 和 Non-Native Asset
  > Native: 拥有可以被 Unity 直接识别的格式，例如MAT文件
Non-Native: 外部资源，需要导入（import）才能被Unity识别，例如FBX文件，PNG文件等等

- 每个Asset文件具有全局唯一的`File Guid`，存放在相应的**“.meta”**文件中

- None-Native资源的meta文件中记载了其使用了哪种 **Importer**
> 例如PNG用的是TextureImporter，FBX用的是ModelImporter，cs代码用的是MonoImporter，shader文件用的是ShaderImporter，等等。很多默认用的是DefaultImporter。也可以自定义一些Importer，继承AssetImporter。

- 所有资源Import后暂存在`Library/metadata`文件夹中，这样就不用每次打开编辑器都进行耗时的导入操作（例如纹理压缩）


### Object在Unity中的内涵

- **一个或多个Object共同描述了`序列化了的`资源的实例**。也就是说，Object是序列化了的，需要经过实例化，才会加载到内存中。
- 一个Asset里面有多个Object，内部以`localID`(或叫`FileID`)区分
> 常见 Object如：Sprite, Texture, AudioClip, Material, Motion, GameObject, Component等等。特殊的两个：`MonoBehavior`, `ScriptableObject`

- 只有 Unity.Object 及其子类，才可以在 Unity 编辑器中拖放。
- 只有 Component 及其子类，才可以附着到gameObject上。代码中访问的Component对象不可单独存在，必须要附着在某个gameObject上。

## （二）File Guid, local ID 和 Instance ID
首先，为了方便用文本编辑器查看资源内部结构，需进行如下操作：
> Edit->Project Settings->Editor ->Assets Serialization，选择 force text

我们已经知道，每个文件有全局唯一的File Guid。一个文件中有多个Object，每个Object有文件中唯一的local Id。所以，每个Object的全局唯一标识为`File Guid + local Id`。Unity编辑器会自动将文件路径和 FileGuid的映射关系保存在一张表里。


一个典型的Object（Monobehavior）的描述如下：
```cpp
--- !u!114 &114105719140368892
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_PrefabParentObject: {fileID: 0}
  m_PrefabInternal: {fileID: 100100000}
  m_GameObject: {fileID: 1612640623411334}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 1980459831, guid: f5f67c52d1564df4a8936ccd202a3bd8, type: 3}
  m_Name: 
  m_EditorClassIdentifier: 
  m_UiScaleMode: 1
  m_ReferencePixelsPerUnit: 100
  m_ScaleFactor: 1
  m_ReferenceResolution: {x: 800, y: 600}
  m_ScreenMatchMode: 0
  m_MatchWidthOrHeight: 0
  m_PhysicalUnit: 3
  m_FallbackScreenDPI: 96
  m_DefaultSpriteDPI: 96
  m_DynamicPixelsPerUnit: 1
```
第一行，`&`符号后面的数字就是这个Object的local id。

那为什么还需要什么`Instance ID`呢？官方给出的解答是：
> While File GUIDs and Local IDs are robust, GUID comparisons are slow.

解决办法是用Instance ID。
>  Unity internally maintains a cache that translates File GUIDs and Local IDs into simple integers that are unique **only during a single session**。

当一个新的Object被注册到cache中，其自动获得一个Instance ID。具体来说，启动时，Unity会载入场景和所有Resource目录下的Objects，为其分配InstanceID。如果动态加载或卸载AssetBundle，Unity会生成或删除为其分配的InstanceID。所以重新载入同一个AB包，获得的InstanceID不可保证相同。

## （三）MonoBehavior, ScriptableObject 和普通 c# 类

从上一节对Monobehavior类型的Obejct的描述可以看到，其有一个对`MonoScript`的引用。Monoscript包含的信息很简单：
> 
- assembly name
- class name
- namespace.

构建工程时，Unity会将Asset文件夹里面所有的脚本打包进程序集里（每种语言有不同的程序集例如`Assembly-CSharp.dll`；Plugin里面也会单独打进一个程序集`Assembly-CSharp-firstpass.dll`）。与其他资源不同，程序集在应用启动时就全部载入了。

由此可以想到，**对于AB包来说，里面其实不含任何的c#代码资源，只有指向这些预先生成的dll文件的引用**。

c#代码可以分为三种，继承Monobehavior的，继承ScriptableObject的，以及c#原生的。其应用场景和区别如下：

- 只有Monobehavior继承了component类，才可以往gameObject上挂。
- 不需要挂在gameObject上，比如只为了存数据，可以用ScriptableObject或c#原生类。
- 接上一条，若要求可序列化，则只有用ScriptableObject。
- ScriptableObject与c#原生类的区别还在于：前者是一种Unity的资源，需要通过`Destroy`或` Resources.UnloadUnusedAssets()`来释放，其不归c#的GC管理。（[相关讨论](https://forum.unity3d.com/threads/scriptableobject-vs-plain-c-class.328325/)）

