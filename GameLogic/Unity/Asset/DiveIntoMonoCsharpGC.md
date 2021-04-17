# 深入Mono的C\#资源回收

---
- 深入Mono的c\#资源回收
    - [mono是什么](#mono是什么)
    - [GC为什么耗时](#GC为什么耗时)
    - [何谓`mono内存泄漏`](#何谓`mono内存泄漏`)
    - [减少GC的常见套路](#减少GC的常见套路)
    - [IDisposable与非托管资源](#IDisposable与非托管资源)
---

* [Unity官方的优化GC建议](https://unity3d.com/learn/tutorials/topics/performance-optimization/optimizing-garbage-collection-unity-games)


C\#资源，在这里指的是Unity.Object以外的类型对象（更具体地分**为托管资源**和**非托管资源**）。在Unity中，是由mono虚拟机通过垃圾回收机制（GC）管理的。

> **托管资源**：由CLR\(Common Language Runtime\)管理分配和释放发资源，即从mono运行时里new出来的对象。  
> **非托管资源**：不受CLR管理的对象，如：Windows内核对象、文件、数据库连接、套接字和COM对象等。

## mono是什么

我们总在说的“mono”，到底指的是什么？[参考这里](http://www.mono-project.com/docs/about-mono/)。直白点说，mono是.NET的一套免费公开的实现。其特性要落后于.NET。~~而Unity目前使用的mono更是老版的mono（2.6.5版）~~ Unity2017以后默认的mono环境兼容了.NET4.6。简而言之：

> Mono has a C\# compiler, a runtime and a lot of libraries.



## GC为什么耗时

具体是怎样判断哪些需要被回收呢？我们的直观想法是`引用计数`，很快很简单。但这样会有循环引用而不得释放的问题（iOS用的是自动引用计数，为了解决这个问题，需要开发者使用弱引用）。Unity携带的mono，GC采用的是`Boehm GC`，而最新的mono已经采用了`分代 GC`——sgen。Boehm策略是`rootObj`，不是引用计数。概括一下Unity游戏中的GC耗时原因：

* `Boehm GC`以全局数据区和当前寄存器中的对象为根节点进行遍历，这本身就耗时；
* 启动GC前，会暂停所有些需要mono内存分配的线程，完成GC后才恢复，造成卡顿。

所以，一方面我们要少制造垃圾，另一方面要控制垃圾回收调用的时机。

## 何谓`mono内存泄漏`

有了GC机制，为什么还有mono内存泄漏？

答：**mono内存泄漏，并不是真的泄漏**，而是mono虚拟机向操作系统（os）申请到的内存不会还给os，即使gc后，那些不用的内存（`unused size`）也是自己保存起来备用。不够的时候会再向os申请。因此，mono占用的内存（`heap size`）是只增不减的。当有些资源实际上已经不用了，却占着空间无法被GC回收，就称为mono内存泄漏。

```csharp
//获取mono占用内存的api
[System.Runtime.InteropServices.DllImport("__Internal")]
public static extern long mono_gc_get_used_size();

[System.Runtime.InteropServices.DllImport("__Internal")]
public static extern long mono_gc_get_heap_size();
```

当整个应用被杀死时，mono占用的内存才能还给os。

一个有趣的小知识：当在PC模拟器上运行Unity游戏时，点击三角形终止游戏运行，所有内存真的会归还给操作系统吗？答：不是的。如果你用了多线程，那么unity是不对其他线程负责的，点击三角形关闭的只是unity对应的线程。若其他线程访问了unityEngine相关的数据，将是无效的。

> The play mode is just another thread inside the same mono VM. --[Dreamora](https://forum.unity3d.com/threads/threading-causes-memory-leak.87652/)



## 减少GC的常见套路

首先可以看看[Unity官方对减少GC的总结](https://unity3d.com/learn/tutorials/topics/performance-optimization/optimizing-garbage-collection-unity-games)

### foreach

注：在`Unity5.5`以前，foreach会产生GC，要使用下面的写法。之后就可以直接用foreach了。

遍历字典的无GC写法：

```csharp
var tmp = dic.GetEnumerator();
while(tmp.MoveNext())
{
    //tmp.Current.key;
    //tmp.Current.Value
}
```

### 字符串拼接

```csharp
string a = "afadf";
string c = a + i;
```

当`i`不是string类型，实际执行的是`string.Concat((object)a, (object)i)`，存在装箱拆箱。  
这种字符串拼接常见于打日志操作。为此可以做一些优化。

```csharp
long uin = 111;

MyDebug.Log("the player uin is: " + uin);//常见写法，有额外GC和装箱拆箱

MyDebug.Log("the player uin is: " + uin.ToString());//无装箱拆箱，有额外GC

StringBuilder sb = new StringBuilder();
MyDebug.Log(sb.Append("the player uin is: ").Append(uin).ToString()); 
//无装箱拆箱，无额外GC，但是这样写麻烦
```

利用泛型函数可以简化上述无装箱拆箱和额外GC的写法。

```csharp
StringBuilder sb = new StringBuilder();
public static void Debug(string str)
{
    Debug.Log(str);
}

public static void Debug<T1, T2>(T1 arg1, T2 arg2)
{
    sb.Length=0;//清空sb
    sb.Append(arg1).Append(arg2);
    Debug.Log(sb.ToString());
}

//可以再补充支持3个、4个参数的。。。

MyDebug.Log("the player uin is: " , uin);//现在可以这么写，无额外GC，无装箱拆箱
```

### yield

- `yield break`。中止协程中的后续操作。
- `yield return null/1/true/false`。挂起协程，回到主函数逻辑，下一帧【执行游戏逻辑时】从挂起的位置继续。
- `yield return new WaitForSeconds(1.0f)`。挂起协程，1s后【执行游戏逻辑时】从挂起的位置继续。
- `yield return new WaitForEndOfFrame()`。挂起协程，这一帧的渲染结束【显示到屏幕之前】时从挂起的位置继续。
- `yield return StartCorotine(subCoroFunc());`。嵌套协程。当【子协程完全执行完毕时 && 执行游戏逻辑时】从挂起的位置继续。嵌套协程的作用不大，只是看上去更有层次。
 

### struct替代class

struct是在栈上的，更快，而且不存在GC。什么时候用呢？  
通常我们创建的引用类型总是多于值类型。如果以下问题的回答都为yes，那么我们就应该创建为值类型：

* 该类型的主要职责是否用于数据存储？
* 该类型的共有借口是否完全由一些数据成员存取属性定义？
* 是否确信该类型永远不可能有子类？
* 是否确信该类型永远不可能具有多态行为？



## IDisposable与非托管资源

[什么是非托管资源（Unmanaged Resources）？](https://stackoverflow.com/questions/3433197/what-exactly-are-unmanaged-resources)
[使用IDisposable的正确姿势](https://stackoverflow.com/questions/538060/proper-use-of-the-idisposable-interface)

非托管资源，顾名思义，占用的是非托管堆上的内存。常见的非托管资源如：

- Open files
- Open network connections
- Unmanaged memory
- In XNA: vertex buffers, index buffers, textures, etc.

但要注意，并不是所有文件读写操作都是在与非托管资源直接打交道，很多时候.net封装了一层wrapper类供我们使用，暗地里完成了很多“dirty work”。

> When you open file in .NET, you probably use some built-in .NET class System.IO.File, FileStream or something else. Because it is a normal .NET class, it is managed.

对于托管资源，其由GC管理；对于非托管资源，是要手动调用自定义的卸载方法的。为了统一命名这个卸载方法，要求实现`IDisposable`接口。如果没有手动调用，那么，由于这个非托管资源是挂在c#的object底下的，当这个object被GC最终处理时，会调用一个`Finalize`方法（c#中则是去实现一个类似c++析构函数名字的函数），在这里可以作为释放非托管资源的最后一道屏障。具体怎么用，参考上面的链接。

