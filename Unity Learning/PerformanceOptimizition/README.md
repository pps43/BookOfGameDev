我的优化ppt还没同步到gitbook，另外要补充以下内容。


- 删除不用的代码
> 所有代码都会被打进`Assembly-CSharp.dll`，而且与其他资源不同，程序集在应用启动时就全部载入。而gameobject的monoscript组件，包含的信息仅仅是assembly name，class name和namespace，而不是编译后的代码。

- 精简Resource文件夹，非多处多次创建的物体，都做到场景里，而不是放到Resource里动态加载。
> 不这么做的最大问题不是包体大小，而是会影响启动速度。具体来说，启动时，Unity会载入场景和**所有Resource目录下的Objects**，为其分配InstanceID。

- 零碎对象的Update逻辑由一个Update Manager统一驱动和管理 （https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity8.html）
> 为的是减少零碎的Monobehaviour脚本中调用Update, FixedUpdate and LateUpdate等函数，因为每个物体每帧调用都需要**从c#层调用native层**（Unity维护了一个列表依次遍历），每帧都这样消耗很可观（初始化Prefab的卡顿也是因为调用每个组件上的Awake和Onenable需要从c#层访问native层）。
更好的方法是：有一个 Update manager，继承Monobehaviour，在它的Update()中去遍历所有对象的某个函数。这样管理对象是否每帧去执行某个逻辑也变得高效和灵活。注意：如果用委托来绑定函数调用关系，切忌不要简单的使用 `+=`，这样效率很低。




待阅读 托管堆内存
https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity4-1.html

## UI scrollView
优化点：
- 在ScrollView所在的GameObject上挂`RectMask2D`脚本，这样在视线外的部分在canvas rebuild的时候就不会去生成几何数据、排序等等操作。
- 对象池用于大量item。

### scrollView 对象池的实现
首先说简单方法：创建几个带有layoutElement的空白项作为“placeholder”，然后从池子中取UI挂到这些placeholder下。当滚动时，**动态改变UI的parent**。

这样的问题是，不只是改变parent的UI，而是所有item的UI都会触发rebuild。原因是：Unity不区分父节点改变和子节点顺序改变，这两种行为都会触发` OnTransformParentChanged`，最终会触发Graphic接口的`SetAllDirty`，因此在下一帧渲染前，会执行layout rebuild。

让需要改变的UI自己rebuild，其他UI不受影响的方法是：**动态改变UI的位置，而不是parent。**通常可以写一个 custom layout去实现`LayoutGroup`并监听`ScrollRect.onValueChanged`。