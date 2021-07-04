# Unity协程实用技巧

---
- [Unity协程实用技巧](#unity%E5%8D%8F%E7%A8%8B%E5%AE%9E%E7%94%A8%E6%8A%80%E5%B7%A7)
    - [基本使用](#%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8)
    - [内置的`YieldInstruction`](#%E5%86%85%E7%BD%AE%E7%9A%84yieldinstruction)
    - [可返回值的协程](#%E5%8F%AF%E8%BF%94%E5%9B%9E%E5%80%BC%E7%9A%84%E5%8D%8F%E7%A8%8B)
    - [嵌套协程](#%E5%B5%8C%E5%A5%97%E5%8D%8F%E7%A8%8B)
    - [多态协程](#%E5%A4%9A%E6%80%81%E5%8D%8F%E7%A8%8B)
    - [可处理异常的协程](#%E5%8F%AF%E5%A4%84%E7%90%86%E5%BC%82%E5%B8%B8%E7%9A%84%E5%8D%8F%E7%A8%8B)

---

## 基本使用

### yield

- `yield break`。中止协程中的后续操作。
- `yield return null/1/true/false`。挂起协程，回到主函数逻辑，下一帧【执行游戏逻辑时】从挂起的位置继续。
- `yield return new WaitForSeconds(1.0f)`。挂起协程，1s后【执行游戏逻辑时】从挂起的位置继续。
- `yield return new WaitForEndOfFrame()`。挂起协程，这一帧的渲染结束【显示到屏幕之前】时从挂起的位置继续。
- `yield return StartCorotine(subCoroFunc());`。嵌套协程。当【子协程完全执行完毕时 && 执行游戏逻辑时】从挂起的位置继续。嵌套协程的作用不大，只是看上去更有层次。

```cs
//type 1
IEnumerator CoroFunc1()
{
    yield return null;
}
//type 2 yield return type is limited to string
IEnumerator<string> CoroFunc2()
{
    yield return "hello";
}
//type 3 with param
IEnumerator<string> CoroFunc3(string str)
{
    yield return "hello" + str;
}
//type 4 return IEnumerable: not necessary.
IEnumberable CoroFunc4(int i)
{
    yield return null;
}
```

创建并开始协程：
```cs
//type 1 use anywhere inside a monobehaviour class
Coroutine coro = StartCoroutine(CoroFunc1());

//type 2 use inside a coroutine, unity automatically create a Coroutine, and call it next frame.
yield return CoroFunc1();

//type 3 just don't use it, very slow
Coroutine coro = StartCoroutine("CoroFunc4", 1);

```

停止协程：
```cs
StopCoroutine(coro);
```



## 内置的`YieldInstruction`
除了常用的`WaitForSeconds`,`WaitForEndOfFrame`等，以下同样继承自`YieldInstruction`的子类也有其便捷之处：
 - WaitUntil
 - WaitWhile

```cs
bool finishCondition = false;
yield return new WaitUntil(() => finishCondition)
//go on when finishCondition is true
```

## 可返回值的协程


有时会有这种需求：等待协程结束后，获取到`yield return`的结果。
```cs
//pseudocode
var retCode = yield return new XXX();
Print(retCode);
```

除了网络上给出的[较早的实现方式](https://answers.unity.com/questions/24640/how-do-i-return-a-value-from-a-coroutine.html)外，现在还可以利用`CustomYieldInstruction`来实现。


```cs
public class CoroutineWithData : CustomYieldInstruction
{
    public int retCode { get; private set; }
    private bool _finished = false;

    public void onFinished(bool isSucc)
    {
        _finished = true;
        retCode = isSucc ? 0 : -1;
    }

    //must override this
    public override bool keepWaiting
    {
        get
        {
            return !_finished;
        }
    }
}

//how to use:
public class main : MonoBehaviour {

    private IEnumerator Start()
    {
        CoroutineWithData coro = new CoroutineWithData();
        yield return coro;
        Debug.Log("retcode:" + coro.retCode);
    }
    //click btn to continue coro
    public void btnClickHandler()
    {
        coro.onFinished(true);
    }
}


```

## 嵌套协程

协程的嵌套使用不仅是为了更明了地组织逻辑代码，而且可以实现协程同步：

- 同步、阻塞式（最基本用法）

```cs
IEnumerator A()
{
    yield return StartCoroutine(B());
    
    //wait for B() finishes
}
```

- 同时进行
```cs
IEnumerator A()
{
    StartCoroutine(B());
    
    //execute at the same time
}
```

- 异步，阻塞式
```cs
IEnumerator A()
{
    Coroutine sub = StartCoroutine(B());

    //do sth at the same time

    yield return sub;

    //wait for B() finishes
}

IEnumerator AA()
{
    Coroutine sub1 = StartCoroutine(B());
    Coroutine sub2 = StartCoroutine(B());

    //do sth at the same time

    yield return sub1;
    yield return sub2;

    //wait for all finishes

}
```




## 多态协程

```cs
//parent class:
protected virtual IEnumberator  DoSomething()
{
	yield return null;
}

//child class
protected override IEnumberator DoSomething()
{
    //you can call base like this:
    yield return StartCoroutine(base.DoSomething();

    //do something
}
```


## 可处理异常的协程

http://twistedoakstudios.com/blog/Post83_coroutines-more-than-you-want-to-know
