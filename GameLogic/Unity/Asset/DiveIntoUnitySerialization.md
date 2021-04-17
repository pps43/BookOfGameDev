
序列化和持久化是游戏开发入门后绕不开的点。本文以Unity引擎的游戏开发为例讲讲遇到的坑和经验，适用于客户端开发，后台同学到这里可以 `return` 了。这是一篇当初学习时写的总结，现在结合工程经验加工了一下。

# 引子

先列举序列化和持久化的差异：

 1. 序列化将对象转化成某种格式（例如二进制、json、xml、yaml、csv等），以供下一步处理。
 2. 持久化将内存数据以某种方式（例如文件I/O、数据库）保存到外存（例如硬盘），以供以后使用。

一言以蔽之，序列化技术侧重于解决对象的传输问题，持久化技术侧重于解决对象的存储问题。

# 干货

	
首先，Unity引擎对`UnityEngine.Object`有自己的序列化机制，但并没有开放成API，且存在一些无法序列化的类型，比如**泛型、字典、高维数组、委托**等（不被Unity序列化的情况具体见[官网的这个说明](https://docs.unity3d.com/ScriptReference/SerializeField.html)）。不能被引擎序列化的表现是：这些类型的字段在 编辑器的 inspector 中无法显示。
	
>	2019.6.20 更新： unity 2019.3已经可以支持对引用类型的序列化。因此可以在inspector中显示接口类型的成员了，只需加一个attribute：`[SerializeReference]`，原讨论帖见[这里](https://forum.unity.com/threads/serializereference-attribute.678868/)

对不能被Unity序列化的类，有一些简单的解决方法弥补不足：
> 
1. 对于高维数组，将其低维化。即底层采用一维数组来替代。
2. 对于字典，key和value各自存储成List，运行时用字典，序列化时用数组。
3. 对于泛型类，用一个新类将其封装并用 [Serializable] 修饰新类。
4. 对于不带返回值的委托，可以用 `UnityEvent`来序列化。注意使用`UnityEvent<T>`时，要参考3中的方法处理一下才行。带返回值的委托解决方法比较复杂，以后我再说。而`UnityEvent`序列化反序列化中**如何保持对象的生命周期和引用关系**也比较有东西可挖，[这里有一个网友给出了他做的插件，我没用过](https://answers.unity.com/questions/661958/how-to-make-delegatesevents-survive-assembly-reloa.html)。
	
以上方法使用简单，能满足日常需求。还有一些Unity序列化失败不是因为类型本身，而是因为不支持的语法特性，比如**空引用、多态**。

先说说空引用。有两个`Serializable`的类：A、B。类A中有一个类B对象的空引用。**当Unity在对A进行序列化时，会自动构造B类对象来填补这个空引用**。这一步有个小坑：当B类存在基类且基类有不止一个构造函数时，Unity不能正确的序列化B的初始值。不过这个问题通常没副作用。但是，当类B就是类A时，一个无法直视的问题发生了：无限循环。Unity引擎对其有深度限制，但在达到这个深度前会**造成严重卡顿并浪费空间**。

再说多态，在一个`List<BaseClass>`中，列表项指向的对象`BaseClass`的派生类，但Unity序列化时，不会识别，只会序列化基类的信息。
	
对于以上以及更加变态的情况，Unity提供了一个接口：`ISerializationCallbackReceiver`，通过实现该接口的两个方法`OnBeforeSerialize` 和 `OnAfterDeserialize`，使得原本不能被引擎正确序列化的类可以按照程序员的要求被加工成引擎能够序列化的类型。[Unity官方的这个例子](https://docs.unity3d.com/ScriptReference/ISerializationCallbackReceiver.html)实现了对Dictionary的加工使其能够序列化。其实c#也有类似的接口，如果不想走.Net提供的序列化方法，可以通过实现 'ISerializable` 自定义序列化和反序列化过程，参考[微软的文档](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.iserializable?view=netframework-4.7.2)。
	
	
如果不想在Unity的序列化机制上缝缝补补，常见的解决方案是：json、xml、yaml和二进制。重点说json和二进制这两种代表性方案，点一下yaml方案。

##json方案
	
Unity 5.3后自带了json工具：`JsonUtility`。常用的第三方库是`Json.Net` 和`LitJson`，这些都比.Net自带的`DataContractJsonSerializer`要快很多，尤其是`JsonUtility`，很快而且GC很少。除了速度，不同的库对序列化的支持程度也不尽相同。
		
`JsonUtility`的能力和限制参考官网。**其对引擎内建类型支持(比如`Vecotr3`)较好，这是其他json不能直接做到的**。然而其底层走的还是引擎自己的序列化，而我们这里讨论的主要是对普通c#类对象的序列化反序列化能力，所以这里有点掉分。对于多态的支持，官网给出了一种解决方法：两次解码。第一次可以解出基类中的公有字段`JsonUtility.FromJson<BASE_CLASS>`，利用这个字段包含的子类信息进行重新进行解码： `JsonUtility.FromJson<SUB_CLASS>`，让人感觉做东西做了一半（Unity经常这样，毕竟发展太快）。`Json.Net`是支持多态的，参考[这里](https://stackoverflow.com/questions/6348215/how-to-deserialize-json-into-ienumerablebasetype-with-newtonsoft-json-net)。


##YAML方案
YAML的全称是`YAML Ain't Markup Language`，是一种对人友好的数据序列化语言。YAML是json的超集，也就是说YAML解析器可以解析json。

json和xml对格式要求非常严格，对任何一个括号和逗号的改动都不行。而YAML的可读性不输xml，大于等于json，编辑性却更方便和健壮。而且**支持注释、支持自引用，支持复杂数据类型**。不过YAML在跨平台的支持上不如json，这就是历史因素了。

Unity采用YAML来描述结构，例如[序列化复杂的场景](https://docs.unity3d.com/Manual/YAMLSceneExample.html)。然而依然没有公开这种序列化方法的API。如果想使用YAML作为序列化的格式，可以从Unity商店下载插件 `YamlDotNet`，支持移动平台和PC平台。使用上还是很方便的，特别的是，其只会序列化具有get和set的property类型。具体本文就不再介绍了。


##二进制方案
一般来说，二进制方案啥都好，除了兼容性的让人不放心。然而，经测试，`BinaryFormatter`对于字段的增删改几种情况下都不会报错，而是尽可能的将二进制数据解析到对应的字段上。猜想是因为c#这里序列化的数据包含了meta信息，理论上完全可以根据比对这些信息来处理字段变化造成的异常，而不是直接crash。同理，如果想让json的自动解析达到同样的效果，那么json序列化时就得包含类似的meta信息。


`BinaryFormatter`使用上参 考[微软官网](https://docs.microsoft.com/en-us/previous-versions/dotnet/netframework-1.1/72hyey7b(v=vs.71)) 即可。


本篇内容写到这里基本结束了，不过既然开头提到了持久化，这里稍微多说两句。

Unity初学者都经历过利用 `DontDestroyOnLoad` 来修饰游戏对象来实现游戏运行中跨场景的数据“持久化”吧，也一定都用过 `PlayerPrefs`保存`key-value`数据到本地，以供下次启动时读取。对于`ScriptableObject` 这种方法，特别指出，继承`ScriptableObject`的类对象确实可以被Unity序列化后持久化成文件，不需要走`Monobehaviour`存储数据那样要依附在一个`GameObject`身上再做成`Prefab`这个弯路，并且其能够保持资源索引关系。但是，这仅限于在编辑时。打包成app在客户端运行时，scriptableObject对象是只读的，不可写。所以非得将其持久化的话（开发插件往往会用到这个，参考这里），只能将scriptableObject先序列化成json或二进制，然后存储为文件。

以上。