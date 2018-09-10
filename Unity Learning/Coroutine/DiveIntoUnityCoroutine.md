# 深入剖析Unity协程

---
- [深入剖析Unity协程](#%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90unity%E5%8D%8F%E7%A8%8B)
    - [开胃菜：从`IEnumerator/IEnumerable` 到`Yield`](#%E5%BC%80%E8%83%83%E8%8F%9C%E4%BB%8Eienumeratorienumerable-%E5%88%B0yield)
    - [主菜：Unity协程的实现](#%E4%B8%BB%E8%8F%9Cunity%E5%8D%8F%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0)
    - [甜点： 带返回值的协程](#%E7%94%9C%E7%82%B9-%E5%B8%A6%E8%BF%94%E5%9B%9E%E5%80%BC%E7%9A%84%E5%8D%8F%E7%A8%8B)
---


## 开胃菜：从`IEnumerator/IEnumerable` 到`Yield`

c#语言中，迭代器这个特性大家不会陌生，最常见的莫过于`foreach`了。`foreach`能够对一个实现了`IEnumerable`接口的对象`dataSource`进行遍历访问其中的元素。
```cs
foreach (var item in dataSource)
{
    Console.WriteLine(item.ToString());
}
```

`foreach`的遍历过程可以拆解为：
```cs
IEnumerator iterator = dataSource.GetEnumerator(); 
while (iterator.MoveNext()) 
{ 
    Console.WriteLine(iterator.ToString());
}
```

细心的读者会发现，为什么迭代器要涉及`IEnumerable` 和 `IEnumerator`两个接口而不是直接在`dataSource`中实现`MoveNext`和`Current`？

这正是迭代器模式的要点。这个模式将存储数据和遍历数据的**职责分离**，在c#中对应为`IEnumerable` 和 `IEnumerator`两个接口（其实还有两个泛型接口：IEnumerable\<T>, IEnumerator\<T>，不加 T 的话则默认为IEnumerator\<object>）。本文对该模式不做展开讨论。

如何利用`IEnumerable` 和 `IEnumerator`，自定义一个支持`foreach`遍历的类呢？

在c#1.0中，你只能这样做：
```cs
public class DataSource : IEnumerable
{
    public IEnumerator GetEnumerator()
    {
        Enumerator enumerator = new Enumerator(0);
        return enumerator;
    }

    public class Enumerator : IEnumerator, IDisposable
    {
        private int state;
        private object current;

        public Enumerator(int state)
        {
            this.state = state;
        }

        public bool MoveNext()
        {
            switch (state)
            {
                case 0:
                    current = "Hello";
                    state = 1;
                    return true;
                case 1:
                    current = "World";
                    state = 2;
                    return true;
                case 2:
                    break;
            }
            return false;
        }

        public void Reset()
        {
            throw new NotSupportedException();
        }

        public object Current
        {
            get { return current; }
        }
        public void Dispose()
        {
        }
    }

}

class Program
{
    static void Main(string[] args)
    {
        DataSource dataSource = new DataSource();
        foreach (string s in dataSource)
        {
            Console.WriteLine(s);
        }
    }
}
```
可以看到，需要在`DataSource`类中实现一个`Enumerator`类，其采用状态机的方式实现`MoveNext`的逻辑，稍有不慎就会产生差错。

c#2.0后引入了语法糖`yield`，较完美的解决了迭代器模式的易用性这个问题。同样的功能，只需寥寥几行便可实现。
```cs
public class DataSource : IEnumerable
{
    public IEnumerator GetEnumerator()
    {
        yield return "Hello";
        yield return "World";
    }
}
```

从这个例子中，可以猜想到`yield return`是如何实现等价效果的：对于一个位于返回值为`IEnumerator`的函数里的yield，做如下处理：

1. 静默创建了一个`IEnumerator`对象
2. 立刻调用了这个对象的`MoveNext()`方法，其执行了第一个yield之前的逻辑
3. 遇到第一个yield时，将`Current`赋值为`yield return`后面的值），保存当前状态并挂起。
4. 下次调用`MoveNext()`时，从刚才的yield之后的语句开始执行。直到最后一个`yield return`语句时，`MoveNext()`返回false。


在实践中，发现yield在返回值为`IEnumerable`的函数中也能起作用，和`IEnumerator`似乎是一样的。这是为何？俗话说“好人做到底，送佛送到西”，本文既然说深入剖析，这个疑点自然不能放过。

首先我们知道，`yield`只是语法糖，那么能不能看到编译器将其展开后的结果呢？笔者将测试代码编译成DLL后，放在**ILSpy 2.4.0**版中看反编译后的c#，终于发现了使用`IEnumerator`和`IEnumerable`的不同。

![IEnumerator和IEnumerable的yield展开结果](/assets/yield_IEnumerator_decompile.png)

从上图可以发现，`IEnumerable`会创建一个线程ID，并且初始状态为 -2（表明 `GetEnumerator()`还没有被调用）。如果另一个线程在迭代中途调用了`GetEnumerator()`,则会新建立一个该类对象。这里介绍的是比较简单的情况，当有参数传递时，要去维护线程安全就比较复杂了。总之，**用 IEnumerable 是线程安全的**。


不过语法糖终究是语法糖，`yield`的使用是有限制的，比如用于异常处理。

> `yield return`不能用于`try-catch`中，只能用在`try-finally`的try中。
> 
> `yield break`可以用于`try-catch`，但不能用在`finally`块中。[《C#参考》](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/yield)

更多这方面的讨论推荐《C# in depth》作者写的[这篇博客](http://csharpindepth.com/Articles/Chapter6/IteratorBlockImplementation.aspx)，以及[这篇](http://twistedoakstudios.com/blog/Post83_coroutines-more-than-you-want-to-know)。





## 主菜：Unity协程的实现
前一节告诉我们：在返回值为`IEnumerator/IEnumerable`的函数中，`yield return [value]` 可以被展开，实现迭代器的效果。`[value]`是本次`MoveNext()`的返回值`Current`，可以是`object`类型。下次调用`MoveNext()`时，从刚才的yield之后的语句开始执行。

Unity利用这个特性实现了协程。协程本篇就不介绍了，这方面已有不少笔墨，亦超出本篇的讨论范围。继续刚才话题，在Unity的协程中，返回值Current并不能直接被使用者获得，而是内部进行了处理。

对于不同类型的返回值，效果亦不相同。比如`yield return null;`就是挂起协程，回到主函数逻辑，下一帧从挂起的位置继续。`yield return new WaitForSeconds(1f);`就是挂起1秒后再继续。这是怎么实现的？先看看反编译后的代码。

- yield return null 的结果：
  
![yield_null](/assets/yield_null.png)

- yield return new WaitForSeconds(1f) 的结果

![yield_ws](/assets/yield_waitSecond.png)

看完后觉得信息量不大。Unity到底用了什么魔法？网上[有篇博客](http://twistedoakstudios.com/blog/Post83_coroutines-more-than-you-want-to-know)给出了自己的猜想（看完源码后发现这位仁兄猜的真准……）


> When you make a call to `StartCoroutine(IEnumerator)` you are handing the resulting IEnumerator to the underlying unity engine.

> `StartCoroutine()` builds a `Coroutine` object, runs the first step of the IEnumerator and gets the first yielded value. That will be one of a few things, either "break", some `YieldInstruction` like `"Coroutine", "WaitForSeconds", "WaitForEndOfFrame", "WWW"`, or something else unity doesn't know about. The Coroutine is **stored** somewhere for the engine to look at later.

> ... At various points in the frame, Unity goes through the stored Coroutines and checks the Current value in their IEnumerators.

>  - `WWW` - after Updates happen for all game objects; check the isDone flag. If true, call the IEnumerator's MoveNext() function;
> - `WaitForSeconds` - after Updates happen for all game objects; check if the time has elapsed, if it has, call MoveNext();
> - `null` or some unknown value - after Updates happen for all game objects; Call MoveNext()
> - `WaitForEndOfFrame` - after Render happens for all cameras; Call MoveNext

> `MoveNext` returns false if the last thing yielded was "break" of the end of the function that returned the IEnumerator was reach. If this is the case, unity removes the IEnumerator from the **coroutines list**.


由于上面已经概括了Unity实现协程的思想，这里稍作补充，源码就不贴了：
- `StartCoroutine`创建了`Coroutine`对象`coroutine`，该对象保存了`yield return`展开后的IEnumerator对象指针、`MoveNext`和`Current`的函数指针、结束后应当唤醒的协程的指针、指向调用者`Monobehaviour`的指针等等，并将该对象`coroutine`保存到该`Monobehaviour`的活跃协程列表中。然后立即调用了`coroutine.Run()`。
- `coroutine.Run()`首先尝试调用`InvokeMoveNext`，若发现当前协程执行完成，则会尝试调用应当唤醒的协程，否则才真正执行`MoveNext`，获得返回值`monoWait`。
- 根据返回值`monoWait`的类型，进行不同的处理。通常是传递不同的参数给`CallDelayed`函数。对于返回值是`Coroutine`类型（c#那边用了协程嵌套），会将这个返回值的结束后应唤醒的协程的指针指向当前的`coroutine`。笔者这里发现了一种不太常见的用法：当返回值为`IEnumerator`类型（c#那边没有用StartCoroutine去开启嵌套协程，而是直接在yield return 后调用）时，Unity会自动为其创建一个`Coroutine`对象并初始化，效果同样。
- `CallDelayed`函数传入了运行协程对象的方法、qingli协程对象的方法、清理条件等。函数内部创建了一个`Callback`对象，加到了全局的`DelayedCallManager`的列表中。游戏主循环会在每一帧调用`DelayedCallManager.Update`，在满足一定条件时（比如对应的Monobehaviour对象还没被销毁等）调用`Callback`对象的方法。


## 甜点： 带返回值的协程

看到这里，似乎意犹未尽。有些读者可能会问，除了知道如何用`yield`实现自己的可迭代的类，以及Unity利用`yield`实现协程的原理外，对日常代码有什么立竿见影的作用？这里就介绍一个小技巧：**自定义可返回值的协程。**

上一节我们知道，Unity内部将`yield return`的结果进行了处理，但常常我们也想去访问协程的结果，比如玩家发起坐下请求。结果是超时了？还是坐下失败了？如果能用协程实现下面的写法，不仅省去了回调函数定义的冗余，还可以使上下文更连贯。

```cs
//期望的用法
SitDownCoroutine cd = new SitDownCoroutine(requestSitDown(seatID));
yield return cd.coroutine;
Debug.Log("result is " + cd.result);  //  'success' or 'fail' or 'timeout'
```

为了实现上述效果，笔者在经历的项目中见过一些复杂的写法，其思想是自己模拟Unity的这套运行协程的机制。不过受限于开发量，往往只能支持简化的功能。[这篇帖子](https://answers.unity.com/questions/24640/how-do-i-return-a-value-from-a-coroutine.html)介绍了一种相对简单有效的方法，有兴趣可以看看。笔者这里提供相当简练的方法，利用Unity5.3以后提供的 [`CustomYieldInstruction`](https://docs.unity3d.com/ScriptReference/CustomYieldInstruction.html) 功能。当Unity遇到返回值为`CustomYieldInstruction`类型时，会检查`keepWaiting`的值，直到该值为`false`才会结束协程。

```cs
public class SitDownCoroutine : CustomYieldInstruction
{
    public RetCode result { get; private set; }
    private bool _finished = false;

    public SitDownCoroutine(Action<long> request)
    {
        //将onResponse加入对应的回包监听
    }

    private void onResponse(RetCode retCode)
    {
        result = retCode;
        _finished = true;
    }

    public override bool keepWaiting
    {
        get { return !_finished; }
    }
}

```

篇幅有限，更多关于Unity协程的实用技巧将收录进下篇。如果本篇没有令读者满足的话，请恕笔者厨艺有限，招待不周。

おそまつ~