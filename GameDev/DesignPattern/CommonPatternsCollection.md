# 居家旅行必备的六大设计模式

本文介绍游戏客户端开发中几个十分**朴实但实用**的设计模式，是个人在经历不同游戏项目后的经验总结。
每种模式在阐明用途后，均给出参考代码（以Unity引擎为例）。在实践中，需要根据项目需求进行扩展或简化。

## 单例
单例可能是被使用最多也是被误用最多的一种模式。由于管理游戏对象时势必需要各类“Manager”，且这些Manager类对象通常生命周期比较长，如果用一个个全局单例来实现，不管在哪里都能访问，似乎十分方便。然而，笔者并不鼓励这种依赖多个全局单例的写法，这样会造成生命周期不明确、资源泄漏隐患增大、模块调用关系混乱而难以维护。推荐的方法是通过依赖注入的方式传递给需要访问它的类。

这并不是否认单例写法的使用面，甚至在早期的小规模预研中，全局单例的写法有助于尽快搭建框架进行玩法验证。用Unity引擎进行原型开发时，由于只有主线程能访问`GameObject`这类Native资源，所以`写法三`通常已经足够。

```cs
//写法一：c# 线程安全的单例（Double-checked locking）
public sealed class Singleton
{
    private static volatile Singleton instance;
    private static object syncRoot = new object();

    private Singleton(){} //防止外界创建该类对象

    public static Singleton Instance
    {
        get
        {
            if (instance == null) //提升性能，不用每次加锁
            {
                lock (syncRoot)
                {
                    if (instance == null)
                    {
                        instance = new Singleton();
                    }
                }
            }

            return instance;
        }
    }
}

//写法二：c# 线程安全的单例，利用了 Lazy<T> 内部已有的多线程处理逻辑。适用 .Net4 以上版本
public class Singleton
{
    private static readonly Lazy<Singleton> instance = new Lazy<Singleton>(() => new Singleton());

    private Singleton() { }

    public static Singleton Instance => instance.Value;
}

//写法三：继承MonoBehaviour以用于Unity引擎，单线程版。
public abstract class USingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;

    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                T[] objs = GameObject.FindObjectsOfType<T>(true);
                if (objs.Length > 0)
                {
                    instance = objs[0];
                    for (int i = 1; i < objs.Length; i++)
                    {
                        GameObject.Destroy(objs[i].gameObject); //去除重复
                    }
                }
                else
                {
                    GameObject newObj = new GameObject(typeof(T).Name); //自动创建 object
                    DontDestroyOnLoad(newObj); //生命周期跨场景
                    instance = newObj.AddComponent<T>();
                }
            }

            return instance;
        }
    }
}
//使用方法
public class GameManager : USingleton<GameManager>
{

}

```

## 抽象工厂

