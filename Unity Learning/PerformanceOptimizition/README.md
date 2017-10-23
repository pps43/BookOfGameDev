集合ppt补充


- 删除不用的代码
> 所有代码都会被打进`Assembly-CSharp.dll`，而且与其他资源不同，程序集在应用启动时就全部载入。而gameobject的monoscript组件，包含的信息仅仅是assembly name，class name和namespace，而不是编译后的代码。

- 精简Resource文件夹，非多处多次创建的物体，都做到场景里，而不是放到Resource里动态加载。
> 不这么做的最大问题不是包体大小，而是会影响启动速度。具体来说，启动时，Unity会载入场景和**所有Resource目录下的Objects**，为其分配InstanceID。
