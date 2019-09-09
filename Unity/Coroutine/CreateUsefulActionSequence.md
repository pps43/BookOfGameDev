# 一个好用的异步动作队列

本篇介绍一个小组件（只有两个类`ActionSequence`和`ActionSequenceManager`），用于在Unity脚本中方便的串行执行命令，命令支持包括匿名函数、范型委托、Unity协程和Wait对象、自定义迭代器等类型等。

举个简单的调用例子：

```csharp
public class MyClass
{
    private bool _isPlayingAni;

    void Func()
    {
        ActionSequenceManager.Create()
            .Then(()=> _isPlayingAni = true;)
            .Then(new WaitForSeconds(2f))
            .Then(()=> _isPlayingAni = false;)
            .Run();
        
        //不会阻塞后面的代码
    }
}
```

对比一下原来的写法：

```csharp
public class MyClass : Monobehaviour
{
    private bool _isPlayingAni;

    void Func()
    {
        StartCoroutine(MyCoro());
        
        //不会阻塞后面的代码
    }

    IEnumerator MyCoro()
    {
        _isPlayingAni = true;
        yield return new WaitForSeconds(2f);
        _isPlayingAni = false;
    }
}


```

可以看到，用类似Promise的写法，使代码变得更紧凑，并且普通c#类也可以调用Unity的各类`WaitForXXX`事件，而不需要继承`Monobehaviour`。
在实际使用中，好处远不止于此。

首先，通过将命令传入队列然后串行执行的方式，是一种解耦，解耦了命令和命令之间的衔接逻辑。当需求发生改动的时候，原来的写法要把整个函数重写并且Debug，而新的写法只要修改传入的命令即可，节省了笔者大量时间。这种把命令当作数据或配置的方法，可以在代码不重新编译的情况下，动态执行不同的命令。这种灵活性正是前面介绍过的“命令模式”带来的。这种思想在项目多个模块中均有应用。

其次，功能扩展方便。`Then`函数通过重载，可以接收丰富的参数类型。而且除了`Run`,`Stop`，还有可选的`Callback`接口等。经过一段时间的迭代，基本能够覆盖开发需求。`Actionsequence`的“并行”、嵌套都能够支持。


下面再介绍下`ActionSequenceManager`编写中的要点。这个类负责管理每一个`Actionsequence`的生命周期，因此是单例。为了节省开销（这部分存在开销是因为要从Mono调用Native的一些函数），对每个`Actionsequence`对象通过对象池来取用（将 reuse 选项打开）。要注意，一旦获取到这个`Actionsequence`，就要设立标志位，而不能等到执行`Run`方法后才设置。因为你不知道是否在同一帧内有多少个获取请求，绝不能让两个请求获取到同一个`Actionsequence`。

目前使用中要注意的一点是：当一个游戏对象调用了这个组件并运行命令中，因为别的原因被直接销毁了（Native层被销毁，c#这里不能马上感知），如果正在运行的命令需要读写该游戏对象的Native数据，会触发异常。为了避免这种情况，使用者可以选择在`OnDestroy()`中将自己持有的、正在运行的`Actionsequence`手动调用其`Stop()`方法。

本篇介绍的该组件源码地址[见这里](https://github.com/jonyzhao/ZToolsForUnity/tree/master/Assets/ZTools/Async/ActionSequence)，会根据实际使用进行更新迭代。