## 对象池
对象池的主要作用是加速对象的分配与回收。游戏中通常会用到大量同类对象（如敌人、子弹、网络协议包），如果每次都调用API去申请内存，不仅更慢，而且更容易产生内存碎片从而降低系统运行效率。很多时候，一个简单的对象池就能在一定程度上缓解这个问题。（相比于内存池，对象池整体也更为简单。内存池的讨论[参考这里](https://www.zhihu.com/question/25527491/answer/56571062)）

🟡TODO: 对象池的设计和注意点

## 观察者模式
🟡TODO: 以一个简易的消息分发系统为例。

## 命令模式
🟡TODO: 以一个简易的输入模块为例。

## 状态模式
当业务逻辑变得越来越复杂，一种常见的处理方法是添加各种标记位或状态变量，配合`if-else`实现整套逻辑。笔者曾经在一份古老的代码仓库中修改长达几千行的`if-else`逻辑块，别有一番滋味在心头。诚然，这种代码扎根在运行了二三十年的系统中，进行重构的风险已经太大。如果能在设计之初预见到未来的复杂度，一定会采用其他方式。

有限状态机（`FSM`）就提供了一种分解复杂逻辑的方案。其有以下内涵：
- 可划分为有限个状态对象；
- 不同状态是互斥的，某一时刻仅有一个状态有效；
- 状态之间具有明确的切换规则；
- 状态机可以接收外部输入；
- 下一个状态是由当前状态、外部输入、当前状态的切换规则决定的；

划分状态的好处是：将变量以及依赖的环境包裹在一起，并与别的变量和逻辑隔离开来，极大减少了认知负担。

`FSM`有多种实现方式，简单如`switch-case`，但从封装性和灵活性出发，状态模式是更好的实现方式。下面就基于`C#`给出状态模式的泛型实现，涉及3个实体：状态、状态机、状态机管理器。

### 状态机管理器的实现
状态机管理器（`FSMManager`）负责创建、终止状态机，以及驱动其执行每帧更新逻辑。

```cs
public class FSMManager : USingleton<FSMManager>
{
    private List<IFSM> allFSMs = null;

    public FSM<T, M> createFSM<T, M>(T owner, BaseState<T, M> initialState, BaseState<T, M> initialGlobalState) where T : class where M : struct
    {
        var newFSM = new FSM<T, M>(owner, initialState, initialGlobalState);

        if (this.allFSMs == null)
        {
            this.allFSMs = new List<IFSM>();
        }

        this.allFSMs.Add(newFSM);
        newFSM.OnSelfDispose += onFSMStop;
        return newFSM;
    }

    private void onFSMStop(IFSM fsm)
    {
        int idx = -1;

        if (this.allFSMs != null && fsm != null)
        {
            idx = this.allFSMs.IndexOf(fsm);
            if (idx >= 0)
            {
                fsm.OnSelfDispose -= onFSMStop;

                if (!fsm.IsRunning)
                {
                    this.allFSMs.RemoveAt(idx);
                }
                else
                {
                    Log.error(fsm.ToString(), "should stop before delete from list");
                }
            }
        }
    }
    
    // 在Unity引擎中，LateUpdate()回调每帧执行，但顺序位于Update()之后
    private void LateUpdate()
    {
        if (this.allFSMs == null)
        {
            return;
        }

        foreach (var fsm in this.allFSMs)
        {
            if (fsm.IsRunning)
            {
                fsm.Update();
            }
        }
    }
}
```

### 状态机的实现
状态机（`FSM<T,M>`）对外暴露启动/终止、暂停/恢复、更新等接口，以及消息驱动和状态转移框架逻辑。这里有四点值得一说的设计：
- **泛型的使用**。对于不同类型的角色对象，其状态种类和可以接受的消息种类应当是相互隔离的。因此，这里用`T`表示状态机所属的角色类。用`M`表示该状态机接收的消息类型。
- **统一的接口**。由于使用了泛型类，为了能将不同类型的状态机统一管理，定义了接口`IFSM`。
- **全局状态**。若存在一种状态，可以从其他任何状态转变而来，那么为了避免繁冗，可以将其从状态转换图中独立出来，称为全局状态，也可以理解为角色的第二状态。这在“死亡”逻辑中很有用：不论在何种状态下，一旦血量小于等于0，则执行死亡逻辑。
- **是否立即状态切换**。当触发了状态切换的条件时，除了立即切换状态，也可以等到该逻辑帧结束后下一逻辑帧开始前切换状态。后者的合理之处在于：同一帧内某个对象暴露给外界所有其他对象的状态是相同的。以敌人碰到子弹后死亡为例，如果子弹立即切换状态并销毁，那么轮到敌人处理碰撞逻辑时，将访问不到子弹对象的一些属性。但下一帧切换状态也会让某些逻辑变得不再直观，反而因此掣肘。总之，是否立即进行状态切换需要根据玩法进行权衡。

```cs
/// <summary>
/// 状态机接口定义，方便 <see cref="FSMManager"/> 统一管理
/// </summary>
public interface IFSM
{
    public bool IsRunning { get; set; }
    public void Start();
    public void Pause();
    public void Resume();
    public void Stop();
    public void Update();
    public delegate void OnDispose(IFSM fsm);
    public event OnDispose OnSelfDispose;
}

/// <summary>
/// 状态机泛型类，T 为状态机所属角色的类型，M 为接收的消息类型
/// </summary>
public class FSM<T, M> : IFSM where T : class where M : struct
{
    public bool IsRunning { get; set; }
    public event IFSM.OnDispose OnSelfDispose;

    private T owner;
    private BaseState<T, M> curState;
    private BaseState<T, M> lastState;
    private BaseState<T, M> globalState;

    public FSM(T owner, BaseState<T, M> state, BaseState<T, M> globalState)
    {
        this.IsRunning = false;
        this.owner = owner;
        this.lastState = state;
        this.curState = state;
        this.globalState = globalState;
    }

    public void Start()
    {
        if (!this.IsRunning)
        {
            this.IsRunning = true;
            this.curState.Enter(this.owner, null);
        }
    }

    public void Pause()
    {
        this.IsRunning = false;
    }

    public void Resume()
    {
        this.IsRunning = true;
    }

    public void Stop()
    {
        if (this.IsRunning)
        {
            this.curState.Exit(this.owner);
            this.IsRunning = false;
            this.OnSelfDispose?.Invoke(this);
        }
    }

    public void Update()
    {
        if (this.IsRunning)
        {
            if (this.curState != null) { this.curState.Update(this.owner); }
            if (this.globalState != null) { this.globalState.Update(this.owner); }
        }
    }

    /// <summary>
    /// 发送消息到该状态机，并支持立即拿到一个返回值。
    /// </summary>
    public object OnMessage(M msg, bool retFromGlobal = false)
    {
        object msgRet = null;
        object msgRetGlobal = null;
        if (this.IsRunning)
        {
            if (this.curState != null) { msgRet = this.curState.OnMessage(this.owner, msg); }
            if (this.globalState != null) { msgRetGlobal = this.globalState.OnMessage(this.owner, msg); }
        }
        return retFromGlobal ? msgRetGlobal : msgRet;
    }

    /// <summary>
    /// 立即切换状态，并支持传入参数到写一个状态
    /// </summary>
    public void ChangeState(BaseState<T, M> newState, object param = null)
    {
        if (!this.IsRunning)
        {
            Log.error("Cannot change state, FSM is not runnning");
            return;
        }

        if (newState == null)
        {
            Log.error(owner, "cannot change state to null");
            return;
        }

        if (this.curState == null)
        {
            Log.error(owner, "Fatal error: curState is null, newState=", newState);
            return;
        }

        if (newState.GetType().Equals(this.curState.GetType()))
        {
            Log.warn(this.owner, "cannot change to the same state:", this.curState);
            return;
        }

        this.lastState = this.curState;
        this.curState = newState;
        this.lastState.Exit(this.owner);
        this.curState.Enter(this.owner, param);
    }

    public bool IsInState(Type type)
    {
        return this.curState.GetType().Equals(type);
    }

    public string GetStateName()
    {
        return this.curState.GetType().Name;
    }
}
```

### 状态基类的实现
所有状态的基类为`BaseState`，其中`Enter`/`Exit`仅在进入/离开状态时调用一次，`Update` 每帧调用一次。另外在实际使用中，状态类可以拥有自己的字段。

```cs
public abstract class BaseState<T, M> where T : class where M : struct
{
    public virtual object OnMessage(T owner, M msg) { return null; }
    public virtual void Enter(T owner, object param) { }
    public virtual void Update(T owner) { }
    public virtual void Exit(T owner) { }
    public override string ToString()
    {
        return this.GetType().Name;
    }
}
```

### 使用案例

上述状态机可以用来实现敌人AI。首先需要：
- 定义角色类，例如 `Enemy` 类，其持有状态机对象`FSM`。
- 定义消息类，例如 `LocalMsg` 结构体，包含消息ID（枚举类型），消息携带的数据。
- 定义若干状态类（例如`IdleState`），继承BaseState `<Enemy, LocalMsg>`，内部通过调用 `owner.FSM.ChangeState` 实现状态切换。

然后便可按如下方式使用：

```cs
FSM<Enemy, LocalMsg> fsm = FSMManager.Instance.createFSM(this, new IdleState(), new GlobalState());
fsm.Start();
fsm.OnMessage(new LocalMsg(LocalMsg.ID.onHurt)); //向其发送受伤消息
fsm.Stop();
```

完整例子见 [github](https://github.com/jonyzhao/ZToolsForUnity/tree/master/Assets/Demo/Demo_01)。

### 状态模式的不足之处和应对方法
状态模式解决了大量`if-else`堆砌造成的混乱，但当状态种类到达几十上百后，维护上述状态机也变得棘手。下面提供两种常见解决思路，这里不展开介绍。
- 层次状态机。即将一组联系紧密的状态归结到一个父状态下。通过这样分层，使得每一层的状态转移图不再那样复杂。
- 行为树。不严谨地说，行为树将状态和状态转移逻辑分离开了，好处是状态转移逻辑比较集中而不是散落在一个个状态类定义里，且修改状态转移逻辑不会破坏状态本身，便于频繁修改以打磨玩法。

以上方案结合可视化节点编辑器，是一个大型游戏项目中常见的解决方案。

## 更多资料
- 《Game Programming Patterns》一书较为系统的介绍了游戏开发中常见的设计模式，以上介绍的几种模式在书中亦有提及。在线阅读地址：http://gameprogrammingpatterns.com/